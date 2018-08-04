---
title: swoole的多进程通信
date: 2018-08-04 12:21:35
desc: swoole开启多个进程实现业务
tags: swoole, multiple process.
---

使用瓦力发布平台发布代码, 发现丫的, 越来越慢, 越来越慢，已经到了忍不了了的节奏了. 看了下原理, 它是单进程的把所有的代码`scp`(没有使用ansible)到对应的机器上。
自然想到的就是开启多个进程去出力吧，这样就有个问题，多个进程如何通信呢。

<!-- more -->

测试环境:
1. linux (centos6.5)
2. php(5.6.16)
3. swoole(1.7.22-rc1)

先写简单的测试demo

```php

use swoole_process as Process;

class MultipleProcess
{
    protected $worknum = 2; // 默认2个进程

    protected $processList = [];

    public function __construct($worknum = 2)
    {
        $this->worknum = $worknum;
    }

    public function longtimeTask()
    {
        sleep(3);
        $rand = rand(1, 100);
        if ($rand % 5 == 0) { // 模拟随机一个进程处理超时
            throw new \Exception('timeout error!', $rand);
        }
        return true;
    }

    public function run()
    {
        $self = $this;
        for($i = 0; $i < $this->worknum; $i++) {
            $process = new Process(function(Process $worker) use ($self) {
                try {
                    $result = $self->longtimeTask();
                    $worker->push('{"code": 0, "msg":"success"}'); // 将出力的结果放到队列中
                    $worker->exit(0);
                } catch(\Exception $e) {
                    $worker->push(sprintf('{"code": %d, "msg":"%s"}',$e->getCode(), $e->getMessage()));
                    $worker->exit(1);
                }
            });
            $process->name("php-demo-$i");
            $process->useQueue(); // 使用队列
            $pid = $process->start();
            $this->processList[$pid] = $process;
        }

        foreach($this->processList as $pid => $process) {
            Process::wait(); // 回收进程
            $result = $process->pop();
            var_dump("process: $pid result: $result");
            $resp = json_decode($result);
            if ($resp->code) {
                $process->freeQueue();
                throw new \Exception("000");
            }
        }
    }
}
$multiple = new MultipleProcess(10);
$multiple->run();
```

**PS: 亲测是可以的, 😄**

