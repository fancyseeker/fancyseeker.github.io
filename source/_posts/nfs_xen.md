title: "利用NFS在Xen中设立共享文件夹"
date: 2014-06-16 13:15:52
tags: [NFS, Xen, Virtualization]
---

# Background

趁着虚拟机在安装guest os的时候抽空写一篇小短文吧。这篇文章主要介绍利用NFS在Xen的host os和guest os中共享文件夹。众所周知的，VMware是很容易在宿主机(Host)的windows和客户机(Guest)中共享文件夹的，点几下按钮的事情。但是同样的事情到了Linux下的Xen上，就变得有点麻烦，因为Xen没有提供Host和Guest之间的共享手段，所以用户就需要利用其它方法来实现虚拟机和宿主机共享文件夹这事。

首先，让我们来介绍一下故事的背景。故事的起因是本人想要阅读下内核代码，觉得单纯的读很枯燥，想顺带改改内核代码试试看效果，但改内核这事也不好直接在物理机上进行，所以自然而然的想到了放到虚拟机中取执行修改的内核。但是代码的修改又需要在Host机上进行(因为嫌麻烦Guest没有配置相对应的工具嘛)，这样就不免要涉及到将Host修改后的代码放入到Guest中编译安装的过程，很自然的，需要实现Host与Guest之间的文件共享机制。

**实验环境：**
本人的物理机上安装的是Linux Mint 16，然后在Mint上安装了Xen虚拟化平台。利用虚拟化软件Xen，安装了一个ubuntu的虚拟机。

**本文结构：**
如果不算开头背景啰嗦的话，本文大致分为2个部分，
1. 介绍NFS(Network File System)的介绍及简易配置 
2. 利用NFS实现虚拟化平台中Host与Guest，或是Guest与Guest之间的文件共享。

---

# NFS的介绍及配置

## NFS介绍

NFS(Network File System)是FreeBSD支持的文件系统中的一种主要用于在Linux机器之间提供文件共享服务，其基本的原理类似与C/S模式，即服务器/客户端模式，一台提供共享文件夹的计算机充当服务器，其它若干台计算机作为客户机来访问这个共享文件夹。因此，在配置NFS的时候，思路的就很简单。首先，针对共享文件夹所在的机器，进行相关的NFS配置，以便将共享文件夹发布(export)给指定网络内的机器以及设定对应的权限控制，这样指定网络内的计算机就都能看到这个共享文件夹，然后将网络上的共享文件夹挂载(mount)到自己的文件系统下便可以进行访问了。通过这种export/mount模式，NFS便实现了文件共享机制。

事实上，在文件共享方面，还有一个更知名(或者说使用者更多)的服务，那便是Samba服务，而NFS则显得相对小众点(或者说使用者多为Linux管理员一类的)，之所以是这样，就是因为NFS只用于Linux与Linux系统之间的文件共享，而Samba则可用于Linux与Windows之间，Linux与Linux之间的文件共享。从Windows和Linux的用户基数的对比上就可以看出其中端倪。虽然Samba看起来比NFS功能强大，但是NFS也有自身优势，相比于Samba，NFS显得更加轻量级也更稳定点，当然，配置起来也更简单点。

## NFS配置

### 安装NFS

首先，查看下机器中是否安装了NFS(一般的Linux发行版都是默认安装NFS的)。

ubuntu系统查看是否安装NFS：`$ dpkg -l | grep nfs`

fedora/centos查看是否安装NFS：`$ rpm -qa | grep nfs`

如果系统内没有NFS，那么便需要安装，

ubuntu系统安装NFS
```
$ sudo apt-get install nfs-kernel-server nfs-common
```

fedora/centos系统安装NFS
```
# yum install nfs
```

### 配置文件

在安装完NFS后，在系统的 `/etc` 目录下会多一个 `exports` 文件，该文件用于指定要共享的目录以及相关的权限设置。

接下来我们来修改配置文件 `/etc/exports`
对于之前没有配置过NFS的机器，`exports` 的文件的样子是这样的：
```
# /etc/exports: the access control list for filesystems which may be exported
#       to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
```
NFS服务的配置基本上就只需要在exports文件中进行。注释中给出了配置文件信息的格式，我们根据注释添加相对应的行就行。

在exports文件的末尾添加一行： `/your_path/nfs_dir *(rw,sync,no_root_squash)` 

其中 **`/your_path/nfs_dir`** 表示用户自定义的路径下的 **`nfs_dir`** 目录，用于与客户机共享；例如要想将 `/home/foo/share` 共享，那么可以在 exports文件中添加一行 `/home/foo/share *(rw,sync,no_root_squash)` 路径后面跟的是对该共享目录的权限控制信息。格式为 `network(permission parameters)`。

