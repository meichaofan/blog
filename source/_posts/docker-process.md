---
title: Docker:从进程说起
date: 2019-04-12 23:54:23
tags: 
- Docker
- 容器
---

## 进程

进程是运行的程序，一旦“程序”被执行起来，它就从磁盘上的二进制文件，变成了计算机内存中的数据、寄存器里的值、堆栈中的指令、被打开的文件，以及各种设备的状态信息的一个集合。像这样一个程序运起来后的计算机执行环境的总和，就是**进程**。


## Namespace

而容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个“**边界**”。

对于 Docker 等大多数 Linux 容器来说，**Cgroups** 技术是用来制造约束的主要手段，而**Namespace** 技术则是用来修改进程视图的主要方法。

假设你已经有了一个 Linux 操作系统上的 Docker 项目在运行，比如我的环境是 Ubuntu 16.04 和 Docker CE 18.05。

接下来，让我们创建一个容器试试。

```bash
$ docker run -it busybox /bin/bash
/ #
```

这就是大名鼎鼎的`docker run`。-it参数告诉Docker项目在启动的时候分配一个输入/输出环境，也就是TTY，跟容器的标准输入相关联，这样我们就可以和这个容器交互了。而/bin/bash就是在这个容器里运行的程序。

所以，上面这条指令翻译成人类的语言就是：请帮我启动一个容器，在容器里执行 /bin/sh，并且给我分配一个命令行终端跟这个容器交互。

在容器里执行`ps`，会出现如下信息：

```bash
/ # ps
PID  USER   TIME COMMAND
  1 root   0:00 /bin/sh
  10 root   0:00 ps
```

我们可以看到，容器启动时执行的/bin/bash，就是这个容器里第1号进程（PID=1）,当前，容器里有两个进程在运行。这两个进程，已经被Docker隔离在了一个跟宿主主机完全不同的世界当中。

那么，这究竟是这么做到的呢？

本来，当我们在宿主主机执行`/bin/bash`,操作系统会给它分配一个进程编号，比如PID=100。而现在，我们通过Docker把`/bin/bash`运行在容器当中。这时候，Docker就会在这个进程启动时，给它实时一个“障眼法”，让它永远也看不到前面的**99**个进程，这里，它就会误认为自己是第1号进程了。

这种机制，其实就是对被隔离应用的进程空间做了手脚，使得这些进程只能看到重新计算过的进程编号，比如 PID=1。可实际上，他们在宿主主机的操作系统中，还是原来的第100号进程。

这种技术，就是Linux中的**Namespace**机制。而Namespace的使用方式也非常有意思：它其实只是Linux创建新进程的一个可选参数。我们知道，在Linux系统中创建线程的系统调用是clone(),比如：

```bash
int pid = clone(main_function, stack_size, SIGCHLD, NULL); 
```

这个系统调用就会为我们创建一个新的进程，并且返回它的进程号 pid。
而当我们用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数，比如：

```bash
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
```

这时，新创建的这个进程将会“看到”一个全新的进程空间，在这个进程空间里，它的 PID 是 1。之所以说“看到”，是因为这只是一个“障眼法”，在宿主机真实的进程空间里，这个进程的 PID 还是真实的数值，比如 100。

而除了刚刚这种PID Namespace，Linux操作系统还提供了**Mount**、**UTS**、**IPC**、**Network**和**User** 这些Namespace，用来对各种不同的进程上下文进行“障眼法”操作。

这就是Linux容器最基本的实现原理了。所以说，**容器只是一种特殊的进程而已**，在创建该容器进程时，指定了这个进程所需要启用的一组Namespace参数。这样，容器就只能“看”到当前Namespace所限定的资源、文件、设备、状态或者配置。而宿主主机以及其他不想关的程序，它就完全看不到了。

## CGroup

除了**Namespace**对容器环境进行隔离，还通过**Linux CGroup**限制容器进程使用相关资源

Linux Cgroups 的全称是 Linux Control Group。它最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。

在 Linux 中，Cgroups 给用户暴露出来的操作接口是文件系统，即它以文件和目录的方式组织在操作系统的 /sys/fs/cgroup 路径下。在 Ubuntu 16.04 机器里，我可以用 mount 指令把它们展示出来，这条命令是：

```bash
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
......
```

可以看到，在 /sys/fs/cgroup 下面有很多诸如 cpuset、cpu、 memory 这样的子目录，也叫子系统。这些都是我这台机器当前可以被 Cgroups 进行限制的资源种类。而在子系统对应的资源种类下，你就可以看到该类资源具体可以被限制的方法。比如，对 CPU 子系统来说，我们就可以看到如下几个配置文件，这个指令是：

