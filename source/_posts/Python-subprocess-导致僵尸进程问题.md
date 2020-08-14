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
      - 执行 cmd 的主要方式： `popen = subprocess.Popen(cmd, stdin=DEVNULL, stdout=DEVNULL, stderr=subprocess.PIPE, shell=True, close_fds=False)`
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
268300      \_ python manage/Serverproxy.py 5464866:1362135226075152384 Transcoding（执行 cmd 进程起的子进程）
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

### 1.分析祖先进程卡主原因

- 首先僵尸进程因为资源清理了所以没有办法查看其中信息
- 通过 gdb python 查看孤儿进程里面的相关信息，主要是卡在多进程的 socket 监听
- 僵尸进程产生的主要原因是子进程退出时父进程并没有处理子进程的退出信号，所以主要问题还在主进程上
- 通过 gdb python 查看 268142 进程发现主要卡在这么一句无用的语句上：
  - 无用的判断来自于：
    - error_info 在上下文中并没有用到
    - `logging.StreamHandler()` 默认输出至 `sys.stderr`，而日志相关的内容本来就会落盘上传

```python
popen = subprocess.Popen(cmd, stdin=DEVNULL, stdout=DEVNULL, stderr=subprocess.PIPE, shell=True, close_fds=False)
error_info = popen.stderr.read()  # 卡主的位置
```

