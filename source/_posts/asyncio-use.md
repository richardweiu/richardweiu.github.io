---
title: asyncio 使用
date: 2019-08-04 18:48:31
tags: ["Python"]
categories: "Python"
---

> ** 前言 **
　为了提高 python 的效率提出了协程,然后出现了 monkey patch, 到后来的 asyncio,现在 python 3.7 中已经非常成熟了,今天小计一下使用方式,有时间再来解析其背后的原理


## 使用 asyncio 的例子

该例子的场景是这样的(只有请求接口部分用到了异步 io 的方式):

- 主入口传入一个 taskid - 功能函数会通过这个 taskid 获取到所有状态异常的任务
- 所有的任务都会通过这个任务信息请求接口获取其相关状态,然后更新其状态

```python
# 主入口
def check_ugc_retranscode(taskid, datatype):
    asyncio.run(update_ugc_status(taskid, datatype), debug=True)
    return 'check success'


async def update_ugc_status(taskid, datatype):
    '''
    通过 taskid 找到所有异常状态的任务信息
    '''
    # semaphore 必须在这里定义,并且通过参数的方式传进去
    semaphore = asyncio.Semaphore(100)
    tasks_list = []
    # connector = aiohttp.TCPConnector(limit_per_host=50)
    check_task = Data.objects.filter(task_id=taskid, status=5)
    for i in check_task:
        task = check_ugc_hvc(semaphore, taskid, i.content, datatype)
        tasks_list.append(task)
    return await asyncio.gather(*tasks_list)


async def check_ugc_hvc(semaphore, taskid, qipuid, datatype):
    '''
    通过接口获取信息,更新状态
    '''
    async with semaphore:
        result = ''
        count = 0
        while count < 100:
            async with aiohttp.ClientSession() as session:
                try:
                    async with session.get(QIPU_URL+str(qipuid), headers=auth_header, timeout=5) as response:
                        assert response.status == 200
                        result = await response.json(content_type=None)
                        # response.close()
                        break
                except asyncio.TimeoutError:
                    logger.error('is timeout %s' % (qipuid))
                    await asyncio.sleep(1)
                    count += 1
        if not result:
            return False, ''
    '''
    一堆逻辑判断
    '''
    Data.objects.filter(task_id=taskid, status=5).update(status=1)
    return True result
```

## 说明

1. asyncio.run 使用

asyncio.run 是 python3.7 中才有的功能函数,此函数运行传入的协程，负责管理 asyncio 事件循环并 完结异步生成器。(摘自[官网](https://docs.python.org/zh-cn/3/library/asyncio-task.html#asyncio.run))

简单来说就用来帮助管理我们 event loop,不用像以前一样手动创建与等待结束,例如:

```python
loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

2. 并发的限制

必须要有并发的限制,否则在任务多的时候会出问题,比如:

- OSError: [Errno 24] Too many open files(比如:[错误示范](https://www.v2ex.com/t/439254),这是一个限制使用错误的例子,看一下报错见识一下)
    - 虽然多开 linux 的文件句柄能够增加同时开启的链接数量,但是核心还是并发限制才对,毕竟:
        - 别人的服务自己存在限流,差一点的可能支撑不住干崩了
        - 系统的承受能力也是有限的,你也不可能开到无限大的

为了限制并发,我们会有两种方式:

- asyncio 的 semaphore(我们主要使用这种方式)
- aiohttp 的 limit_per_host

我们主要是使用的是 semaphore,asyncio 的同步机制设计的与 python 多线程机制很像,也可以使用锁与释放锁的方式,而 semaphore 就是一个帮助我们管理锁的一个对象,内部有一个计数器来控制,从而来限制并发数,例如

```python
semaphore = asyncio.Semaphore(100)
```

我们限制的并发数是 100,等于是有 100 个锁,首先拿到这 100 个锁权利的就会直接开始,在他们还没有释放之前,其他的任务是会被阻塞调,释放一个就会有空位置,剩下的任务谁拿到谁就可以开始.

而 aiohttp 是一个第三方的包, 可以通过这个 aiohttp.TCPConnector 的如下参数来控制 aiohttp 的并发:

- limit: The total number of simultaneous connections.
- limit_per_host: Number of simultaneous connections to one host.

```python
conn=aiohttp.TCPConnector(limit=100)#防止ssl报错
async with aiohttp.ClientSession(connector=conn) as session: #创建session
```

通过源码可以看到其是通过相关的链接或者链接 host 放到 set 中来限制的([aiohttp connector 部分](https://github.com/aio-libs/aiohttp/blob/889b186daac0df8e75908cb43a555f4938c62f63///aiohttp/connector.py#L149:7))


3.semaphore 的踩坑

semaphore 不能像多线程一般将其放置于全局定义,只能在其使用的上层去定义并且将其传入相关的功能函数中,否则就会有 RuntimeError 的报错:

```python
RuntimeError: Task <Task pending coro=<check_ugc_hvc() running at /utils.py:504> cb=[gather.<locals>._done_callback() at /usr/lib/python3.7/asyncio/tasks.py:664] created at /usr/lib/python3.7/asyncio/tasks.py:719> got Future <Future pending> attached to a different loop 
```

这个是因为 semaphore 在初始化的时候会获取判断有没有 event loop,若是没有会创建一个,而 asyncio.run 在启动的时候会创建一个 event_loop,所以就导致了他们根本不是处理的一个 loop 所以有这个报错(参考[解答](https://www.oipapio.com/question-4921406), 这个还是开启了 asyncio.run 的 debug 才看到的)

4.还遇到这个报错 Cannot write to closing transport

```python
aiohttp.client_exceptions.ClientOSError: Cannot write to closing transport
```

这个主要是 client 端关闭调了链接,但是 server 端还在响应导致的,主要就是注意 session.close() 使用的位置



## 小结

这就是当时初次使用的时候遇到的一些问题,主要还是因为没有哦深刻理解就乱使用导致的,因为总结的比较晚所以其他犯的低级错已经忘记了

## 参考