上面例子中`*`代表任意网段的计算机均可访问到共享文件夹。当然，你也可以根据需要来指定特定的网段，例如特定IP地址的主机 `192.168.1.36`，或者特定网段内的主机 `58.154.190.0/24(58.154.190.0/255.255.255.0)`，在或者指定域名的主机 `*.neu.edu.cn` (域名能在本地 `/etc/hosts` 文件中或通过DNS服务器找到)。如果你不明确访问共享文件夹的计算机网络的话，那还是用`*`吧。

**`(rw,sync,root_squash)`**表示其它机器访问共享文件夹时的权限。

NFS常用的参数有：

|Parameter|Description|
|:-|:-|
|`ro`|只读访问|
|`rw`|读写访问sync所有数据在请求时写入共享|
|`async`|nfs在写入数据前可以响应请求|
|`secure`|nfs通过1024以下的安全TCP/IP端口发送|
|`insecure`|nfs通过1024以上的端口发送|
|`wdelay`|如果多个用户要写入nfs目录，则归组写入（默认）|
|`no_wdelay`|如果多个用户要写入nfs目录，则立即写入，当使用async时，无需此设置。|
|`hide`|在nfs共享目录中不共享其子目录|
|`no_hide`|共享nfs目录的子目录|
|`subtree_check`|如果共享/usr/bin之类的子目录时，强制nfs检查父目录的权限（默认）|
|`no_subtree_check`|和上面相对，不检查父目录权限|
|`all_squash`|共享文件的UID和GID映射匿名用户anonymous，适合公用目录。|
|`no_all_squash`|保留共享文件的UID和GID（默认）|
|`root_squash`|root用户的所有请求映射成如anonymous用户一样的权限（默认）|
|`no_root_squas`|root用户具有根目录的完全管理访问权限(不安全，不推荐)|
|`anonuid=xxx`|指定nfs服务器/etc/passwd文件中匿名用户的UID|
|`anongid=xxx`|指定nfs服务器/etc/passwd文件中匿名用户的GID|

修改完成之后输入：`# exportfs –rv` 来使配置文件生效。

重启NFS服务
```
$sudo /etc/init.d/nfs-kernel-server restart
```

网上一些帖子里说由于NFS是一个RPC程序，使用它前，需要通过portmap映射好端口故需要重启portmap服务

```
$ sudo /etc/init.d/portmap restart
```


但是本人的Mint 16上并未发现有portmap服务，故忽略，最后也能成功执行NFS。
最后，查看目录是否已经共享，

```
$ showmount -e localhost  #查询本机nfs共享目录情况
$ showmount -a localhost  #查询本机共享目录连接情况
```

通过 `$ showmount -e`  显示出共享出来的目录，如果结果中有你之前在exports文件中指定的目录，那么NFS服务器端的共享文件夹就成功发布了，即指定网段内的主机可以发现这个共享文件夹了。

## NFS客户机挂载共享目录

我们已经完成了NFS服务器端的共享目录配置，并确保了共享目录可以被特定网段的计算机发现了，那么接下来，我们就需要在其它机器上“接收”由NFS服务器那台机器“发布”出来的共享文件夹。

这里需要将网络共享目录挂载(mount)到本地机器的文件系统中。
```
$ mount -t nfs hostip:/host_shared_path/shared_dir /opt/shared_dir
```

例如，已知NFS服务器的IP为 `192.168.1.36`，并且其 `/home/foo/share` 目录是共享的，要将其mount至本机的 `/home/zoo/nfs` 目录下，那么便可执行：
```
$ mount -t nfs 192.168.1.36:/home/foo/share /home/zoo/nfs
```
这样就可以通过本机中的 `/home/zoo/nfs` 目录操作共享文件了。

如果你愿意的话，还可以启用开机自动挂载，将挂载信息写入 `/etc/fstab` 文件，在末尾添加一行：
```
host_ip:/host_share_path /local_mount_path nfs defaults
```
然后执行 `# mount -a` 即可。

至此，NFS的配置和使用过程讲解结束，目前未涉及到半点虚拟机的内容，接下来，我们来讲讲如何将NFS用与虚拟机中Host与Guest共享文件。

---

# 采用NFS实现Xen中Host与Guest的文件共享

Xen并未提供在宿主机Host与客户机Guest之间的文件共享方式。因此，我们不得不自己动手来完成这个事情。

