title: "SSH连接认证原理概述"
date: 2013-12-30 13:54:46
tags: [ssh]
---

# SSH简要介绍

SSH的英文全称为Secure Shell，是IETF（Internet Engineering Task Force）的Network Working Group所制定的一族协议，其目的是要在非安全网络上提供安全的远程登录和其他安全网络服务。<sup>[[1]](http://blog.csdn.net/oncoding/article/details/4365062)</sup>用于在主机之间建立起安全连接, 并加密传输内容, 以达到安全的远程访问, 操作以及数据传输的目的.

SSH主要有两个特点: 1. 安全性 2. 传输速度快

为什么要强调SSH的安全特性, 因为传统的网络服务程序，如 FTP、POP 和 Telnet 其本质上都是不安全的；因为它们在网络上用明文传送数据、用户帐号和用户口令，很容易受到中间人（man-in-the-middle）攻击方式的攻击。就是存在另一个人或者一台机器冒充真正的服务器接收用户传给服务器的数据，然后再冒充用户把数据传给真正的服务器。 而 SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。透过 SSH 可以对所有传输的数据进行加密，也能够防止 DNS 欺骗和 IP 欺骗。<sup>[[2]](http://biaobiaoqi.me/blog/2013/04/19/use-ssh/)</sup>

SSH的传输速度快是因为SSH传输的数据都是经过压缩的, 自然传输速度快.

# SSH认证原理简述

在介绍SSH具体的认证原理之前, 先简单的介绍下关于公钥和密钥加密的概念.所谓的公钥加密 (public-key cryptography)，或非对称密钥加密 (asymmetric key cryptography) 是一类广泛使用的加密算法。这类算法使用一对(pair)密钥即**公钥 (public key)** 和**私钥 (private key)**。 其中公钥可以随便分发，只用于加密 (encryption)，私钥则只由一人持有，只用于解密。通过公钥加密过的密文使用密钥可以轻松解密, 但根据公钥来猜测私钥却十分困难.

因此, 我们可以很容易的发现这么一个事实, 一个消息在用公钥加密后, 哪怕在传输中被人截获, 如果没有私钥, 是无法解密获得其内容的, 只有拥有私钥的人才能成功解密消息. 所以私钥应该由消息的接收方紧紧攥在手里, 不应让他人得知.

公钥加密的关键点在于，一方面，公钥加密是可逆的，但是不能用公钥推断出私钥。显然数学上，已知一个公钥是能够算出对应私钥的，但是只要设计足够好的加密算法（以及使用足够复杂的密钥对），使得不能在可以接受的时间内破译即可。

RSA<sup>[[3]](http://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95)</sup> 是一种常见的公钥加密算法。RSA 的工作原理依赖于如下事实：破译 RSA 私钥需要对某些极大的整数进行因数分解，而目前尚未找到快速的对极大整数作因数分解的算法。换言之，如果有人找到了这样的算法，那么全世界的 RSA 加密都会失效。

RSA是由Ron Rivest, Adi Shamir, Leonard Adleman三人在1978年首次提出的.三人并因此项工作荣获了2002年[Turing Award](http://en.wikipedia.org/wiki/Turing_award). Rivest还是[算法导论](http://book.douban.com/subject/1885170/)的作者之一, 书中在31章对RSA系统的原理进行了简要说明, 系统实现中利用到了数论中的[Euler-Fermat theorem](http://en.wikipedia.org/wiki/Euler%E2%80%93Fermat_theorem).<sup>[[4]](https://wiki.tuna.tsinghua.edu.cn/SshKeyHowto)</sup>

一个基于RSA的**公钥**大概如下所示
```
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAvSkEZ0fKXRqQ/DkjCfsAETsQgV8OR/RVQmwBk/J5IWoknf8Dr
y5kOs+1bnx9zaf8oIcVuXf0jRxTccLBOXiReFJE4aD2rWO33sqA0M4qP1ESYhsU4yokRA0IMDJ62JUv2cWVJg
GpeQriol2t7mH8E6aB8OiJ+NgRbh6+/0LbtQs40VA2+W5PtaBwT4sjv9LOHIdzQcsEeCM8MIHqmXHst7/JuVI
i7wLCxB7Ur8qtwZ2/Ii8Ckjfo6mikWmSh6mRMq9qn0FkMkPCcpm8o4f1zJWOuf+RnjPpopFTqIa8JssMHJMuQ
cCm3EHDkBHjLk/SkidWOzqOtSvUeGKieWiijuw== username@localhost.localdomain
```
与之配对的**私钥**形如
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAvSkEZ0fKXRqQ/DkjCfsAETsQgV8OR/RVQmwBk/J5IWoknf8D
ry5kOs+1bnx9zaf8oIcVuXf0jRxTccLBOXiReFJE4aD2rWO33sqA0M4qP1ESYhsU
4yokRA0IMDJ62JUv2cWVJgGpeQriol2t7mH8E6aB8OiJ+NgRbh6+/0LbtQs40VA2
+W5PtaBwT4sjv9LOHIdzQcsEeCM8MIHqmXHst7/JuVIi7wLCxB7Ur8qtwZ2/Ii8C
kjfo6mikWmSh6mRMq9qn0FkMkPCcpm8o4f1zJWOuf+RnjPpopFTqIa8JssMHJMuQ
cCm3EHDkBHjLk/SkidWOzqOtSvUeGKieWiijuwIBIwKCAQEAjITesx9jIJdkY5go
qFQOrbbZD6Wy1l27ra9RoRqF3k7ZX2z7bDEXQaGcuHm8iiUEzwVDVpOfuUg9/LyP
icdHffP4p5wk9dUMPxoWjHvk3pP/BwzNsBCtOd3LkYSVxXYji9SaroTkS0nqL3jK
WVAaV72FGVxJPINAJer0SJgQ7OJ7f0kZJYoPwG0VpMTLOsmulH7/M5e9i+kwDLmH
+TMvJ3y8gXjc/Sj5UqN/JQIY+h07UxYGvoju/isqFHh8OmahsjJ3Nb3QsUEkC/Kj
cKG2K+mUzTe/4/XQuqMadrjX82QF2b2gL+cdvUAwLn55boXmrtUzxEPo4xxiDNzY
AbYxSwKBgQDxIkaa+EPSJXYej/5YzJLJLVsnb9ygqe/0fM0x022J6Z+0Yq9AIwJ7
N7dvMgNSROLoJ17Oij/N7aoiAZy1KHg1vOoqx6T6BClay8I1+dOsbY6Fi0QbkZeI
Rr6CDueq5PBR8j/oghNhQeFfJUEICBFU6Z7+uFwxuAtQqqdFHyDhSwKBgQDI0nX6
Jz9oW+Y+dXAvhpL8Yln0R45ufiqatkTQgGbeZ70XlKeR7VzQiXecy/XDteJCMlnv
4BhjHL967nSAmrCza13I3eXIYxbzypeNC57bVVXM9BBTg5f4nWwTN+I8Gd/BA5Y8
mekQPBuSyAOp7ULAecjKZrD9Jhw1vybT/aYxUQKBgQDcdxv7ZqRoXMPEK+E7PrIX
BOWgZkYPPEkaC7RKz+8fAXwSo15mhmirK6BlqhGqTZxB+BwqjQcirWhZmxLuxeo/
wqo1vdiqEm7z7X5dPC8+kA1G5bqc3OJQtbV+OYNaaupZjQc6+pVgPDvEtFi24s4E
fdMyB6S/vjY7H67gHHXVSwKBgQC3m9mUQSQHpHq8wyS9vN4olG+AQWxHw88uXYC+
oUgbzI+gh+mp/ZawCKff0GurnvrALgkV1DON0SQYn4B1lL7P7SKLw5BCLrXmNZHg
Cp/eeegL19RpnOK3a1t/SQlbhV7cWw0ENPJX/HD7OocBwvszoouxvPmW/kWte5E2
```
SSH 连接是 `C/S` 模型, 客户端发出连接申请, 服务器对客户端进行验证, 再考虑是否接受连接申请.

# SSH具体认证过程

SSH提供了两种认证的方式, 分别为:

## Password认证

即账号口令验证, SSH的实现方式是,

1. 客户端向 ssh 服务器发出请求, 服务器将自己的公钥返回给客户端;
2. 客户端用服务器的公钥加密自己的登录密码, 再将信息发送给服务器;
3. 服务器接收到客户端传送的密码, 用自己的私钥解码, 如果结果正确, 则同意登录, 建立起连接.

这种方式还是存在漏洞, 中间人可以假扮成服务器, 骗取客户端的密码.<sup>[[2]](http://biaobiaoqi.me/blog/2013/04/19/use-ssh/)</sup>而且在每次登陆的时候都需要输入密码, 所以一般不采用该验证方式.

## Public Key认证

Public Key认证利用公钥私钥对进行认证, 在请求连接时不需要输入密码, 并且由于整个请求连接和通信过程全程加密, 因此安全性高.

SSH协议第二版中有RSA和DSA两种算法认证, 其中RSA加密验证比较常用. RSA加密验证方式，充分利用了非对称加密体系的优势，不需要在网络传输密码，完全杜绝了中间人攻击的可能。

认证的具体步骤如下:

### 准备工作

1. 客户端先使用 `ssh-keygen` 命令, 生成公钥和私钥. 按照默认配置, 私钥会被保存在`~/.ssh/id_rsa` 中, 公钥则在`~/.ssh/id_rsa.pub` 中.

2. 客户端通过安全的方式(一般通过scp(Secure Copy Protocol)方式或者U盘拷贝的方式)将公钥发送给服务器. 在服务器端, 将客户端发的公钥写入到`~/.ssh/authorized_keys` 文件末尾.

*(其实公钥和私钥不是必须在客户端生成, 在服务器端生成也是可以的, 在哪生成的对公钥和私钥并没有任何影响, 重要的是二者是配对的, 而准备阶段其实是要保证这么一个事实, 即服务器端握有公钥, 并在对应账户的家目录下的`.ssh/authorized_keys`文件中保存以便验证程序访问. 请求端握有私钥. 至于为什么要将公钥和私钥这么分配, 接下来将会解释.)*

### 建立连接

在建立连接的时候, 涉及到2对密钥, 其中一对为准备阶段产生并分配好的密钥对, 另一对为服务器在接收到一个连接请求时生成的密钥对. 为了讲述方便, 我们将这两组密钥对表示如下

|Symbol|Description|
|:-:|:-|
|PubC| 客户端密钥对应的公钥|
|PrvC| 客户端握有的私钥|
|PubS| 服务器端产生的公钥|
|PrvS| 服务器端产生的私钥|

1. **认证**

①服务器生成随机数(称之为challenge) `x`, 并用 `PubC` 加密后生成结果 `s(x)`, 发送给客户端.

②客户端使用 `PrvC` 解密 `s(x)` 得到 `x`, 再将`x`用`PubS`加密发送回服务器端.

③服务器端使用`PrvS`解密得到`x`, 进行核对, 如果正确则链接正式成立.

2. **通信加密**

在请求连接前, 服务器端和客户端拥有的密钥为
```
Server          |           Client
--------------------------------------
PubC            |            PrvC
```

①客户端发出申请. 服务器会产生一组 `session` 密钥对, 即`PubS`和`PrvS`.

此时服务器端和客户端拥有的密钥如下所示
```
Server           |           Client
-------------------------------------
PubC             |            PrvC
PubS             |
PrvS             |
```

②服务器端利用客户端的公钥`PubC`对 `session` 公钥PubS进行加密后发送给客户端.

③客户端用自己的密钥`PrvC`解密信息，得到 `session` 公钥`PubS`。
```
Server           |           Client
-------------------------------------
PubC             |            PubS
PrvS             |            PrvC
PubS             |
```

④之后的数据交互，都通过对方方公钥加密，对方收到信息后，用其私钥解密，实现安全加密过程。<sup>[[2]](http://biaobiaoqi.me/blog/2013/04/19/use-ssh/)[[5]](http://tech.idv2.com/2006/10/21/ssh-rsa-auth/)</sup>


# 参考引用

至此, 关于SSH原理性部分的讲解结束. 以下是相关的参考引用出处:

1. [SSH协议基础](http://blog.csdn.net/oncoding/article/details/4365062)
2. [SSH原理和使用](http://biaobiaoqi.me/blog/2013/04/19/use-ssh/)
3. [RSA加密算法](http://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95)
4. [使用 RSA 密钥对进行 SSH 登录验证](https://wiki.tuna.tsinghua.edu.cn/SshKeyHowto)
5. [ssh 公钥方式认证攻略](http://tech.idv2.com/2006/10/21/ssh-rsa-auth/)
[SSH隧道加密技术](http://en.flossmanuals.net/circumvention-tools-zh/advanced-techniques/ssh/)
[MD5 wikipedia](http://zh.wikipedia.org/wiki/MD5)

关于SSH的具体配置以及在windows下通过SSH远程登录Linux服务器的详细操作过程请看下一篇文章[SSH配置-在Windows下远程登陆Linux服务器Shell]().