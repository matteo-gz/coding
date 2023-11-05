---
title: k8s
---
> [极客时间.深入剖析 Kubernetes](https://time.geekbang.org/column/intro/100015201)
## 容器基础

容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个"边界".

- Cgroups 技术// 制造约束
- Namespace 技术//修改进程视图

### Namespace
> 用来对各种不同的进程上下文进行“障眼法”操作
- PID
- Mount
- UTS
- IPC
- Network
- User
- Cgroup // Linux 内核从 4.6 开始

### PID namespace
> 对被隔离应用的进程空间做了手脚，使得这些进程只能看到重新计算过的进程编号，比如 PID=1。可实际上，他们在宿主机的操作系统里，还是原来的第 100 号进程

```c
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
```
> 多次执行上面的 clone(),就会创建多个 PID Namespace,每个 Namespace 里的应用进程，都会认为自己是当前容器里的第 1 号进程，它们既看不到宿主机里真正的进程空间，也看不到其他 PID Namespace 里的具体情况
### 障眼法
Docker 项目帮助用户启动的，还是原来的应用进程，只不过在创建这些进程时，
Docker 为它们加上了各种各样的 Namespace 参数。

这时，这些进程就会觉得自己是各自 PID Namespace 里的第 1 号进程，
只能看到各自 Mount Namespace 里挂载的目录和文件，
只能访问到各自 Network Namespace 里的网络设备，
就仿佛运行在一个个“容器”里面，与世隔绝。
> Namespace 技术实际上修改了应用进程看待整个计算机“视图”，即它的“视线”被操作系统做了限制，只能“看到”某些指定的内容

"敏捷"和"高性能"是容器相较于虚拟机最大的优势.

### 隔离得不彻底
> 容器只是运行在宿主机上的一种特殊的进程，那么多个容器之间使用的就还是同一个宿主机的操作系统内核

**行不通**
- Windows 宿主机上运行 Linux 容器
- 或者在低版本的 Linux 宿主机上运行高版本的 Linux 容器

在 Linux 内核中，有很多资源和对象是**不能被Namespace化**的

比如**时间**.如果你的容器中的程序使用 settimeofday(2) 系统调用修改了时间，整个宿主机的时间都会被随之修改

### Linux Cgroups
> Linux 内核中用来为进程设置资源限制的一个重要功能

Linux Cgroups 的全称是 Linux Control Group。
它最主要的作用，就是限制一个进程组能够使用的**资源上限**，包括 CPU、内存、磁盘、网络带宽等等.

Linux Cgroups 的设计还是比较易用的，简单粗暴地理解呢，它就是一个子系统目录加上一组资源限制文件的组合.

Linux Cgroups不足:/proc 文件系统.
### Mount
Mount Namespace 修改的，是容器进程对文件系统“挂载点”的认知.

它对容器进程视图的改变，一定是伴随着挂载操作（mount）才能生效.

Mount Namespace 正是基于对 chroot 的不断改良才被发明出来的，它也是 Linux 操作系统里的第一个 Namespace.

而这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。它还有一个更为专业的名字，叫作：**rootfs**（根文件系统）

容器诞生

1. 启用 Linux Namespace 配置；
2. 设置指定的 Cgroups 参数；
3. 切换进程的根目录（Change Root）// 优先使用 pivot_root 系统调用.

rootfs 只是一个操作系统所包含的文件、配置和目录，并**不包括操作系统内核**。
在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像.

正是由于 rootfs 的存在，容器才有了一个被反复宣传至今的重要特性：**一致性**


由于 rootfs 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，
应用以及它运行所需要的所有依赖，都被封装在了一起.

**对一个应用来说，操作系统本身才是它运行所需要的最完整的“依赖库”**
{{< hint info >}}
难道我每开发一个应用，或者升级一下现有的应用，都要重复制作一次 rootfs 吗？
{{< /hint  >}}
 Docker 在镜像的设计中，引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。

### 联合文件系统
> 联合文件系统（Union File System）,也叫 UnionFS. 最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下.

#### AuFS
AuFS 的全称是 Another UnionFS，后改名为 Alternative UnionFS，再后来干脆改名叫作 Advance UnionFS，
从这些名字中你应该能看出这样两个事实：
- 它是对 Linux 原生 UnionFS 的重写和改进；
- 它的作者怨气好像很大。我猜是 Linus Torvalds（Linux 之父）一直不让 AuFS 进入 Linux 内核主干的缘故，所以我们只能在 Ubuntu 和 Debian 这些发行版上使用它。
> OverlayFS better than AuFS,AuFS 已经成为历史.

分层
- 只读层(ro+wh,readonly+whiteout)
- Init 层 // 存放 /etc/hosts、/etc/resolv.conf 等信息
- 可读写层 // 一旦在容器里做了写操作，你修改产生的内容就会以**增量**的方式出现在这个层中.

whiteout
>  如果我现在要做的，是删除只读层里的一个文件呢？
> 
> 为了实现这样的删除操作，AuFS 会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来

### 应用容器化
#### 制作容器镜像
Dockerfile 的设计思想，是使用一些标准的原语（即大写高亮的词语），描述我们所要构建的 Docker 镜像。
并且这些原语，都是按**顺序**处理的.

CMD 都是 Docker 容器进程启动所必需的参数，完整执行格式是：“ENTRYPOINT CMD”.
默认情况下，Docker 会为你提供一个隐含的 **ENTRYPOINT**，即：`/bin/sh -c`.

**Dockerfile 中的每个原语执行后，都会生成一个对应的镜像层**

docker exec 的实现原理
>一个进程，可以选择加入到某个进程已有的 Namespace 当中，从而达到“进入”这个进程所在容器的目的

#### Network

Docker 还专门提供了一个参数，可以让你启动一个容器并“加入”到另一个容器的 Network Namespace 里，这个参数就是 -net

–net=host，就意味着这个容器不会为进程启用 Network Namespace。这就意味着，
这个容器拆除了 Network Namespace 的“隔离墙”，所以，它会和宿主机上的其他普通进程一样，直接**共享宿主机的网络栈**。
这就为容器直接操作和使用宿主机网络提供了一个渠道。

### Volume

Volume 机制解决问题
- 容器里进程新建的文件，怎么才能让宿主机获取到？
- 宿主机上的文件和目录，怎么才能让容器里的进程访问到？
> 允许你将宿主机上指定的目录或者文件，挂载到容器里面进行读取和修改操作

执行挂载操作时，“容器进程”已经创建了，也就意味着此时 Mount Namespace 已经开启了。所以，这个挂载事件只在这个容器里可见。你在宿主机上，是看不见容器内部的这个挂载点的。这就**保证了容器的隔离性不会被 Volume 打破**
> 注意：这里提到的 " 容器进程 "，是 Docker 创建的一个容器初始化进程 (dockerinit)，而不是应用进程 (ENTRYPOINT + CMD)。dockerinit 会负责完成根目录的准备、挂载设备和目录、配置 hostname 等一系列需要在容器内进行的初始化操作。最后，它通过 execv() 系统调用，让应用进程取代自己，成为容器里的 PID=1 的进程。

Linux 的绑定挂载（bind mount）机制
> 它的主要作用就是，允许你将一个目录或者文件，而不是整个设备，挂载到一个指定的目录上。并且，这时你在该挂载点上进行的任何操作，只是发生在被挂载的目录或者文件上，而原挂载点的内容则会被隐藏起来且不受影响

在一个正确的时机，进行一次绑定挂载，Docker 就可以成功地将一个宿主机上的目录或文件，不动声色地挂载到容器中.

`docker run -d -v /test helloworld`
>容器 Volume 里的信息，并不会被 docker commit 提交掉；但这个挂载点目录 /test 本身，则会出现在新的镜像当中.

## Kubernetes的本质

一个正在运行的 Linux 容器
- 一组联合挂载在 /var/lib/docker/aufs/mnt 上的 rootfs，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图；
- 一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。

容器就从一个开发者手里的小工具，一跃成为了云计算领域的绝对主角；而能够定义容器组织和管理规范的“**容器编排**”技术，则当仁不让地坐上了容器技术领域的“头把交椅”.

k8s架构图
![](../k1.png)

节点
- Master 控制节点
  - API 服务 kube-apiserver
  - 负责调度 kube-scheduler
  - 负责容器编排 kube-controller-manager
  - 整个集群的持久化数据，则由 kube-apiserver 处理后保存在 Etcd 中
- Node 计算节点
  - kubelet **负责同容器运行时（比如 Docker 项目）打交道**
  
### kubelet
kubelet交互所依赖的，是一个称作 CRI（Container Runtime Interface）的远程调用接口，
这个接口定义了容器运行时的各项核心操作，比如：启动一个容器需要的所有参数.

- 通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互
  - 接口为CNI（Container Networking Interface）
- 调用网络插件和存储插件为容器配置网络和持久化存储
  - 接口为CSI（Container Storage Interface）

观点:
> 运行在大规模集群中的各种任务之间，实际上存在着各种各样的关系。这些关系的处理，才是作业编排和管理系统最困难的地方。

Kubernetes 项目最主要的**设计思想**是，**从更宏观的角度**，以统一的方式来定义任务之间的各种关系，并且为将来支持更多种类的关系留有余地

Service 服务的主要作用，就是作为 Pod 的代理入口（Portal），从而代替 Pod 对外暴露一个固定的网络地址

![](../k2.png)

**除了应用与应用之间的关系外，应用运行的形态是影响“如何容器化这个应用”的第二个重要因素。**

在 Kubernetes 项目中，我们所推崇的使用方法是：
- 首先，通过一个“编排对象”，比如 Pod、Job、CronJob 等，来描述你试图管理的应用；
- 然后，再为它定义一些“服务对象”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能。

这种使用方法，就是所谓的“**声明式 API**”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）。

### kubeadm
> 让用户能够通过这样两条指令完成一个 Kubernetes 集群的部署
```shell
	
# 创建一个 Master 节点
$ kubeadm init
 
# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master 节点的 IP 和端口 >
```

> 把 kubelet 直接运行在宿主机上，然后使用容器部署其他的 Kubernetes 组件。