如果说仅仅实现了Xen中的Host与Guest的文件共享，我想也未免太针对了，事实上，利用NFS我们可以完成任意台Linux主机之间的文件共享，哪怕这些Linux主机中包含有虚拟的计算机。不管是在Xen中，还是在VMware中又或者是在Virtual Box中，只要Host是Linux系统，Guest也是Linux系统，Host和Guest在网络上能互相查找到对方，那么便可利用NFS完成文件共享。那么，显然的，如果Host机或者Guest机中有Windows系统机器，NFS就不再适用了，这时候你就需要Samba。

我们注意到，利用NFS来进行文件共享，有2个主要的要求：

1. 参与共享的主机都必须是Linux系统

2. 参与共享的主机必须能在网络上互相找到对方

对于第1点，要不是显而易见的，要不是无能为力的(不过，如果非要在Linux和Windows之间利用NFS实现文件共享，也是可以的)。我们这里着重来说说第2点，在虚拟机的环境中，如何判断Host和Guest能互相在网络上找到对方。

说到这里，我们就不得不稍微讲一讲几种虚拟机常用的网络模型：Bridge，Host-Only，NAT。

> 1.Bridged(桥接模式)

> 在Bridged模式下，虚拟出来的操作系统就像是局域网中的一台独立的主机，它可以访问网内任何一台机器。但是你需要手工为虚拟系统配置IP地址、子网掩码，而且还要和宿主机器处于同一网段，这样虚拟系统才能和宿主机器进行通信。同时，由于这个虚拟系统是局域网中的一个独立的主机系统，那么就可以手工配置它的TCP/IP配置信息，以实现通过局域网的网关或路由器访问互联网。 使用bridged模式的虚拟系统和宿主机器的关系，就像连接在同一个Hub上的两台电脑。

> 2.NAT(Network Address Translation, 网络地址转换模式)

> 使用NAT模式，就是让虚拟系统借助NAT(网络地址转换)功能，通过宿主机器所在的网络来访问公网。也就是说，使用NAT模式可以实现在虚拟系统里访问互联网。NAT模式下的虚拟系统的TCP/IP配置信息是由虚拟机提供的虚拟网络DHCP服务器提供的，无法进行手工修改，因此虚拟系统也就无法和本局域网中的其他真实主机进行通讯。采用NAT模式最大的优势是虚拟系统接入互联网非常简单，你不需要进行任何其他的配置，只需要宿主机器能访问互联网即可。 这种方式也可以实现Host OS与Guest OS的双向访问。但网络内其他机器不能访问Guest OS，Guest OS可通过Host OS用NAT协议访问网络内其他机器。NAT方式的IP地址配置方法是由虚拟DHCP服务器中分配一个IP ，在这个IP地址中已经设置好路由，就是指向192.168.138.1的。 如果你想利用虚拟机安装一个新的虚拟系统，在虚拟系统中不用进行任何手工配置就能直接访问互联网，建议你采用

> NAT模式。

> Host-Only(主机模式)

> 在某些特殊的网络调试环境中，要求将真实环境和虚拟环境隔离开，这时你就可采用Host-Only模式。在Host-Only模式中，所有的虚拟系统是可以相互通信的，但虚拟系统和真实的网络是被隔离开的。 提示:在Host-Only模式下，虚拟系统和宿主机器系统是可以相互通信的，相当于这两台机器通过双绞线互连。 在Host-Only模式下，虚拟系统的TCP/IP配置信息(如IP地址、网关地址、DNS服务器等)，都是由虚拟网络的DHCP服务器来动态分配的。 如果你想利用VMWare创建一个与网内其他机器相隔离的虚拟系统，进行某些特殊的网络调试工作，可以选择Host-Only模式。

让我们以Xen为例介绍一下，本人在Xen中新建虚拟机时，网络连接模式采用的是 net bridge形式，在Host上，通过 `$ ifconfig` 命令，可以发现Host中eth0网卡的IP地址为 `58.154.190.×`，并且多了一块虚拟的 `virbr0` 网卡，其IP地址为 `192.168.122.1`。在Guest中，通过 `$ ifconfig` 命令，发现其IP地址为 `192.168.122.168`。

因此，Host采用virbr0的IP地址就能和Guest进行通信，在Host中采用 `$ ping 192.168.122.168` 查看是否能连接的上Guest，同样的，在Guest中采用 `$ ping 192.168.122.1` 查看是否能连接的上Host。如果二者能正常连接，那么，便可采用文章上半部分讲述的NFS方式来实现Host与Guest直接的文件共享了(只需要在宿主机配置好了NFS服务后，在客户机mount 共享文件夹时填入正确的宿主机的IP地址即可)。
