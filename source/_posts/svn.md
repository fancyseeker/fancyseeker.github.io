title: "CentOS/Fedora下SVN+Apache服务器配置"
date: 2013-11-17 14:03:34
tags: [CentOS, Fedora, Linux, SVN, maintenance, apache, server]
---

SVN一个优秀的版本管理工具, 并且适用于小型的开发团队. 本文介绍了如何配置SVN服务器以及如何通过Apache使得能通过浏览器访问SVN服务.
<!-- more -->

# SVN简介
SVN是一个优秀的版本管理工具，并且适用于小型的团队开发。SVN可以独立服务器运行或者借助Apache运行，所谓独立运行是指在服务器上配置好SVN服务器后，网内的计算机可以利用诸如TortoiseSVN这样的SVN客户端软件通过SVN协议(svn://***.***.***.***)对服务器进行签出(checkout)提交(commit)等操作。若SVN借助Apache运行，则可通过浏览器的http协议直接访问服务器对应地址下的代码。

本文先讲述如何单独配置SVN服务器，之后再讲述如何添加Apache的http访问支持。

---

# 安装环境
Fedora 19 / CentOS 6.4
(Windows也可当作SVN服务器，貌似配置过程很简单，不过从稳定性上考虑，还是选用CentOS作为服务器更为理想)

---

# SVN服务器配置

## 安装subversion包
```
# yum install subversion
```
## 初始化版本仓库(Repositories)
安装完subversion后，需要初始化一个版本仓库，用于管理代码。 新建目录
```
# mkdir -p /home/svn/project
```
在新建的目录上创建仓库
```
# svnadmin create /home/svn/project
```
这里，`svnadmin create`之后跟的为版本仓库存放代码的目录地址，在本文中为 `/home/svn/project`，当然，可以根据需求改成其它目录，但请在以下的操作中根据具体目录进行对应的修改。 在执行完`svnadmin create`操作后，在创建的仓库下，会自动生成几个目录
```
ls /home/svn/project/
conf  db  format  hooks  locks  README.txt
```
此时，如果你的电脑上有待进行管理的代码目录，可以将其导入(import)到SVN的仓库中。作为示例，这里要导入的代码目录为
```
# svn import /home/chen/Lab file:///home/svn/project -m "初始化导入代码目录"
```
在`svn import`命令中，末尾的`-m(--message)`选项表示为执行操作的日志消息，如果不加`-m` “消息内容”的话，此操作将会报错`svn: E205007`。
如果导入成功，你将会看到诸如如下过程(此处截取了导入过程中的末尾部分)
```
Adding  (bin)  /home/chen/Lab/IntelPerformanceCounterMonitorV2.4/PCM-Memory_Win/pcm-memory-win.vcproj
Adding         /home/chen/Lab/IntelPerformanceCoun(((ujterMonitorV2.4/freegetopt/README
Adding         /home/chen/Lab/IntelPerformanceCounterMonitorV2.4/freegetopt/getopt.c
Adding         /home/chen/Lab/IntelPerformanceCounterMonitorV2.4/freegetopt/ChangeLog
Adding         /home/chen/Lab/IntelPerformanceCounterMonitorV2.4/.gitignore
Adding         /home/chen/Lab/IntelPerformanceCounterMonitorV2.4/FREEBSD_HOWTO.txt

Committed revision 1.
```
## 用户管理及权限设置
SVN服务器的配置文件主要有3个，分别为
```
/your_svn_repos/conf/passwd --用户名及密码管理
/your_svn_repos/conf/authz  --权限配置
/your_svn_repos/conf/svnserve.conf  --SVN全局配置文件
```
### 添加用户
添加用户只需要打开`/home/svn/project/conf/passwd`文件，添加一行形如 `user = password` 条目即可。这里，作为示例，添加一个admin账户，密码为123456，以及其它2个账户则`passwd`文件如下
```
### This file is an example password file for svnserve.
### Its format is similar to that of svnserve.conf. As shown in the
### example below it contains one section labelled [users].
### The name and password for each user follow, one account per line.

[users]
# harry = harryssecret
# sally = sallyssecret
admin = 123456
role_a = 123456
role_b = 123456
```
### 用户访问策略配置
`/home/svn/project/conf/authz`文件用于管理用户以及用户组的访问策略。authz文件中包含若干个节，包括`[groups]`以及类似`[repository:/baz/fuz]`这样的节，注意`[groups]`节，这里用于定义用户组，在`[groups]`节中定义完不同组的用户后可以很方便的利用组进行权限管理。
```
[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe
g_admin = admin
g_common = role_a,role_b

[/]
@g_admin = rw
* = 

[project:/]
@g_admin = rw
@g_common = rw
* =
```
在以上的authz文件中，定义了组`g_admin`和`g_common`，`g_admin`组中包含有admin用户，`g_common`组中包含了用户role_a和role_b。
接下来进行具体目录的权限控制，可用类似 `user = rw` 类似的方式为用户分配该目录的对应权限，也可用 `@groupname = rw` 类似的方式为用户组分配该目录的对应权限，r代表可读，w代表可写，rw代表可读可写。同时，一定要注意在末尾添加其他人的权限设置，利用 `*` 通配符代表除了之前提到的其他人。`* =` 表示其他人不具有任何权限。
注意到文件中的两个节`[/]`和`[project:/]`，`[/]`表示SVN根目录的权限配置，而`[project:/]`表示库project的根目录权限配置，二者存在细微的差别，但是若没搞清楚，则会导致访问时验证失败或连接失败等问题。

### SVN全局配置文件
`/home/svn/project/conf/subserve.conf`为SVN的全局配置文件，这里取消对应行的注释并指定适当的值，注意不要行前不要留空格，具体如下:
```
[general]
anon-access = none   --匿名用户默认情况下不具有任何权限
auth-access = write  --授权用户具默认情况下有写权限
password-db = /home/svn/project/conf/passwd  --指定passwd文件所在位置
authz-db = /home/svn/project/conf/authz      --指定authz文件所在位置
```

## 启动SVN服务器
```
# svnserve -d -r /home/svn
```
`-d` 表示以deamon方式运行，即后台运行。 `-r`用于指定SVN服务根目录，这里我们指定的根目录为 `/home/svn`。
联系到authz文件中，`[/]`节中的权限则对应 `/home/svn` 下权限，`[project:/]`节中的权限即对应 `/home/svn/project` 下的权限。但是如果指定SVN服务根目录为 `/home/svn/project`，那么authz文件中`[/]`代表目录 `/home/svn/project`，而此时`[project:/]`则没有对应目录，因为`/home/svn/project`下并没有名为project的仓库，所以就会出错。
如果修改了SVN的配置文件，那么需要重启SVN服务器才能使修改生效。
```
# ps -aux|grep svnserve
# kill -9 pid_of_svnserve
# svnserve -d -r /home/svn
```

## 在防火墙中开放SVN端口

如果不在防火墙中开放SVN默认端口3690，则会出现，服务器本机可以访问SVN服务，而其它网内机器无法连接SVN服务器的错误。
首先关闭selinux，修改 `/etc/selinux/config` 文件
```
SELINUX=disabled
```
SVN服务的默认端口为3690，设置防火墙开放3690端口
```
# iptables -I INPUT -p tcp --dport 3690 -j ACCEPT
# iptables -I OUTPUT -p tcp --sport 3690 -j ACCEPT
# service iptables save
# service iptables restart
```
## 测试SVN服务器
随意找一个目录，尝试从SVN服务器签出(check out)仓库
```
# svn co svn://58.154.190.***/project
Authentication realm: <svn://58.154.190.***:3690> ecf95e5d-5de1-44de-91b3-f44b1bd795d1
Password for 'root': 
Authentication realm: <svn://58.154.190.***:3690> ecf95e5d-5de1-44de-91b3-f44b1bd795d1
Username: admin
Password for 'admin': 

-----------------------------------------------------------------------
ATTENTION!  Your password for authentication realm:

   <svn://58.154.190.***:3690> ecf95e5d-5de1-44de-91b3-f44b1bd795d1

can only be stored to disk unencrypted!  You are advised to configure
your system so that Subversion can store passwords encrypted, if
possible.  See the documentation for details.

You can avoid future appearances of this warning by setting the value
of the 'store-plaintext-passwords' option to either 'yes' or 'no' in
'/root/.subversion/servers'.
-----------------------------------------------------------------------
Store password unencrypted (yes/no)?yes 
...
...
A    project/IntelPerformanceCounterMonitorV2.4/pcm3d/widget.h
A    project/IntelPerformanceCounterMonitorV2.4/pcm-memory.cpp
A    project/IntelPerformanceCounterMonitorV2.4/pcm-sensor.cpp
A    project/IntelPerformanceCounterMonitorV2.4/.gitattributes
Checked out revision 1.
```
在第一次签出的时候会提示是否保存明文密码(因为SVN的密码是明文保存的，所以SVN的账户安全依赖与Linux系统账户的安全)选择"yes"的话下次就不需要在输入密码了，反之，选择"no"的话下次操作还需进行身份验证。如果想换个身份操作SVN或者密码输错太多被拒绝，可以删除用户目录下的 `～/.subversion` 文件夹以清空身份信息。
如果能顺利 check out revision 则表明独立SVN服务器配置已成功。
这时不论是在服务器本机上还是其它客户端机器，均能使用SVN服务。对于客户端操作系统为Windows的计算机，可以安装TortoiseSVN进行相应的客户端操作。

---

# 为SVN服务器添加HTTP支持

经过上面的配置过程后，SVN服务器已经可以正常使用，但是，只能通过SVN协议，无法通过http协议访问。为SVN服务器添加HTTP支持可以使得用户能用浏览器通过HTTP协议直接查看仓库内容，同时也可以在代码签出提交时使用HTTP协议。

## 安装必要包
```
yum install httpd mod_dav_svn mod_perl  perl* ntsysv vim-enhanced
```
## 转换SVN明文密码为HTTP要求的加密格式
SVN的密码是明文保存的，而http服务器不支持明文密码，所以需要将SVN服务的明文密码转换为http要求的加密格式，可以通过以下的perl脚本完成：
移动到 `/home/svn/project/conf` 目录下，新建并编辑perl脚本 `PtoWP.pl`  (此脚本的作者为ha97)
```
cd /home/svn/project/conf/
vim PtoWP.pl
```
```
#!/usr/bin/perl
use warnings;
use strict;

#open the svn passwd file
open (FILE, "passwd") or die ("Cannot open the passwd file!!!n");

#clear the apache passwd file
open (OUT_FILE, ">webpasswd") or die ("Cannot open the webpasswd file!!!n");
close (OUT_FILE);

#begin
foreach () {
if($_ =~ m/^[^#].*=/) {
$_ =~ s/=//;
`htpasswd -b webpasswd $_`;
}
}
```
为脚本添加可执行属性
```
# chmod +x PtoWP.pl
```
在目录 `/home/svn/project/conf/passwd` 所在目录中运行`PtoWP.pl`脚本，对passwd文件进行转换
```
# ./PtoWP.pl
```
转换过程如下
```
Adding password for user admin
Adding password for user role_a
Adding password for user role_b
```
转换完成后目录下会出现一个 `webpasswd` 文件，即为http支持的密码文件。

## 修改http.conf，使得http支持SVN

```
vim /etc/httpd/conf/httpd.conf
```
在`httpd.conf`文件的最后添加如下信息
```
# 服务模块
DAV svn
# SVN仓库根目录
SVNPath /home/svn/project/
# 授权方式,这里配置为基本授权方式
AuthType Basic
# 授权名
AuthName "svn for project"
# 用户名及用户密码文件
AuthUserFile /home/svn/project/conf/webpasswd
# 访问权限配置文件
AuthzSVNAccessFile /home/svn/project/conf/authz
Satisfy all
# 访问方式，这里配置为必须输入账户密码
Require valid-user
```
看了不少文章，有部分文章说在这里还需添加两个模块的语句，另外还需要 `yum install subversion-deps-*` 包来使得svn得到http的支持。
```
LoadModule dav_svn_module modules/mod_dav_svn.so
LoadModule authz_svn_module modules/mod_authz_svn.so
```
但是，根据官方的说明(Please note that the dependencies distribution subversion-deps-* is no longer available in 1.7 and later.)也就是说在1.7以及1.7之后的版本中不需要安装依赖的`subversion-deps-*`包，而本人实验证明，也不需要在`httpd.conf`文件中添加上述两行加载模块的语句。

修改SVN主目录的所有者和所属组为Apache
```
# chown -R apache.apache /home/svn/project/
```

## 在防火墙中开放httpd服务的80端口
```
# iptables -I INPUT -p tcp --dport 80 -j ACCEPT
# iptables -I OUTPUT -p tcp --sport 80 -j ACCEPT
# service iptables save
# service iptables restart
```
如果不在防火墙中开放80端口，则会出前客户机无法通过http协议连接SVN服务器的情况。
**注:** 关于服务重启命令，在fedora 19中采用的是systemctl restart service_name.service形式。

## 重启httpd服务
```
# service httpd restart
```

## 通过浏览器测试
在浏览器地址栏输入 `http://your_svn_server_ip/project`，在弹出的身份确认框中输入用户名密码，能通过浏览器查看SVN仓库。
同时可以通过http协议执行svn操作
```
$ svn co http://your_svn_server_ip/project
Authentication realm: <http://58.154.190.***:80> svn for project
Password for 'chen': 
Authentication realm: <http://58.154.190.***:80> svn for project
Username: admin
Password for 'admin':
```

# 参考资料
[（总结）CentOS Linux搭建SVN Server配置详解](http://www.ha97.com/4467.html)
