---
title: 'SSH Authentication Refused: Bad Ownership'
date: 2017-10-26 12:22:30
tags: [" SSH "," Linux "]
categories: " 运维 "
---

> ** 前言 **
　今天无意之中触发了一个小问题，长见识了，将其他用户加入当前用户组导致 ssh Permission denied (publickey)。

## 问题

为了图方便，我将 jenkins 加入我当前的用户组中，然后神奇的事情发生了，我再也登录不上去了，ssh 返回：

```
Permission denied (publickey)
```

检查了 ssh 的 config 没有问题，检查了 server 端的 authorized_keys 里面也没有写错，但是 ssh 始终失败，debug 信息是：

```
debug1: identity file /xxx/xxxx/xxxx type 1
debug1: key_load_public: No such file or directory
```

## 解决

通过 console 登录到机器，查看 `/var/log/auth.log` 中 ssh 的日志：

```
Oct 26 12:17:45 xxxxx sshd[4800]: Authentication refused: bad ownership or modes for file /home/xxxxx/.ssh/authorized_keys
Oct 26 12:17:45 xxxxx sshd[4800]: Connection closed by 118.112.59.59 [preauth]
```

这个报错的原因是 `~/.ssh/authorized_keys` 这个文件只能被当前的用户所修改，若是其他用户加入到该用户组中，并且 group 有修改该文件的权限，就会曝出这样的错误，若是想解决可以：

```
chmod g-w authorized_keys
```

或者 `sudo gpasswd -d jenkins xxxxx` 将组中的外来人员删掉就可以解决这个问题了。

## 参考

- [Dave Perrett blog](https://www.daveperrett.com/articles/2010/09/14/ssh-authentication-refused/)
