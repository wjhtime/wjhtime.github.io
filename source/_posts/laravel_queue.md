---
title: laravel队列源码分析
date: 2017-12-21
categories: laravel
tags: laravel
---

php artisan queue:listen该命令通过php artisan queue:work来实现队列监听，源码中采用proc_open等相关函数实现多进程启动队列，每次启动均重启框架，所以性能上不是很高，占用的内存过大。但是在代码调试过程中可使用，每次更新队列文件的代码之后不需重新启动队列监听，即可实现队列文件的修改。

### php artisan queue:listen
```php
//获取相关参数，之后调用queue Listener类，实现队列监听
public function fire()
{
    $this->setListenerOptions();

    $delay = $this->input->getOption('delay');

    // The memory limit is the amount of memory we will allow the script to occupy
    // before killing it and letting a process manager restart it for us, which
    // is to protect us against any memory leaks that will be in the scripts.
    $memory = $this->input->getOption('memory');

    $connection = $this->input->getArgument('connection');

    $timeout = $this->input->getOption('timeout');

    // We need to get the right queue for the connection which is set in the queue
    // configuration file for the application. We will pull it based on the set
    // connection being run for the queue operation currently being executed.
    $queue = $this->getQueue($connection);

    $this->listener->listen(
        $connection, $queue, $delay, $memory, $timeout
    );
}
```

```php
//创建进程，并通过无限循环调用进程
public function listen($connection, $queue, $delay, $memory, $timeout = 60)
{
    $process = $this->makeProcess($connection, $queue, $delay, $memory, $timeout);

    while (true) {
        $this->runProcess($process, $memory);
    }
}
```

```php
//运行进程，回调函数表示可执行运行时输出，并且每次执行时验证是否超过内存限制
public function runProcess(Process $process, $memory)
{
    $process->run(function ($type, $line) {
        $this->handleWorkerOutput($type, $line);
    });

    // Once we have run the job we'll go check if the memory limit has been
    // exceeded for the script. If it has, we will kill this script so a
    // process manager will restart this with a clean slate of memory.
    if ($this->memoryExceeded($memory)) {
        $this->stop();
    }
}
```

```php
//队列实际运行的过程
public function run($callback = null)
{
    $this->start($callback);

    return $this->wait();
}
```

### php artisan queue:work
省略获取参数相关的代码

#### 第一步：获取任务前的处理
```php
如果有$daemon参数，则运行监听事件，否则只取出位于队首的元素执行
WorkCommand.php
protected function runWorker($connection, $queue, $delay, $memory, $daemon = false)
{
    $this->worker->setDaemonExceptionHandler(
        $this->laravel['Illuminate\Contracts\Debug\ExceptionHandler']
    );

    if ($daemon) {
        $this->worker->setCache($this->laravel['cache']->driver());

        return $this->worker->daemon(
            $connection, $queue, $delay, $memory,
            $this->option('sleep'), $this->option('tries')
        );
    }

    return $this->worker->pop(
        $connection, $queue, $delay,
        $this->option('sleep'), $this->option('tries')
    );
}
```

```php
workr pop方法的代码，取出下一个元素之后运行，完成之后调用sleep方法
Work.php
public function pop($connectionName, $queue = null, $delay = 0, $sleep = 3, $maxTries = 0)
{
    try {
        $connection = $this->manager->connection($connectionName);

        $job = $this->getNextJob($connection, $queue);

        // If we're able to pull a job off of the stack, we will process it and
        // then immediately return back out. If there is no job on the queue
        // we will "sleep" the worker for the specified number of seconds.
        if (! is_null($job)) {
            return $this->process(
                $this->manager->getName($connectionName), $job, $maxTries, $delay
            );
        }
    } catch (Exception $e) {
        if ($this->exceptions) {
            $this->exceptions->report($e);
        }
    } catch (Throwable $e) {
        if ($this->exceptions) {
            $this->exceptions->report(new FatalThrowableError($e));
        }
    }

    $this->sleep($sleep);

    return ['job' => null, 'failed' => false];
}
```

