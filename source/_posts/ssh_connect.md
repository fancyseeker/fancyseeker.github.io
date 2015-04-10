title: "SSH配置-在Windows下远程登陆Linux服务器Shell"
date: 2013-12-31 17:24:47
tags: [ssh, shell, PuTTY]
---

# Background

这几天需要在实验室空闲电脑上配置一个samba服务器用于共享文件, 而那台电脑所在座位上刚好来了一个人临时要在这边待几个星期, 因此无法直接在服务器前进行配置, 于是乎想到了利用SSH来远程登陆服务器进行操作. 其实说是远程, 也就隔了不到5米, 但是通过自己的电脑操控另一台电脑的感觉真的还是很美妙的. 想象一下此刻你正处于地球上不知道哪个地方, 然后通过庞大的互联网, 连接到了与你距离十万八千里的某台主机上进行操作, 想想还有点小激动呢.

---

# 系统环境

服务器: CentOS 6.4 x86_64    OpenSSH
SSH客户机: Windows 7 64bit    PuTTY

---

# 安装启动SSH服务

在CentOS上查看是否安装了ssh相关的包.
```
[fancyseeker@localhost ~]$ rpm -qa | grep ssh
openssh-5.3p1-94.el6.x86_64
openssh-askpass-5.3p1-94.el6.x86_64
libssh2-1.4.2-1.el6.x86_64
openssh-server-5.3p1-94.el6.x86_64
openssh-clients-5.3p1-94.el6.x86_64
openssh-ldap-5.3p1-94.el6.x86_64
```
如果没有安装, 那么需要手动安装下
```
# yum install openssh*
```

2. 设置开机启动SSH服务
```
# chkconfig sshd on
```
3. 开启SSH服务
```
# /etc/init.d/sshd start
```
4. 查看SSH服务运行状态
```
# /etc/init.d/sshd status
```

---

# 配置SSH服务

SSH服务配置 SSH服务的配置文件主要有2个, 分别为 `/etc/ssh/ssh_config` 以及 `/etc/ssh/sshd_config`

## ssh_config配置文件

