---
title: Python subprocess 导致僵尸进程问题
date: 2020-08-12 09:27:07
tags: "Python"
categories: "Python"
---

> **前言**
　记一次针对线上发现僵尸进程的问题

## 问题

- 现象: 生产环境中发现 worker 被 hangs 并且同时存在僵尸进程与孤儿进程的情况
- 环境: 产生问题的环境描述:
  - 运行环境： 在 docker 的 container 中
  - 调用逻辑： 
    - worker 通过死循环从 mq 中获取任务（任务信息中包含执行的 Command）
    - worker 通过 subprocess popen 的方式调用 Command(command 是一个 python 脚本)
    - 执行 Command 中会起单独的进程来运行其他团队的模块，主进程会通过适当的超时机制来终止子进程（防止卡主）
      - 超时机制：
        - 最终超时时间: time.time() + 限制时间
        - 主进程轮询检查:
          - 若是达到预计超时时间则判断为超时，并 terminate 子进程
          - 若是没有达到预计超时时间则去 multiprocessing.queue 中获取执行结果

超时前的进程树情况：

```bash
268142 python qae_service/entry.py（入口程序）
268151  \_ python qae_service/entry.py（因为 import 其他团队模块，该模块初始化会创建多进程）
268152  \_ python qae_service/entry.py
268156  \_ python qae_service/entry.py
268158  \_ python qae_service/entry.py
268163  \_ python qae_service/entry.py
268169  \_ python qae_service/entry.py
268176  \_ python qae_service/entry.py
268184  \_ python qae_service/entry.py
268190  \_ python qae_service/entry.py
268201  \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding（执行 mq 获取到的 cmd）
268210      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding（import 其他团队模块，初始化会创建多进程）
268211      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding
268215      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding
268217      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding
268222      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding
268228      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding
268235      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding
268243      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding
268249      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding
268300      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding（主进程起的子进程）
268307          \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding（其他团队模块，运行进程）
268472          \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding（其他团队模块，运行进程）
268486              \_ /bin/sh -c /usr/local/vtc/tools/vtcoder_new -v -i /data/tmp/xxx.mkv
268487                  \_ /usr/local/vtc/tools/vtcoder_new -v -i /data/tmp/xxx.mkv
```

超时后的进程树情况：

```bash
268142  S python qae_service/entry.py
268151  S  \_ python qae_service/entry.py
268152  S  \_ python qae_service/entry.py
268156  S  \_ python qae_service/entry.py
268158  S  \_ python qae_service/entry.py
268163  S  \_ python qae_service/entry.py
268169  S  \_ python qae_service/entry.py
268176  S  \_ python qae_service/entry.py
268184  S  \_ python qae_service/entry.py
268190  S  \_ python qae_service/entry.py
268201  Z  \_ [python] <defunct>（僵尸进程）
268307  S python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding（孤儿进程，父进程为 1）
```

## 解决

- 首先僵尸进程因为资源清理了所以没有办法查看其中信息
- 通过 gdb python 查看孤儿进程里面的相关信息，主要是卡在多进程的 socket 监听
- 僵尸进程产生的主要原因是子进程退出时父进程并没有处理子进程的退出信号，所以主要问题还在主进程上
- 通过 gdb python 查看 268142 进程发现主要卡在这么一句无用的语句上：
  - 无用的判断来自于：error_info 在上下文中并没有用到

```python
error_info = popen.stderr.read()
```