```php
队列容器采用redis时，$connection使用redis连接
没有指定queue名称时，取默认的连接default
如果指定队列名称，则用逗号分隔，按顺序取出所有数据
Worker.php
protected function getNextJob($connection, $queue)
{
    if (is_null($queue)) {
        return $connection->pop();
    }

    foreach (explode(',', $queue) as $queue) {
        if (! is_null($job = $connection->pop($queue))) {
            return $job;
        }
    }
}
```

```php
从redis中取出队列数据
1.首先将队列每次执行时检查所有的过期或执行中的任务，将其从有序集合中删除，同时重新推入到队列的末尾
2.再取出队列头部的数据，并从redis中删除，之后添加到名称为$queue:reserved的有序集合中，表示该条数据正在处理中，过期时间为当前时间+redis的过期时间
(每次检查时比较当前时间和这条数据的分值来判断是否过期，如果大于该分值，则表示已经过期，会进行过期处理，重新调回1步骤)
RedisQueue.php
public function pop($queue = null)
{
    $original = $queue ?: $this->default;

    $queue = $this->getQueue($queue);

    if (! is_null($this->expire)) {
        $this->migrateAllExpiredJobs($queue);
    }

    $job = $this->getConnection()->lpop($queue);

    if (! is_null($job)) {
        $this->getConnection()->zadd($queue.':reserved', $this->getTime() + $this->expire, $job);

        return new RedisJob($this->container, $this, $job, $original);
    }
}
```

处理异常数据
```php
移除redis键为$queue:delayed和$queue:reserved名称的数据
RedisQueue.php
protected function migrateAllExpiredJobs($queue)
{
    $this->migrateExpiredJobs($queue.':delayed', $queue);

    $this->migrateExpiredJobs($queue.':reserved', $queue);
}

取出所有过期的任务，如果数量大于0，则去掉，然后将任务推入到新的队列
$this->getConnection()为Predis\Client类，通过他启动事务，同时对redis的键为$queue:reserved进行监听，取出所有超时任务进行删除，并将这些任务推入到队列尾部

public function migrateExpiredJobs($from, $to)
{
    $options = ['cas' => true, 'watch' => $from, 'retry' => 10];

    $this->getConnection()->transaction($options, function ($transaction) use ($from, $to) {
        // First we need to get all of jobs that have expired based on the current time
        // so that we can push them onto the main queue. After we get them we simply
        // remove them from this "delay" queues. All of this within a transaction.
        $jobs = $this->getExpiredJobs(
            $transaction, $from, $time = $this->getTime()
        );

        // If we actually found any jobs, we will remove them from the old queue and we
        // will insert them onto the new (ready) "queue". This means they will stand
        // ready to be processed by the queue worker whenever their turn comes up.
        if (count($jobs) > 0) {
            $this->removeExpiredJobs($transaction, $from, $time);

            $this->pushExpiredJobsOntoNewQueue($transaction, $to, $jobs);
        }
    });
}

$from为队列的键，$time是当前时间，取出有序集合中分数最小值到分数为当前时间的所有数据

protected function getExpiredJobs($transaction, $from, $time)
{
    return $transaction->zrangebyscore($from, '-inf', $time);
}
protected function removeExpiredJobs($transaction, $from, $time)
{
    $transaction->multi();

    $transaction->zremrangebyscore($from, '-inf', $time);
}

protected function pushExpiredJobsOntoNewQueue($transaction, $to, $jobs)
{
    call_user_func_array([$transaction, 'rpush'], array_merge([$to], $jobs));
```

