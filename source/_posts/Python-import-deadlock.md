---
title: Python import 导致死锁
date: 2020-11-22 11:30:59
tags: " Python "
categories: " Python "
---

> **前言**
    记录一下之前帮同事解决的因为 import 导致的死锁问题

## 问题

问题表现: worker 启动之后并未消费 mq 中的任务,处于 hangs 的状态

问题环境:

- Python 版本为 2.7.5
- 执行流程
  - 上游通过接口创建任务,接口将任务发送至 mq 中
  - worker 通过死循环常驻消费队列
    - 通过 supervisor 运行入口程序
    - 入口程序通过环境变量决定运行的脚本

问题代码:

```Python
# 入口

import os
import importlib

def main():
    app = os.getenv('SELECTED_APP')
    if app:
        libs = importlib.import_module(app)
        # libs.main()
    else:
        raise Exception('invalid app given')

if __name__=='__main__':
    main()


# APP 程序

class C(Thread):
    def run(self):
        subprocess.Popen("ls")

c = C()
c.start()
c.join()

```

### 解决

分析的思路是

- 通过 `gdb` 的方式或者 `python -m trace -t entry.py` 的方式跟踪 hangs 的位置
- 通过打印的信息发现是死锁导致代码 hangs 

```bash
threading.py(294):         self.__lock.release()           # No state to save
threading.py(337):         try:    # restore state no matter what (e.g., KeyboardInterrupt)
threading.py(338):             if timeout is None:
threading.py(339):                 waiter.acquire()
```

- 往前追溯最后一次加锁的位置是:

```bash
videoin.py(7):         subprocess.Popen("ls")
 --- modulename: subprocess, funcname: __init__
subprocess.py(656):         _cleanup()
 --- modulename: subprocess, funcname: _cleanup
subprocess.py(461):     for inst in _active[:]:
..........
subprocess.py(1224):                         self.pid = os.fork()
threading.py(338):             if timeout is None:
threading.py(339):                 waiter.acquire()
threading.py(341):                     self._note("%s.wait(): got it", self)
 --- modulename: threading, funcname: _note
threading.py(64):             if self.__verbose:
threading.py(374):             self._acquire_restore(saved_state)
 --- modulename: threading, funcname: _acquire_restore
threading.py(297):         self.__lock.acquire()           # Ignore saved state
threading.py(623):             return self.__flag
threading.py(625):             self.__cond.release()
```

- 但是通过 `trace` 未能找到上一次加锁的位置,无奈之下只好通过 `gdb attach` 来分析(`gdb -p pID` 与 `gdb attach PID` 出来的信息不一样,这个不太清楚,通过 `gdb -p` 的方式没有发现有用的线索,无意间用 `attach` 才发现的端倪)

```bash
#0  sem_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/sem_wait.S:85
#1  0x00007faea159c735 in PyThread_acquire_lock (lock=0xbb50a0, waitflag=waitflag@entry=1) at /usr/src/debug/Python-2.7.5/Python/thread_pthread.h:323
#2  0x00007faea15812ed in _PyImport_AcquireLock () at /usr/src/debug/Python-2.7.5/Python/import.c:309
#3  0x00007faea15a7796 in posix_fork (self=<optimized out>, noargs=<optimized out>) at /usr/src/debug/Python-2.7.5/Modules/posixmodule.c:3846
```

通过 `gdb` 的信息发现 `subprocess` 通过创建新的进程来执行的命令,而创建新的进程调用了 `posix_fork`, 而在 `posix_for` 中触发了 `_PyImport_AcquireLock` 锁的获取,在查相关资料的时候了解是因为当我们在 `import` module 的时候会获取 `import_lock`,而在创建进程的时候也会获取,从而导致的死锁

((import 源码)[https://github.com/python/cpython/blob/v2.7.5/Python/import.c] 中可以看到 `_PyImport_AcquireLock` 的逻辑,注意看的时候切换 python 版本)

解决问题之后的代码是:

```Python
# 入口

import os
import importlib

def main():
    app = os.getenv('SELECTED_APP')
    if app:
        libs = importlib.import_module(app)
        libs.main()
    else:
        raise Exception('invalid app given')

if __name__=='__main__':
    main()


# APP 程序

class C(Thread):
    def run(self):
        subprocess.Popen("ls")

def main():
    c = C()
    c.start()
    c.join()

if __name__=='__main__':
    main()
```

## 总结

触发这样死锁问题的条件为:

在 import A 模块时,执行其最外层的代码产生新的线程,并在新的线程中存在 `import` 与 `os.fork` 的操作就会导致这样的死锁

教训:在单文件中最外层尽量不要放过多的操作与非必要的操作, 让 `import` 操作更纯粹不带有副作用


## 参考

- [奇怪的 dead lock](https://www.jianshu.com/p/fc5e4d4c3ef4)
- [import 模組或 fork 導致 deadlock](https://ephrain.net/python-%E5%9C%A8%E5%9F%B7%E8%A1%8C%E7%B7%92%E4%B8%AD-import-%E6%A8%A1%E7%B5%84%E6%88%96-fork-%E5%B0%8E%E8%87%B4-deadlock%EF%BC%9F/)