```bash
$ ls /sys/fs/cgroup/cpu
cgroup.clone_children cpu.cfs_period_us cpu.rt_period_us  cpu.shares notify_on_release
cgroup.procs      cpu.cfs_quota_us  cpu.rt_runtime_us cpu.stat  tasks
```

如果熟悉 Linux CPU 管理的话，你就会在它的输出里注意到 cfs_period 和 cfs_quota 这样的关键词。这两个参数需要组合使用，可以用来限制进程在长度为 cfs_period 的一段时间内，只能被分配到总量为 cfs_quota 的 CPU 时间。

而这样的配置文件又如何使用呢？
你需要在对应的子系统下面创建一个目录，比如，我们现在进入 /sys/fs/cgroup/cpu 目录下：

```bash
root@ubuntu:/sys/fs/cgroup/cpu$ mkdir container
root@ubuntu:/sys/fs/cgroup/cpu$ ls container/
cgroup.clone_children cpu.cfs_period_us cpu.rt_period_us  cpu.shares notify_on_release
cgroup.procs      cpu.cfs_quota_us  cpu.rt_runtime_us cpu.stat  tasks
```

这个目录就称为一个“控制组”。你会发现，操作系统会在你新创建的 container 目录下，自动生成该子系统对应的资源限制文件。

现在，我们在后台执行这样一条脚本：

```bash
$ while : ; do : ; done &
[1] 226
```

显然，它执行了一个死循环，可以把计算机的 CPU 吃到 100%，根据它的输出，我们可以看到这个脚本在后台运行的进程号（PID）是 226。
这样，我们可以用 top 指令来确认一下 CPU 有没有被打满：

```bash
$ top
%Cpu0 :100.0 us, 0.0 sy, 0.0 ni, 0.0 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
```

在输出里可以看到，CPU 的使用率已经 100% 了（%Cpu0 :100.0 us）。
而此时，我们可以通过查看 container 目录下的文件，看到 container 控制组里的 CPU quota 还没有任何限制（即：-1），CPU period 则是默认的 100 ms（100000 us）：

```bash
$ cat /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us 
-1
$ cat /sys/fs/cgroup/cpu/container/cpu.cfs_period_us 
100000
```

接下来，我们可以通过修改这些文件的内容来设置限制。
比如，向 container 组里的 cfs_quota 文件写入 20 ms（20000 us）：

```bash
$ echo 20000 > /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us
```

结合前面的介绍，你应该能明白这个操作的含义，它意味着在每 100 ms 的时间里，被该控制组限制的进程只能使用 20 ms 的 CPU 时间，也就是说这个进程只能使用到 20% 的 CPU 带宽。
接下来，我们把被限制的进程的 PID 写入 container 组里的 tasks 文件，上面的设置就会对该进程生效了：

```bash
$ echo 226 > /sys/fs/cgroup/cpu/container/tasks 
```

我们可以用 top 指令查看一下：

```bash
$ top
%Cpu0 : 20.3 us, 0.0 sy, 0.0 ni, 79.7 id, 0.0 wa, 0.0 hi, 0.0 si, 0.0 st
```

可以看到，计算机的 CPU 使用率立刻降到了 20%（%Cpu0 : 20.3 us）。

除 CPU 子系统外，Cgroups 的每一项子系统都有其独有的资源限制能力，比如：

* blkio，为​​​块​​​设​​​备​​​设​​​定​​​I/O 限​​​制,一般用于磁盘等设备；
* cpuset，为进程分配单独的 CPU 核和对应的内存节点；
* memory，为进程设定内存使用的限制。

Linux Cgroups 的设计还是比较易用的，简单粗暴地理解呢，它就是一个子系统目录加上一组资源限制文件的组合。而对于 Docker 等 Linux 容器项目来说，它们只需要在每个子系统下面，为每个容器创建一个控制组（即创建一个新目录），然后在启动容器进程之后，把这个进程的 PID 填写到对应控制组的 tasks 文件中就可以了。
而至于在这些控制组下面的资源文件里填上什么值，就靠用户执行 docker run 时的参数指定了，比如这样一条命令：

```bash
$ docker run -it --cpu-period=100000 --cpu-quota=20000 ubuntu /bin/bash
```

在启动这个容器后，我们可以通过查看 Cgroups 文件系统下，CPU 子系统中，“docker”这个控制组里的资源限制文件的内容来确认：

```bash
$ cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_period_us 
100000
$ cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_quota_us 
20000
```

这就意味着这个 Docker 容器，只能使用到 20% 的 CPU 带宽。

## 总结

一个正在运行的 Docker 容器，其实就是一个启用了多个 Linux Namespace 的应用进程，而这个进程能够使用的资源量，则受 Cgroups 配置的限制。