#### 第二步：处理任务
```php
回到取出队列数据后运行的代码
先验证是否设置了最大尝试次数，如果任务记录的尝试次数大于最大尝试次数，则
Work.php
public function process($connection, Job $job, $maxTries = 0, $delay = 0)
{
    if ($maxTries > 0 && $job->attempts() > $maxTries) {
        return $this->logFailedJob($connection, $job);
    }

    try {
        // First we will fire off the job. Once it is done we will see if it will
        // be auto-deleted after processing and if so we will go ahead and run
        // the delete method on the job. Otherwise we will just keep moving.
        $job->fire();

        $this->raiseAfterJobEvent($connection, $job);

        return ['job' => $job, 'failed' => false];
    } catch (Exception $e) {
        // If we catch an exception, we will attempt to release the job back onto
        // the queue so it is not lost. This will let is be retried at a later
        // time by another listener (or the same one). We will do that here.
        if (! $job->isDeleted()) {
            $job->release($delay);
        }

        throw $e;
    } catch (Throwable $e) {
        if (! $job->isDeleted()) {
            $job->release($delay);
        }

        throw $e;
    }
}
```

尝试次数过多处理

```php
记录失败日志，如果设置了failer处理器，则失败日志
Work.php
protected function logFailedJob($connection, Job $job)
{
    if ($this->failer) {
        $this->failer->log($connection, $job->getQueue(), $job->getRawBody());

        $job->delete();

        $job->failed();

        $this->raiseFailedJobEvent($connection, $job);
    }

    return ['job' => $job, 'failed' => true];
}

插入当前时间的一条失败记录，payload为任务体，记录名称、连接等信息
DatabaseFailedJobProvider.php

public function log($connection, $queue, $payload)
{
    $failed_at = Carbon::now();

    $this->getTable()->insert(compact('connection', 'queue', 'payload', 'failed_at'));
}

RedisJob.php
public function delete()
{
    parent::delete();

    $this->redis->deleteReserved($this->queue, $this->job);
}


由于上一步把每次处理的数据都从$queue中取出，放入$queue:reserved中，所以此处将该值从$queue:reserved中删除
RedisQueue

public function deleteReserved($queue, $job)
{
    $this->getConnection()->zrem($this->getQueue($queue).':reserved', $job);
}
取出job，转换成对象CallQueuedHandler，调用failed方法，传入的值为任务的实体(也就是自己写的队列类)
Job.php
public function failed()
{
    $payload = json_decode($this->getRawBody(), true);

    list($class, $method) = $this->parseJob($payload['job']);

    $this->instance = $this->resolve($class);

    if (method_exists($this->instance, 'failed')) {
        $this->instance->failed($this->resolveQueueableEntities($payload['data']));
    }
}

调用任务类的failed方法
CallQueuedHandler.php
public function failed(array $data)
{
    $handler = $this->dispatcher->resolveHandler($command = unserialize($data['command']));

    if (method_exists($handler, 'failed')) {
        call_user_func([$handler, 'failed'], $command);
    }
}
```

任务处理

```php
队列数据例如：
{
    "job": "Illuminate\\Queue\\CallQueuedHandler@call",
    "data": {
        "command": "O:24:\"App\\Commands\\TestCommand\":1:{s:12:\"\u0000*\u0000requestNo\";i:18;}"
    },
    "id": "P5nKwZs2LNSjOFBlVCPyvdK7jAAHCi3U",
    "attempts": 1
}
job为队列处理的程序，data为要处理的数据(自己写的队列文件)，attempts为处理次数，推入队列时即为1
```

