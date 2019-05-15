title: understand-ssh-port-forwarding
date: 2018-02-25 02:33:07
tags: network shell
---

以前我对ssh端口转发命令的格式一直没理解，最近看了[一篇博客](https://blog.fundebug.com/2017/04/24/ssh-port-forwarding/)很有启发，琢磨了一下总算是弄明白了。

让我就尝试用最通俗易懂的语言解释给你听什么是ssh端口转发以及如何运用这个命令。

## 回顾ssh连接中的一些概念

ssh连接可以双向

假设你有一台笔记本，hostname是laptop，同时你还有一台ISP主机服务器，hostname是isp。

笔记本电脑是“本地”，称ISP主机是“远程”。

当你在笔记本上执行 `ssh root@isp` 成功登陆ISP主机后，一个ssh连接便建立起来了。

这是一条双向的连接，意味着