而卡主的可能原因（来自python [官方文档](https://docs.python.org/zh-cn/3/library/subprocess.html#subprocess.Popen.communicate)）：

```shell
警告 使用 communicate() 而非 .stdin.write， .stdout.read 或者 .stderr.read 来避免由于任意其他 OS 管道缓冲区被子进程填满阻塞而导致的死锁。
```

所以在删掉该语句之后发现解决了僵尸进程的问题，但是孤儿还在，所以还有问题还未得到答案:

- 为什么会存在孤儿？
- 为什么会导致 `popen.stderr.read()` hangs， 即使替换成 `communicate` 也会 hangs，并不能解决问题呢？

### 2.分析僵尸、孤儿进程存在原因

代码中 `Popen` 初始化时 `stderr =subprocess.PIPE`,通过 python 的源码中可以看到当参数 `stderr == PIPE` 时的操作是：

```python
# 源码位置：https://github.com/python/cpython/blob/2.7/Lib/subprocess.py 第 817 行
elif stdout == PIPE:
    c2pread, c2pwrite = self.pipe_cloexec()
    to_close.update((c2pread, c2pwrite))
```

根据这个线索我们通过 `lsof` 看到主进程 268142、其子进程以及残留孤儿进程都在对着这个管道（stderr PIPE）读或写，而孤儿进程连接着 `stderr` 应该是写日志导致的

同时只要将孤儿进程 `kill`, 就会发现主进程就不会被 hangs，就能够成功结束就 `stderr` 的读取，程序继续往下走，看来主要问题不在 `stderr` 死锁而是 `popen.stderr.read()` 一直在等 `EOF` 表示文件结束输入关闭的标示，所以僵尸、孤儿进程存在原因：

- 执行 cmd 的进程（268201）在终止了其子进程，但是其孙子中的一个进程并没有处理信号，依然在运行中，所以孙子变成孤儿
- 因为孤儿还在以 write 的身份开着管道，所以启动 cmd 的进程认为写入还没有结束所以导致卡主
- 而启动 cmd 的进程卡在了 `stderr.read` 上所以没有处理执行 cmd 的进程退出信号，所以僵尸进程出现

追究根本原因的处理应该是：

- 父进程在 terminate 子进程之后要确保所有进程都成功杀死结束退出。
- 子进程需要有处理终止的信号

### 3.分析子进程没有杀死原因

通过当前代码看杀死子进程的方式：

```python
def cancel(self):
    """Terminate any possible execution of the embedded function."""
    if self.__process.is_alive():
        self.__process.terminate()

    _raise_exception(self.__timeout_exception, self.__exception_message)
```

通过 Python 源码我们查到此处的 `terminate()` 是在 `multiprocessing/forking.py` 中实现的：

```python
def terminate(self):
    if self.returncode is None:
        try:
            os.kill(self.pid, signal.SIGTERM)
        except OSError, e:
            if self.wait(timeout=0.1) is None:
                raise
```

通过官网看到对于 `SIGTERM` 的说明：

```bash
Macro: int SIGTERM
The SIGTERM signal is a generic signal used to cause program termination. Unlike SIGKILL, this signal can be blocked, handled, and ignored. It is the normal way to politely ask a program to terminate.

The shell command kill generates SIGTERM by default.
```

通过实践我们发现 `kill` 命令也就是 `SIGTERM` 信号只有指定的进程收到信号，子进程不会收到。所以其子进程变成孤儿

由此解决孤儿进程的方式就应该是(杀死之前将其子孙都逐一干掉):

```python
def cancel(self):
    """Terminate any possible execution of the embedded function."""
    if self.__process.is_alive():
        pro = psutil.Process(self.__process.pid)
        child_process = pro.children(recursive=True)
        self.__process.terminate()
        for c in child_process:
            c.terminate()
        if self.__process.is_alive():
            self.__process.join()
        time.sleep(1)
    _raise_exception(self.__timeout_exception, self.__exception_message)
```

## 总结

虽然解决方法很简单，但是其中可以总结的知识点不少：

### 1.gdb python 的调试方式
  
- 通过[官方使用文档](https://wiki.python.org/moin/DebuggingWithGdb) 可以获取使用方式
- `py-bt`、`py-list` 获取到出问题的上下文，非常的方便

### 2.巩固僵尸进程与孤儿进程的知识点

- 僵尸进程:子进程退出但是父进程并没有调用 wait 或 waitpid 获取子进程的状态信息时就会产生僵尸进程
- 孤儿进程:当父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。

### 4.终止信号量

- SIGTERM: 
  - 等同于没有带参数的 `kill` 命令，这个信号可以被捕获和解释（或忽略）
  - 只会作用于当前进程，不会影响其子进程
- SIGINT: 
  - Linux `ctrl+c` 等同使用该信号
  - 信号当前的进程树接会收到，也就是说，不仅当前进程会收到信号，它的子进程也会收到（[来源](https://blog.csdn.net/alibo2008/article/details/6162970),实践证明是，但是没有找到官方说明）
- SIGKILL: 
  - 用于立即终止程序，它不能被处理或忽略，也不可能阻塞该信号，如果杀死失败的话估计已经产生操作系统级别错误了
  - 等同于 `kill -9`

### 3.Python 中 subprocess.popen

- communicate 避免死锁的处理方式是在读取 PIPE 的外套一层捕获异常：

```python
def _eintr_retry_call(func, *args):
    while True:
        try:
            return func(*args)
        except (OSError, IOError) as e:
            if e.errno == errno.EINTR:
                continue
            raise

def communicate(self, input=None):
    """Interact with process: Send data to stdin.  Read data from
    stdout and stderr, until end-of-file is reached.  Wait for
    process to terminate.  The optional input argument should be a
    string to be sent to the child process, or None, if no data
    should be sent to the child.
    communicate() returns a tuple (stdout, stderr)."""

    # Optimization: If we are only using one pipe, or no pipe at
    # all, using select() or threads is unnecessary.
    if [self.stdin, self.stdout, self.stderr].count(None) >= 2:
        stdout = None
        stderr = None
        if self.stdin:
            if input:
                try:
                    self.stdin.write(input)
                except IOError as e:
                    if e.errno != errno.EPIPE and e.errno != errno.EINVAL:
                        raise
            self.stdin.close()
        elif self.stdout:
            stdout = _eintr_retry_call(self.stdout.read)
            self.stdout.close()
        elif self.stderr:
            stderr = _eintr_retry_call(self.stderr.read)
            self.stderr.close()
        self.wait()
        return (stdout, stderr)

    return self._communicate(input)
```

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
- [centos7 gdb python](https://blog.csdn.net/github_40094105/article/details/81287572)
- [找到指定版本的 python debug 包](http://debuginfo.centos.org/7/x86_64/)
- [GUN signal 说明](https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html)