这里我们先对`ssh_config`配置文件进行修改, 添加或修改如下几项
```
# 使用RSA算法进行安全验证
RSAAuthentication yes
# 关闭密码验证
PasswordAuthentication no
# 强制使用的SSH2
Protocol 2
```
由于接下来我们会选用RSA算法来产生密钥对, 因此将`RSAAuthentication`设置成`yes`, `PasswordAuthentication`设置成`no`使得客户端无法通过不安全的账户密码方式登录, 增强安全性. `Protocol 2`设定强制使用SSH的第二版. 当然, 如果希望仅在某一个网段内进行SSH连接, 那么可以设置 `Host *`选项, 将`*`换成允许的网段, 例如`192.168.1.`代表IP地址为`192.168.1.x`的电脑可以进行SSH连接, 而其他IP地址的电脑则无法连接. 关于`ssh_config`文件的具体选项的解释如下所示<sup>[[1](http://blog.lizhigang.net/archives/249)</sup>
```
Host *
# 选项“Host”只对能够匹配后面字串的计算机有效。“*”表示所有的计算机。
ForwardAgent no
# “ForwardAgent”设置连接是否经过验证代理（如果存在）转发给远程计算机。
ForwardX11 no
# “ForwardX11”设置X11连接是否被自动重定向到安全的通道和显示集（DISPLAY set）。
RhostsAuthentication no
# “RhostsAuthentication”设置是否使用基于rhosts的安全验证。
RhostsRSAAuthentication no
# “RhostsRSAAuthentication”设置是否使用用RSA算法的基于rhosts的安全验证。
RSAAuthentication yes
# “RSAAuthentication”设置是否使用RSA算法进行安全验证。
PasswordAuthentication no
# “PasswordAuthentication”设置是否使用口令验证。
FallBackToRsh no
# “FallBackToRsh”设置如果用ssh连接出现错误是否自动使用rsh。
UseRsh no
# “UseRsh”设置是否在这台计算机上使用“rlogin/rsh”。
BatchMode no
# “BatchMode”如果设为“yes”，passphrase/password（交互式输入口令）的提示将被禁止。当不能交互式输入口令的时候，这个选项对脚本文件和批处理任务十分有用。
CheckHostIP yes
# “CheckHostIP”设置ssh是否查看连接到服务器的主机的IP地址以防止DNS欺骗。建议设置为“yes”。
StrictHostKeyChecking no
# “StrictHostKeyChecking”如果设置成“yes”，ssh就不会自动把计算机的密匙加入“$HOME/.ssh/known_hosts”文件，并且一旦计算机的密匙发生了变化，就拒绝连接。
IdentityFile ~/.ssh/identity
# “IdentityFile”设置从哪个文件读取用户的RSA安全验证标识。
Port 22
# “Port”设置连接到远程主机的端口。
Cipher blowfish
# “Cipher”设置加密用的密码。
EscapeChar ~
# “EscapeChar”设置escape字符。
```

## sshd_config配置文件

在sshd_config文件中添加或修改如下选项
```
PermitRootLogin no
PermitEmptyPasswords no
```
出于安全性的考虑, 我们不允许客户端直接用root账户进行登录, 故将`PermitRootLogin`设置成`no`(当然, 可以在登陆了其他用户后切换至root用户), 同时不允许空密码. `sshd_config`文件的其它具体选项的解释如下所示<sup>[[1](http://blog.lizhigang.net/archives/249)</sup>
```
Port 22
# “Port”设置sshd监听的端口号。
ListenAddress 192.168.1.1
# “ListenAddress”设置sshd服务器绑定的IP地址。
HostKey /etc/ssh/ssh_host_key
# “HostKey”设置包含计算机私人密匙的文件。
ServerKeyBits 1024
# “ServerKeyBits”定义服务器密匙的位数。
LoginGraceTime 600
# “LoginGraceTime”设置如果用户不能成功登录，在切断连接之前服务器需要等待的时间（以秒为单位）。
KeyRegenerationInterval 3600
# “KeyRegenerationInterval”设置在多少秒之后自动重新生成服务器的密匙（如果使用密匙）。重新生成密匙是为了防止用盗用的密匙解密被截获的信息。
PermitRootLogin no
# “PermitRootLogin”设置root能不能用ssh登录。这个选项一定不要设成“yes”。
IgnoreRhosts yes
# “IgnoreRhosts”设置验证的时候是否使用“rhosts”和“shosts”文件。
IgnoreUserKnownHosts yes
# “IgnoreUserKnownHosts”设置ssh daemon是否在进行RhostsRSAAuthentication安全验证的时候忽略用户的“$HOME/.ssh/known_hosts”
StrictModes yes
# “StrictModes”设置ssh在接收登录请求之前是否检查用户家目录和rhosts文件的权限和所有权。这通常是必要的，因为新手经常会把自己的目录和文件设成任何人都有写权限。
X11Forwarding no
# “X11Forwarding”设置是否允许X11转发。
PrintMotd yes
# “PrintMotd”设置sshd是否在用户登录的时候显示“/etc/motd”中的信息。
SyslogFacility AUTH
# “SyslogFacility”设置在记录来自sshd的消息的时候，是否给出“facility code”。
LogLevel INFO
# “LogLevel”设置记录sshd日志消息的层次。INFO是一个好的选择。查看sshd的man帮助页，已获取更多的信息。
RhostsAuthentication no
# “RhostsAuthentication”设置只用rhosts或“/etc/hosts.equiv”进行安全验证是否已经足够了。
RhostsRSAAuthentication no
# “RhostsRSA”设置是否允许用rhosts或“/etc/hosts.equiv”加上RSA进行安全验证。
RSAAuthentication yes
# “RSAAuthentication”设置是否允许只有RSA安全验证。
PasswordAuthentication no
# “PasswordAuthentication”设置是否允许口令验证。
PermitEmptyPasswords no
# “PermitEmptyPasswords”设置是否允许用口令为空的帐号登录。
AllowUsers admin
# “AllowUsers”的后面可以跟着任意的数量的用户名的匹配串（patterns）或user@host这样的匹配串，这些字符串用空格隔开。主机名可以是DNS名或IP地址。
```

---

# 密钥对配置

在上面的设置中, 我们已经禁止了用户通过提供账号密码这种不太安全的方式进行登录, 因此, 用户只能通过公钥认证这种方式登录. 公钥认证登录方式相对而言安全, 而且方便, 不要输入密码. 关于公钥认证的原理请看上一篇文章[<SSH连接认证原理概述>](). 接下来, 需要生成密钥对, 即一组配对的公钥`Public Key`和私钥`Private Key`. 登录一个普通用户, 利用`$ ssh-keygen -t rsa`命令为其生成密钥对, `-t rsa`开关代表利用RSA算法产生密钥对.
```
[fancyseeker@localhost ~]$ who 
fancyseeker pts/0 2013-12-30 17:49 (192.168.1.9)
[fancyseeker@localhost ~]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/fancyseeker/.ssh/id_rsa): //此处设置私钥存放目录,直接回车即可,将私钥保存在~/.ssh/id_rsa
Enter passphrase (empty for no passphrase): //此处设置公钥认证密码,直接回车表示不设置
Enter same passphrase again:
Your identification has been saved in /home/fancyseeker/.ssh/id_rsa. //私钥
Your public key has been saved in /home/fancyseeker/.ssh/id_rsa.pub. //公钥
The key fingerprint is:
9b:f3:bf:f6:39:70:c9:66:d2:1c:99:c6:71:6a:b8:a9 fancyseeker@localhost.localdomain
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|              . .|
|             o * |
|            . O  |
|        S    O o |
|         o  = O  |
|        +  . *   |
|         oE . .. |
|          .oooo. |
+-----------------+
```
注: 运行`ssh-keygen`命令并不需要切换至特定的用户 (比如你想要通过sample_user这个用户来进行ssh登陆, 但是并不一定要求说需要切换至sample_user来执行`ssh-keygen`这个命令), 同样的也不需要指定在服务器还是客户端执行, 执行`ssh-keygen`的目的仅仅是为了得到一个密钥对, 与谁来执行, 在哪执行没有特别的关系. 当然, 生成的密钥对会默认存放在当前用户的`~/.ssh`文件夹下, 因此如果用特定用户来执行`ssh-keygen`会省事很多. 在得到密钥对之后, 将公钥和私钥设置好权限之后放置在合适的目录下即可. 具体的权限和位置请看接下来的部分.

这时候, 可以查看下`~/.ssh`目录下生成的公钥和私钥文件.
```
[fancyseeker@localhost .ssh]$ ll ~/.ssh
-rw------- 1 fancyseeker fancyseeker 1679 Dec 31 14:12 id_rsa
-rw-r--r-- 1 fancyseeker fancyseeker  410 Dec 31 14:12 id_rsa.pub
```
这里可以看到`ssh-keygen`命令生成了公钥 **`id_rsa.pub`** 和私钥 **`id_rsa`**文件.

接下来, 需要做的就是把公钥文件留在服务器特定用户的 `~/.ssh` 文件夹下, 而将私钥文件放置于客户端.

## 公钥处理

首先, 将公钥 `id_rsa.pub` 文件重命名为 **`authorized_keys`**放置于服务器端指定登陆用户的 `~/.ssh` 文件夹下.
```
[fancyseeker@localhost .ssh]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
这里有几点要说明下, 关于为什么要将公钥名称更改为`authorized_keys`, 这是因为ssh服务会在客户端发起连接请求的时候在对应用户家目录的`.ssh`文件夹下寻找 `authorized_keys` 文件, 并打开访问文件中包含的公钥. 除此之外, `authorized_keys` 文件中可以包含多个公钥, 因此上述的命令中用了`>>`指令将之前生成的公钥追加至`authorized_keys`文件的尾部.

为了使登陆更加安全, 我们需要将公钥的权限设置为只有所有者可读, 同时删除之前的 `id_rsa.pub` 文件, 即
```
[fancyseeker@localhost .ssh]$ chmod 400 ~/.ssh/authorized_keys
[fancyseeker@localhost .ssh]$ rm -f id_rsa.pub
[fancyseeker@localhost .ssh]$ ll
-r-------- 1 fancyseeker fancyseeker  410 Dec 29 20:51 authorized_keys
-rw------- 1 fancyseeker fancyseeker 1679 Dec 31 14:12 id_rsa
```
至此, 服务器端的配置结束. 接下来介绍Windows客户端PuTTY的配置和使用.

---

# Windows下使用PuTTY登陆

[PuTTY](http://www.putty.org/)是Windows环境下一个简单易用的SSH客户端软件.

首先, 我们需要利用一种安全的方式(可以用U盘, 移动硬盘之类的存储介质, 也可以使用scp(secure copy)协议进行安全传输), 将服务器端产生的私钥文件 `id_rsa` 移动到客户端上来.

## 转换私钥格式
由于服务器端产生的私钥文件`id_rsa`的格式PuTTY无法识别, 因此需要先利用`PuTTYgen`工具转换成PuTTY识别的格式.(PuTTYgen工具需要[此处](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)下载)

运行PuTTYgen工具, 点击 Load, 选择要转换的私钥文件.

![PuTTYgen](/img/ssh_connect/puttygen.jpg)

找到之前存放私钥文件的地方.

![](/img/ssh_connect/PuTTYgen_all.jpg)

这里需要将文件名后面的文件类型更改为All File(*.*)才能看到服务器端产生的私钥文件.

![](/img/ssh_connect/PuTTYgen_pk.jpg)

此时, PuTTYgen会要求你输入passphrase口令, 该口令是之前在服务器端利用`ssh-keygen`生成密钥对时,由用户输入的一个口令, 如果为空, 直接回车即可.

![](/img/ssh_connect/passphrase.jpg)

输入口令, 点击ok, PuTTYgen会提示私钥格式转换成功

![](/img/ssh_connect/pksucc.jpg)

点击 Save private key 将转换成功后的 ppk 文件保存至自己想要的指定位置. 此时, 私钥的转换工作结束, 接下来配置PuTTY.

## 导入私钥

双击打开PuTTY, 在左侧的列表中找到`Connection-SSH-Auth`, 在右边的窗口中, 选择需要导入的私钥.

![](/img/ssh_connect/PuTTYconfig.jpg)

然后找到刚才利用PuTTYgen转换得到的ppk私钥文件

![](/img/ssh_connect/pkpos.jpg)

## 配置会话Session

之后点击左侧列表中的Session, 来配置SSH会话, 即连接的具体参数

![](/img/ssh_connect/session.jpg)

此处需要配置服务器的IP地址, 以及要保存的连接名称(任意取), 设定完后点击右侧的 `Save` 按钮将连接保存, 这样下次就不需要设置而能快速访问了.

至此, SSH的服务器端和客户端的配置全部结束, 接下来是测试过程.

## 测试登录

双击打开PuTTY, 选中之前我们保存的名为 Whatever you like 的连接, 点击`Open`, 发起SSH连接
![](/img/ssh_connect/testPuTTY.jpg)
我们会看到一个黑色的连接对话框, 在输入要登陆的用户名以及在生成密钥对时要求的passphrase后, 成功地登陆了远程服务器.

![](/img/ssh_connect/sshlogin.jpg)

有几点需要在本文的最后稍微做下说明.
1. SSH配置的关键在于首先需要先生成一个密钥对, 该密钥对是由哪个用户生成以及是在服务器端生成还是在客户端生成的都不重要, 重要的是最后这个密钥对中的公钥必须要在服务器端的特定用户家目录下的.ssh文件夹中,并命名为authorized_keys,而私钥则必须经由客户端转换后导入客户端软件,这样才能满足SSH连接的需求.
2. 服务器端的公钥权限不对,无法被读取有可能导致在进行SSH登陆的时候被服务器端拒绝.
3. 一个萝卜一个坑, 不同用户不可共同使用同一组密钥对, 否则可能导致登陆失败.
4. 服务器之所以接受该用户利用SSH登陆是因为在服务器端的改用户家目录的.ssh文件夹下有公钥文件, 并且和发起请求客户端的私钥文件是配对的. 显然的, 如果发起请求的用户在服务器端的家目录.ssh文件夹下没有公钥文件, 那么便无法成功登陆.

至此, 本文结束.