而卡主的原因（来自python [官方文档](https://docs.python.org/zh-cn/3/library/subprocess.html#subprocess.Popen.communicate)）：

```shell
警告 使用 communicate() 而非 .stdin.write， .stdout.read 或者 .stderr.read 来避免由于任意其他 OS 管道缓冲区被子进程填满阻塞而导致的死锁。
```

所以删掉该语句即可解决问题

## 总结

虽然解决方法很简单，但是其中可以总结的知识点不少：

- gdb python 的调试方式:
  - 通过[官方使用文档](https://wiki.python.org/moin/DebuggingWithGdb) 可以获取使用方式
  - `py-bt`、`py-list` 获取到出问题的上下文，非常的方便
- 巩固僵尸进程与孤儿进程的知识点:
  - 僵尸进程:子进程退出但是父进程并没有调用 wait 或 waitpid 获取子进程的状态信息时就会产生僵尸进程
  - 孤儿进程:当父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。
- Python 中 subprocess.popen:
  - 在 Linux 与 Windows 中的执行方式有些许差别
    - 在 Linux 中是通过 exec 来执行相关的命令，而参数 shell 的作用仅仅只是改变 cmd 的方式（是 string 或是 list）
    - 在 Windows 中执行方式与 shell 参数的功能也是不同的:
      - 运行方式是通过 `_subprocess.CreateProcess` 来执行，因为 Windows 缺少`bash exec`的工作方式, 后期 Windows `_execv()` 在 MSVCRT 中确实具有其POSIX兼容性层，但是AFAIK（尚未对其进行测试）只是一个包装
      - shell 参数主要是用于判断是否用 cmd （对于权限会有一定的影响）

```python

# Linux 中的逻辑
def _execute_child(self, args, executable, preexec_fn, close_fds,
        cwd, env, universal_newlines,
        startupinfo, creationflags, shell, to_close,
        p2cread, p2cwrite,
        c2pread, c2pwrite,
        errread, errwrite):
    """Execute program (POSIX version)"""

    if isinstance(args, types.StringTypes):
        args = [args]
    else:
        args = list(args)

    if shell:
        args = ["/bin/sh", "-c"] + args
        if executable:
            args[0] = executable

    if executable is None:
        executable = args[0]

    def _close_in_parent(fd):
        os.close(fd)
        to_close.remove(fd)
    
    """
    下面的细节就不在展示了
    就是 fork，然后用 os.execvp 或者 os.execvpe 执行具体的命令的过程
    """

# 在 Windows 上的逻辑
def _execute_child(self, args, executable, preexec_fn, close_fds,
                    cwd, env, universal_newlines,
                    startupinfo, creationflags, shell, to_close,
                    p2cread, p2cwrite,
                    c2pread, c2pwrite,
                    errread, errwrite):
    """Execute program (MS Windows version)"""

    if not isinstance(args, types.StringTypes):
        args = list2cmdline(args)

    # Process startup details
    if startupinfo is None:
        startupinfo = STARTUPINFO()
    if None not in (p2cread, c2pwrite, errwrite):
        startupinfo.dwFlags |= _subprocess.STARTF_USESTDHANDLES
        startupinfo.hStdInput = p2cread
        startupinfo.hStdOutput = c2pwrite
        startupinfo.hStdError = errwrite

    if shell:
        startupinfo.dwFlags |= _subprocess.STARTF_USESHOWWINDOW
        startupinfo.wShowWindow = _subprocess.SW_HIDE
        comspec = os.environ.get("COMSPEC", "cmd.exe")
        args = '{} /c "{}"'.format (comspec, args)
        if (_subprocess.GetVersion() >= 0x80000000 or
                os.path.basename(comspec).lower() == "command.com"):
            # Win9x, or using command.com on NT. We need to
            # use the w9xpopen intermediate program
            w9xpopen = self._find_w9xpopen()
            args = '"%s" %s' % (w9xpopen, args)
            creationflags |= _subprocess.CREATE_NEW_CONSOLE

    def _close_in_parent(fd):
        fd.Close()
        to_close.remove(fd)

    # Start the process
    try:
        hp, ht, pid, tid = _subprocess.CreateProcess(executable, args,
                                    # no special security
                                    None, None,
                                    int(not close_fds),
                                    creationflags,
                                    env,
                                    cwd,
                                    startupinfo)
    except pywintypes.error, e:
        raise WindowsError(*e.args)
    finally:
        if p2cread is not None:
            _close_in_parent(p2cread)
        if c2pwrite is not None:
            _close_in_parent(c2pwrite)
        if errwrite is not None:
            _close_in_parent(errwrite)

    # Retain the process handle, but close the thread handle
    self._child_created = True
    self._handle = hp
    self.pid = pid
    ht.Close()
```

## 参考

- [僵尸/孤儿进程知识点](https://zhuanlan.zhihu.com/p/96098130)