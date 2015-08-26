title: "CentOS和Fedora的源配置及repo文件解析"
date: 2012-12-14 15:30:06
tags: [CentOS, Fedora, Linux, repo, maintenance, yum]
---

本文对CentOS和Fedora的源配置文件进行了适当的解析, 同时介绍了配置源的相关方法.
<!-- more -->

# Background

最近由于项目上的需要, 在VMware上装上了CentOS. 由于学校的校园网无法访问国外的源, 所以CentOS默认的源无法使用, 于是装完系统的第一件事情自然是配源. 对于国内用户来说163的源和sohu的源速度都不错更新的也快, 对于校园网用户来说, 校园网内比较好的源有上海交大(sjtu), 中科大(ustc)... 呵呵, 不知道东北大学(neu)的算不算, 不过我用起来速度也不错.

之前我自己在配Fedora和CentOS的源的时候都是直接google, 然后从网上直接select--copy--paste到本地的repo文件的. 经常找个合适的源文件要找半天, 最令人失望的是有时还不能正常使用. 俗话说的好, "自己动手, 丰衣足食"嘛. 所以花了点时间研究了下yum源配置文件的问题, 发现还是蛮简单的.

# Yum与repo文件简介

[Yum(Yellow dog Updater, Modified)](http://en.wikipedia.org/wiki/Yellowdog_Updater,_Modified)是一个在Fedora和RedHat以及SUSE、CentOS中的Shell前端软件包管理器。基於RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。

在通过yum更新或者下载软件包的时候, yum需要从"源"获得软件包的信息, 所谓的"源", 在我理解看来就是一个规范化的软件包下载点, 之所以说它是规范化的是因为"源"中有一个列表和一些信息,用于记录"源"中软件的发布情况, 这样yum就可以通过列表获取软件包的信息,进而判断一个是否在"源"中存在指定软件或者当前的软件包过期需要升级等等.

repo文件, 由于yum需要利用"源"来获取软件包, 那么很自然的, 在本地机器上自然需要有描述"源"信息的文件, 这就是repo文件. repo文件存放在`/etc/yum.repo.d/`目录下, 通过root权限对其进行访问或者修改.
![yum.repo.d](/img/repo_config/repofile.png)

# repo源配置文件解析

这里以网易的163源中的fedora源配置文件作为例子, 简单讲述下repo文件的结构.
实际上, 获得163的源配置文件是件很容易的事情. 登陆[网易的镜像服务器](http://mirrors.163.com/)

![163 mirror](/img/repo_config/163mirror.png)

可以看到, 网易很人性化的在每个系统条目后面都添加了对应的使用帮助, 点击即可显示:
![fedora mirror](/img/repo_config/fedorarepo.png)
按照网页指示的, 我们下载对应的[fedora-163.repo](http://mirrors.163.com/.help/fedora-163.repo)文件下来看看.

```bash
[fedora]
name=Fedora $releasever - $basearch - 164.com
failovermethod=priority
baseurl=http://mirrors.163.com/fedora/releases/$releasever/Everything/$basearch/os/
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
enabled=1
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$basearch

[fedora-debuginfo]
name=Fedora $releasever - $basearch - Debug - 163.com
failovermethod=priority
baseurl=http://mirrors.163.com/fedora/releases/$releasever/Everything/$basearch/debug/
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=fedora-debug-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$basearch

[fedora-source]
name=Fedora $releasever - Source - 163.com
failovermethod=priority
baseurl=http://mirrors.163.com/fedora/releases/$releasever/Everything/source/SRPMS/
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=fedora-source-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$basearch
```

可以很明显的看到, repo文件分为若干个不同的节, 每节的内容类似.以上述文件为例,我们对其进行简要的分析.

```bash
#注:"#"后面的内容为repo文件的注释内容.
[fedora]        #中括号内部的是软件源的名称,由yum提取识别

name=Fedora $releasever - $basearch - 164.com   
#name字段给出了软件仓库的名称,只是为了方便阅读,可随意填写

failovermethod=priority
#failovermethod 有两个值,priority是默认值,表示当baseurl中有多个地址时,
#从列出的baseurl中顺序选择镜像服务器地址,roundrobin表示在列出的服务器中随机选择

baseurl=http://mirrors.163.com/fedora/releases/$releasever/Everything/$basearch/os/     
#baseurl给出源的镜像服务器地址,yum就是通过它来访问镜像服务器的
.
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch    
#mirrorlist给出镜像服务器的列表,作用不大,可注释掉.

enabled=1       #enable有两个值,1代表当前软件源启用,0代表禁用.

metadata_expire=7d      
#此字段用于指定yum的元数据即metadata的过期时间,默认单位为秒,
#可以通过加后缀来更改单位,例如h代表小时,m代表分钟,d代表天.
#此处的值为7d即代表当yum更新了一次metadata后,在7天之内,它不会在主动更新metadata.

gpgcheck=1      
#gpgcheck有两个值,1和0,当值为1的时候,代表对下载的rpm包进行gpg校验,以确保包的有效性和安全性.

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$basearch     
#定义用于校验的gpg密钥信息

#以下的两个软件源相对来说不是很常用,看他们的enable值就知道了=_=,格式和上述的软件源相同,故不再赘述,做注释也很累的有没有!
[fedora-debuginfo]
name=Fedora $releasever - $basearch - Debug - 163.com
failovermethod=priority
baseurl=http://mirrors.163.com/fedora/releases/$releasever/Everything/$basearch/debug/
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=fedora-debug-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$basearch

[fedora-source]
name=Fedora $releasever - Source - 163.com
failovermethod=priority
baseurl=http://mirrors.163.com/fedora/releases/$releasever/Everything/source/SRPMS/
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=fedora-source-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$basearch
```

到这里,repo文件各个字段的大致含义,大家应该都了解了.但是想必各位童鞋都注意到了repo文件中的一些变量,例如: `$releasever`, `$arch`, `$basearch`等等.那么接下来,我们来介绍下repo文件中出现的几个变量吧.我们可以通过

```
$man yum.conf
```

在man手册yum.conf(5)中找到关于这几个变量的解释.节选一部分如下:

> VARIABLES
> There are a number of variables you can use to ease maintenance of yum's configuration files. They are available in the values of several options including name, baseurl and commands.
>
>     $releasever This will be replaced with the value of the version of the package listed in distroverpkg. This defaults to the version of `redhat-release' package.
>
>     $arch This will be replaced with your architecture as listed by os.uname()[4] in Python.
> 
>     $basearch This will be replaced with your base architecture in yum. For example, if your $arch is i686 your $basearch will be i386.

简单的说明下man手册中关于各个变量的解释
`$releasever`:该变量将会被替换成具体的系统发行版本,比如我的系统为fedora 17,那么变量$releasever的值就为17.

`$arch`:该变量会被替换成系统对应的架构,一般情况下$arch的值为i686(32位系统)或者x86_64(64位系统),当然,如果你的电脑是古董级的,有可能出现什么i386或者i486之类的值,或者你用的是PowerPC架构的处理器,$arch的值会是ppc = =.

`$basearch`: basearch顾名思义就是基础架构啦,man手册中的解释是yum中对应的基础架构,我的理解是不管你用的是686,586还是486的CPU,只要是32位的都属于基础的i386架构,这里所谓的base,指的应该就是这个意思.

关于这几个变量,有意思的是它们无法直接在终端下利用`echo $variable`方式输出得到.我找到一个比较方便的办法就是通过执行:
```
$uname -r
```
来间接得到`$releasever`和`$arch`的值.例如我执行`uname -r`命令得到的结果就是: `3.6.8-2.fc17.x86_64`
所以针对我的电脑, `$releasever`的值为17, `$arch`的值为x86_64(我的系统是64bit的)

常用源及其目录组织结构

在掌握了repo文件后,我们现在要开始自己动手配置源了,毕竟本片文章的题目还涉及到源配置的问题嘛,不讲怎么配源肯定是对不起观众的XD.

首先介绍几个国内比较知名的源:
[网易163](http://mirrors.163.com/)
[搜狐sohu](http://mirrors.sohu.com/)

校园网下比较好的源有:
[上海交大sjtu](ftp://ftp.sjtu.edu.cn/)
[中科大ustc](http://mirrors.ustc.edu.cn/)
[东北大学neu](http://mirror.neu.edu.cn/)

其中比较推荐的是163的源和上海交大的源,因为他们相对来说更新及时,比较全面,同时速度也快.顺便说以下,**在这些源中,163,sohu和中科大的源是有包含帮助页面的,从而可以在帮助页面下直接下载到对应的repo文件,不需要做任何修改,直接放到对应位置然后运行yum makecache就可以了.**但是其他的一些源就没有相应的repo文件下载,所以需要我们自行配置repo文件.(这里不得不吐槽下,如果他们都弄个repo文件方便大家下载,我就不用这么辛苦的码字写文章了=_=b)

自行配置源的思路很简单,就是更改repo文件中对应的字段的值.其中,对我们来说最重要的字段自然是baseurl字段了.正如前文说过的,yum通过baseurl字段给出的地址找到源镜像服务器,进而从镜像服务器下载软件包.

从上文的示例repo文件中,我们可以看到baseurl字段通常是如下形式:
`baseurl=http://mirrors.163.com/fedora/releases/$releasever/Everything/$basearch/os/`
很明显的,如果用具体的值带入其中的变量,那么baseurl给出的是一个网址.
以本人电脑为例,`$releasever=17`, `$basearch=x86_64`,那么得到的网址是:
`http://mirrors.163.com/fedora/releases/17/Everything/x86_64/os/`
我们不妨打开对应的网址,你会发现,居然成功打开了,而且网页还有Packages和repodata两个文件夹,这两个文件夹下面还有好多东西,很神奇有没有~

很显然的,yum就是在根据这个网址找到需要的软件包的.所以,我们的目的就是要让yum找到镜像源服务器的对应目录.这不禁让我们联想到,如果163的源需要的软件包在以上给出的目录下,那么像上海交大那样的源很可能也有一样的目录结构,因此我们不妨在上海交大的源服务器上找找是不是也有类似 `/fedora/releases/17/Everything/x86_64/os/`这样的目录.很容易的,我们就找到了类似的目录:
`ftp://ftp.sjtu.edu.cn/fedora/linux/releases/17/Everything/x86_64/os/`
通过查看其它的一些镜像源服务器,我们都可以发现实际上它们的目录结构都是差不多的.

修改repo文件

我们已经知道镜像源服务器的目录结构都是相似,因此我们可以利用已有的repo文件,适当的进行下修改,得到一个新的repo文件,从而达到配置新的软件源的目的.
这里以163的源配置文件为基础,进行适当修改,得到上海交大的源配置文件.

以下为163源配置文件中的一节
```bash
[fedora]
name=Fedora $releasever - $basearch - 163.com
failovermethod=priority
baseurl=http://mirrors.163.com/fedora/releases/$releasever/Everything/$basearch/os/
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
enabled=1
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$basearch
```
以下为修改后上海交大源配置文件中的一节

```bash
[fedora-sjtu]
name=Fedora $releasever - $basearch - sjtu
baseurl=http://ftp.sjtu.edu.cn/fedora/linux/releases/$releasever/Fedora/$basearch/os/
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
enabled=1
metadata_expire=7d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$basearch
```

通过对比我们不难发现,实际上就只对几处进行了修改.

将中括号中更改为对应的软件源名字,例子中`fedora`->`fedora-sjtu`(不改也是可以的,仅仅是为了便于识别)
name字段更改为对应的软件源名字(同样的,那么的更改也是非必须的,仅仅是为了便于识别)
baserul字段更改为新的镜像源服务器地址.这是最关键的一处更改,为了获得正确的地址,最好之前先通过浏览器访问下对应的地址查看是否正确,同时合理的利用`$releasever`, `$basearch`等变量可以方便的应对当系统升级或者其他情况带来的版本变更所导致的baserul错误等问题.
除了以上给出的节之外,repo文件里还记录了一些不常用的软件仓库信息,它们一般都是不启用的(`enable`字段值为0),例如debuginfo等,具体的更改方法都是一样的.童鞋们可自行参照以上的例子进行更改.

**最后,只需要将repo文件放入/etc/yum.repo.d/目录下,然后利用root权限运行**

```bash
#yum makecache
```
**更新yum的元数据,新的软件源即可正常使用了.**

# 结束语

至此,整个源配置过程结束,由于本人对ubuntu和Debian系的接触比较少,不太了解apt-get的源配置方法,就不丢人现眼啦.在这里只是简单介绍了下RedHat系的fedora和CentOS的源配置问题.由于linux distribution的更新速度很快,所以文中难免存在一些不足之处,还望指出.希望本文章能让大家了解源配置的过程,更快的配置自己需要的源.

# 参考资料

- yum的具体介绍参见wiki页面:[Yellowdog Updater, Modified](http://en.wikipedia.org/wiki/Yellowdog_Updater,_Modified)
- repo文件解析可参考这篇帖子:[带你认识repo文件](http://bbs.fedora-zh.org/showthread.php?1376-%E5%B8%A6%E4%BD%A0%E8%AE%A4%E8%AF%86repo%E6%96%87%E4%BB%B6)
- 变量$releasever和$basearch的讨论可以查看这篇帖子: [Yum: How can I view variables like$releasever, $basearch & $YUM0?](http://unix.stackexchange.com/questions/19701/yum-how-can-i-view-variables-like-releasever-basearch-yum0)
- 官方的mirrorlist参考: [Fedora Public Active Mirrors](https://mirrors.fedoraproject.org/publiclist/)