```php
回到队列处理的process方法中，该方法调用fire()方法
RedisJob.php
public function fire()
{
    $this->resolveAndFire(json_decode($this->getRawBody(), true));
}

protected function resolveAndFire(array $payload)
{
    list($class, $method) = $this->parseJob($payload['job']);

    $this->instance = $this->resolve($class);

    $this->instance->{$method}($this, $this->resolveQueueableEntities($payload['data']));
}

通过@分割类和方法，调用CallQueuedHandler的call方法处理队列
protected function parseJob($job)
{
    $segments = explode('@', $job);

    return count($segments) > 1 ? $segments : [$segments[0], 'fire'];
}

调用call方法，取出队列任务$command，调用Dispatch类的dispatchNow方法处理任务
CallQueuedHandler.php
public function call(Job $job, array $data)
{
    $command = $this->setJobInstanceIfNecessary(
        $job, unserialize($data['command'])
    );

    $this->dispatcher->dispatchNow($command, function ($handler) use ($job) {
        $this->setJobInstanceIfNecessary($job, $handler);
    });

    if (! $job->isDeletedOrReleased()) {
        $job->delete();
    }
}

队列类继承了SelfHandling接口，可以直接执行handle方法，afterResolving匿名函数貌似没有执行
Dispatch.php

public function dispatchNow($command, Closure $afterResolving = null)
{
    return $this->pipeline->send($command)->through($this->pipes)->then(function ($command) use ($afterResolving) {
        if ($command instanceof SelfHandling) {
            return $this->container->call([$command, 'handle']);
        }

        $handler = $this->resolveHandler($command);

        if ($afterResolving) {
            call_user_func($afterResolving, $handler);
        }

        return call_user_func(
            [$handler, $this->getHandlerMethod($command)], $command
        );
    });
}
进入Container类的call方法，执行getMethodDependencies方法获取类的所有依赖，然后通过call_user_func_array函数调用[$command, 'handle']方法，参数为获取到的依赖对象
Container.php

public function call($callback, array $parameters = [], $defaultMethod = null)
{
    if ($this->isCallableWithAtSign($callback) || $defaultMethod) {
        return $this->callClass($callback, $parameters, $defaultMethod);
    }

    $dependencies = $this->getMethodDependencies($callback, $parameters);

    return call_user_func_array($callback, $dependencies);
}

通过反射获取匿名函数的所有依赖
Container.php
protected function getMethodDependencies($callback, array $parameters = [])
{
    $dependencies = [];

    foreach ($this->getCallReflector($callback)->getParameters() as $key => $parameter) {
        $this->addDependencyForCallParameter($parameter, $parameters, $dependencies);
    }

    return array_merge($dependencies, $parameters);
}
```

执行完成后删除

```php
回到$job->isDeletedOrReleased(),此处将队列任务删除

RedisJob.php
public function isDeletedOrReleased()
{
    return $this->isDeleted() || $this->isReleased();
}

RedisJob.php

public function delete()
{
    parent::delete();

    $this->redis->deleteReserved($this->queue, $this->job);
}
将任务从redis为$queue:reserved的列表中删除，此处队列正常处理即为完成
RedisQueue.php

public function deleteReserved($queue, $job)
{
    $this->getConnection()->zrem($this->getQueue($queue).':reserved', $job);
}
```

触发处理后的事件illuminate.queue.after
```php
激活illuminate.queue.after事件，参数为redis连接，任务，还有任务类
protected function raiseAfterJobEvent($connection, Job $job)
{
    if ($this->events) {
        $data = json_decode($job->getRawBody(), true);

        $this->events->fire('illuminate.queue.after', [$connection, $job, $data]);
    }
}

在AppServiceProvider.php中注册illuminate.queue.after事件
Queue::after(function ($connection, $job, $data) {
    // 写日志
});
```

#### 异常处理
```php
首先看process方法的异常，抛出异常后执行$job->release($delay)方法，RedisJob::delete()方法为上面分析的删除方法，此处会把$queue:reserved的值删除。也就是说队列会把正在处理的任务放到delayed队列中。
设置延迟时间为参数中的--delay参数，
RedisJob.php
public function release($delay = 0)
{
    parent::release($delay);

    $this->delete();

    $this->redis->release($this->queue, $this->job, $delay, $this->attempts() + 1);
}

设置当前任务的的attempts次数+1，同时添加到$queue:delayed的队列中，超时时间为当前时间+延迟秒数
RedisQueue.php
public function release($queue, $payload, $delay, $attempts)
{
    $payload = $this->setMeta($payload, 'attempts', $attempts);

    $this->getConnection()->zadd($this->getQueue($queue).':delayed', $this->getTime() + $delay, $payload);
}

pop方法的异常处理，发送消息通知
$this->exceptions->report($e);
```

疑问点：
如果redis处理过程中down掉，存在reserved中数据不会清掉