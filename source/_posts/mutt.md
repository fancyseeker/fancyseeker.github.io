title: "Mutt: 阅读邮件列表"
date: 2015-08-19 09:42:04
tags: [mutt, tools]
notoc:
---

# 前言

网络上关于介绍Mutt的文章很多, 相信最有名的应该算是王垠的[Mutt使用指南](http://www.ctex.org/documents/shredder/mutt_frame.html)了吧, 这是我看过诸多讲述Mutt文章中讲的最好的, 为什么? 因为这篇文章真的只讲Mutt啊! 当然, 最后我还是无法根据这篇文章把我的Mutt配置用起来就是了. 除此之外, 网上遍布了各式各样的形如`(从零配置)?mutt+(getmail|fetchmail)+(msmtp|esmtp)(+procmail)?收发邮件`之类的文章, 那为什么我还要再造一个轮子, 哦不, 是再写这一篇文章来给互联网添加冗余的信息呢?

- 首先, 介绍下我的使用场景, 我需要使用阅读相关社区的Mailing List, 而邮件列表这种东西大家或多或少了解过, 主要的特点是邮件主题(Thread)多, 回复(Re)多, 引用(Quoted)多, 代码(Patch)多...如果你订阅(Subscribe)了一个邮件列表, 那么一天收个一百来封邮件很正常. 那么问题就来了, 这么多这么乱的邮件, 如何阅读? 如何整理? 我们常用的邮箱诸如Gmail, 163, QQmail etc. 这类的邮箱在日常使用中确实表现不错, 但是碰到邮件列表, 似乎就不怎么好用了. 此外, 还有一些邮件客户端, 诸如Outlook, Thunderbird etc. Thunderbird我没用过就不置评了, Outlook看邮件列表我没试过, 但是就是在Outlook中多几层引用回复我看起来就很费劲了, 可能是比较low吧, 并没有学习如何高效使用神器Outlook = =.

- 其次, 除了邮件又杂又多的问题, 在社区的开发者列表(devel)中经常会包含很多代码, 其中有讨论代码的, 也有是使用`git sendmail`发送的Patch的, 因此邮件内容中会包含很多的代码, 其中的代码还是`git diff`格式的, 增减的代码行首有`+`有`-`, 如何能高亮修改的代码行以及其它信息是高效阅读邮件的关键, 毕竟, 时间就是金钱, 我的朋友 ;-P

- 最后, 当我在网上搜索Mutt的相关介绍时候, 其实我是想找到如何配置Mutt的, 而搜索引擎返回给我的结果也确实是如何配置Mutt的, 但是很遗憾的, Mutt这个工具对于刚接触的人来说实在是太不友好了. 我只想收发邮件, 为什么还要整那么多没用的, 又是getmail, 又是msmtp的, 还有个看起来巨复杂的procmail? 然后当我困惑于这些个东西是什么关系的时候, 文章只是甩了我几个配置文件就完了? 当我半知不解的照猫画虎按照文章内容小心翼翼的编辑完配置文件后, 发现不能用亦或是根本就不对的时候, 我的内心是崩溃的(摔! 谁能告诉我配置文件的参数是什么意思, 为什么要配这个参数, 这个参数还有什么其它值, 行为是怎么样的, 还有没有其它的参数来完成XXX功能? 而当我苦苦在一篇篇文章中苦苦上下求索一无所获满头雾水的时候, 所幸最后Stackoverflow, 官网manual以及man page救了我.

综上, 可以看到, 我需要用Mutt来阅读邮件列表, 之所以选Mutt而不使用传统的邮件客户端或是常用的邮箱是因为其在阅读邮件列表时表现不佳, 缺乏效率. 而当我要着说配置Mutt的时候, 网络上的诸多文章仅仅只能起到参考的作用, 我希望能做到自己来定制Mutt并且是真的了解为什么这么配才去配, 而不是随手拿来一个配置文件就用. 再者, 就是本人有些许强迫症, 像Mutt和其它的一些在Unix-like环境下运行的Tools具有极强定制性的工具, 总是希望能将其尽可能的配置到顺手为止. 因此, 我才花时间来写下这篇文章, 一方面记录自己的配置过程, 方便以后回顾用, 另一方面, 若能顺便帮助下有需要的人, 那也算是意外收获了吧.

# Why: 为什么使用Mutt

如上文所述, 从阅读邮件的角度来说, 有很多很好用的邮箱, Gmail就是本人很喜欢的邮箱, 并且一直在用, 除了一些不可控的原因影响其用户体验外, 其它方面确实表现的很好. 那为什么我还要花这么大的力气去配置Mutt呢? Mutt跟其它的邮箱相比, 到底有哪些好处呢?

邮件归类
Thread模式条理清晰
高亮嵌套引用
Patch高亮

# What: Mutt是什么

Mutt工具的定位, 负责的事情, 因此需要其它工具来一起完成任务, 关系图.
MTA MUA MDA spool 基础知识

## Mutt简介

Mutt是什么? 或者说Mutt是什么样子的? 根据[Mutt官网](http://www.mutt.org)上的介绍, Mutt是Unix系统环境下一个小巧但是强大的文本邮件客户端. 小是显而易见的, 功能强大对于本人而言主要体现在以下几个[Mutt Feature](http://www.mutt.org/#Features):

- 支持配色
- 对邮件列表支持很好
- 高度可定制, 支持键绑定和宏
- 支持正则表达式以及内部模式匹配等多种搜索
- 高效

此外, Mutt还支持其它很多特性, 比如MIME(这是什么我并不知道, 估计也不会用到), PGP(没有加密邮件的习惯), 支持POP3和IMAP协议(是个邮箱都支持), 完全控制邮件头部等等等等...但是这些特性其它的邮箱也有, 有的甚至做的更好.

因此, 对于Mutt, 我最看重的还是其它邮箱所没有的特性.

1. 高度可定制化, 键位绑定和宏对于习惯了Vim和CLI的人来说能提高不少效率;

2. 对邮件列表支持完善, 这也是我之所以选用Mutt最关键的原因, Mutt针对邮件列表添加了很多很方便的设置和操作, 极大的提高了阅读和回复邮件列表邮件的效率;

3. 邮件内容支持配色, 这对于阅读带有代码或是多层嵌套引用的邮件来说简直是神技.

4. 高效, 主要反应在打开一个1000+邮件的信箱只需要1s不到时间, 很快.

诚然Mutt有很多优点, 但是Mutt仅仅只是一个邮件客户端, 那么问题来了, 什么是邮件客户端(Mail Client)? 通常我们认为一个邮件客户端应该像Outlook和Thunderbird那样, 能收能发能整理能查找. 但是很遗憾的, Mutt跟他们并不一样, Mutt是Unix环境下的small and powerful的邮件客户端, 按照Unix大多工具的尿性, 一般只会提供小而精的功能, 并不会做大而全的封装. Mutt也是一样, **Mutt只管理邮件, 而不负责邮件的收发**. 详细点说, Mutt做的事情只是从本机上的某个位置读取邮件, 或是把邮件存放到本机上的某个位置, 此外, 还负责邮件的阅读, 查找, 标记等事务, 与其说Mutt是一个邮件客户端, 不如说**Mutt是一个邮件管理工具**.

## Email工作原理

说到这里, 我们就不得不简单的讲一下Email的工作原理了<sup>[[x]](http://ccm.net/contents/116-how-email-works-mta-mda-mua)</sup>.
![Email的工作原理](/img/mutt/how_email_works.png)

从上面的原理图中, 我们可以看到整个邮件发送接收的过程. 首先, 邮件在MUA(Mail User Agent)中编辑完成, 交由MTA(Mail Transfer Agent)进行发送, 在发送的过程中, 邮件会在多个路由和MTA服务器中中转, 邮件从一个MTA服务器被传输到另一个MTA服务器, 其中使用的是名为[SMTP](https://zh.wikipedia.org/zh/%E7%AE%80%E5%8D%95%E9%82%AE%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)的邮件传输协议. 当邮件到达收件人所处的网络中的MTA时, MTA将邮件交给邮件服务器, 邮件服务器负责将邮件发送给指定用户的信箱中, 邮件服务器在分发邮件的时候, 具有MDA(Mail Delivery Agent)的功能, 其中可能会包含防火墙, 过滤垃圾邮件, 屏蔽黑名单等功能, 我们常见的网络邮箱的工作模式大致是这样的.

另外, 如果我们使用的是邮件客户端来接收邮件的话, 客户端还需要负责从远端的信箱中拖取邮件到本地的信箱中. 这里就需要使用MRA(Mail Retrieval Agent)来完成从远端信箱收取邮件至本地的操作, 在MRA取邮件的时候, 涉及到这么一个问题, 是单纯的将远端邮箱的邮件复制来呢, 还是让本机的信箱跟远端的信箱保持同步呢, 根据收取方式的不同, 区分了两种主流的收取邮件协议[POP3](https://zh.wikipedia.org/wiki/%E9%83%B5%E5%B1%80%E5%8D%94%E5%AE%9A)和[IMAP](https://zh.wikipedia.org/wiki/IMAP). 在邮件到达本地后, 有时在本机上, 会再次对邮件进行一部分分拣和过滤, 例如将来自邮件列表的邮件放到一个特定的文件夹中, 将包含特定文字的邮件放到垃圾邮件文件中等, 这些功能, 是通过本地的MDA来完成的. 最后, 邮件到达了特定的文件夹, MUA, 即邮件客户端从特定文件夹读取邮件, 并解码邮件格式, 展示给用户阅读. 而Mutt, 在整个邮件收发过程中, 做的也就是这个部分的事情!

## Mutt与msmtp, getmail和procmail的关系

解释了半天, 相信大家已经被各种M*A给整懵了吧. 以下对邮件发送过程中的各个部分(Agent)做下简要的解释:

- **MUA(Mail User Agent)**: 邮件客户端, 负责邮件的管理, 阅读等. 一些集成度较高的MUA会带有收发邮件乃至过滤邮件的功能如Outlook和Thunderbird, Mutt也带有简单的发送邮件功能, 不过一般不采用.

- **MTA(Mail Transfer Agent)**: 邮件传送代理, 负责邮件的发送, 处理发送过程中的主机识别, 路由跳转等问题. 对于用户而言MTA就是负责从本地发送邮件的部件. 常见的MTA有sendmail, emstp, msmtp.

- **MRA(Mail Retrieval Agent)**: 邮件收取代理, 负责从远端的邮件服务器收取邮件至本地文件夹. 常见的MRA有fetchmail, getmail.

- **MDA(Mail Delivery Agent)**: 邮件分发代理, 负责根据规则过滤邮件并将邮件投放到不同的文件夹中. 常见的MDA有procmail.

之所以讲了这么多Email的工作原理, 主要是为了两件事情.

1. 明确Mutt在整个邮件收发过程中的位置.
2. 解释为什么在配置Mutt的时候需要额外安装配置msmtp, getmail乃至procmail.

当弄明白了Email的工作原理后, 其实也就不难理解为什么在网上搜索Mutt相关资料的时候, 跳出来的都是mutt+getmail+msmtp+procmail之类的文章了. 因为光一个Mutt, 根本无法完成最基本的邮件收发功能啊, 必须需要靠msmtp来发邮件, 靠getmail来收邮件, 如果还需要自定义一些邮件分类过滤行为的话, 还需要procmail来帮忙. 因此, 现在我们在谈论Mutt的时候, 我们其实实在谈论mutt+getmail+msmtp这一串东西.

# How: 搭建Mutt环境

在明白了为什么用Mutt, 什么是Mutt, 已经Mutt与msmtp, getmail和procmail的关系之后, 接下来, 我们来了解下如何配置Mutt以及msmtp和getmail, 在这节中我们先不介绍procmail, 因为很多的邮箱反垃圾邮件和防火墙已经做的很好了, 用不着用户自定义规则来反垃圾邮件. Mutt+msmtp+getmail已经可以完成基础的收发管理邮件的功能.

## 安装Mutt, msmtp和getmail

这步没什么好说的, 直接apt-get

```shell
$ sudo apt-get install mutt msmtp getmail4
```

## 配置Mutt

Mutt的配置主要分为2个部分: 一个是muttrc文件, 另一个是信箱目录的组织.

### muttrc文件

我们先来看一下基本的muttrc配置文件. 在用户的`$HOME`目录下创建并编辑`.muttrc`文件.

```shell
$ vim ~/.muttrc
```

一个本的muttrc文件如下所示:

```
# 设置发信人名称
set realname = "Your Name"

# 设置邮件发送程序
set sendmail = "/usr/bin/msmtp"

# 默认的发信地址
set from = "yourmail@mail.com"

# 在发送邮件时使用from域作为msmtp发送邮件的sender, 否则使用的是user@localhost
set envelope_from = yes

# mutt自动生成from地址
set use_from = yes

# 设置编辑时使用的编辑器
set editor = "vim"

# 建立信箱, 默认信箱是inbox, 所有新邮件都会被默认发送到inbox文件夹中
# 告知mutt邮件的存放格式(format), maildir格式要求相关文件夹下必须有tmp, new, cur这3个子目录
set mbox_type = maildir
# 存放邮件的根目录, 在muttrc文件中用"+"或者"="表示
set folder = "~/.mail"
# 接收到的邮件默认存放的位置, 系统会自动创建该目录, 而其它的目录需要手动创建
set spoolfile = "+inbox"
# 从收件箱(inbox)保存的邮件存放位置
set mbox = "+mbox"
# 延迟发送(postpone)邮件的存放位置
set postponed = "+postponed"
# 已发送的邮件存放的位置
set record = "+sent"

# 设置终端显示时采用的字符集
set charset = "utf-8"

# 设置发送字符集, 根据linux内核邮件列表推荐设置
set send_charset = "us-ascii:utf-8"

# 回信时引用原文是否加入原文的邮件头
set header = no

# 将inbox中已读邮件自动移动到mbox文件夹中(ask-yes表示会询问, yes则直接移动)
set move = ask-yes

# 在信件内容窗口(pager)滚动到底部时, 不自动跳到下一封邮件
set pager_stop

# 告知mutt订阅的邮件列表,以便开启相关特性,便于阅读邮件列表
subscribe xen-devel@lists.xen.org xen-devel@lists.xensource.com xen-devel@lists.xenproject.org
```

### 指定msmtp作为邮件发送工具

由于我们采用msmtp来发送邮件, 需要告知Mutt, 以便在发送邮件的时候, Mutt可以将邮件交给msmtp进行发送. 因此, 需要指定邮件发送工具路径.

```
set sendmail = "/usr/bin/msmtp"
```

### 设置信箱目录

另外, 我们看到在muttrc配置文件中设置了很多信箱文件夹, 如下:

```
set mbox_type = maildir
set folder = "~/.mail"
set spoolfile = "+inbox"
set mbox = "+mbox"
set postponed = "+postponed"
set record = "+sent"
```

这里就有必要解释下其意思了. 首先, 我们看到的是`mbox_type`这个变量, 这是用来设置邮件储存格式的, 通常将其设置为`maildir`格式<sup>[[x]](http://dev.mutt.org/trac/wiki/MaildirFormat)</sup>. `maildir`格式要求每个信箱文件下要包含3个子文件夹, 分别为`cur`, `new`, `tmp`.

说到邮件储存格式, 我们就来简单了解下Mutt管理邮件的方式. 事实上, 我们通过Mutt看到的邮件, 都是以文件的形式保存在特定的目录中的, 一封邮件就是一个文件. Mutt做的事情就是从特定的信箱目录中读取邮件, 解码文件, 然后展示内容给用户. 在把邮件从一个信箱移动到另一个信箱的时候, Mutt实际上是将邮件文件从一个目录移动或是复制到另一个目录中, 就是这么简单!

接着我们看一下变量`$folder`, 该变量即指定了Mutt读取操作邮件的"根目录", 例如, 示例配置文件中, 将其设置为`~/.mail`目录, 那么在打开Mutt的时候, Mutt会默认从该目录下去读取操作邮件文件.

在信箱组织方面, 一般建议设置`$spoolfile`, `$mbox`, `$postponed` 和 `$record`这四个变量. 当然, 也可以都不设置, 但是Mutt会默认在`$folder`变量指定的目录下建立`inbox`目录, 并将全部的邮件都一股脑放到这个目录下. 这里, 我们都对其进行了设置, 当然, 设置前要确保这些指定目录已经建立好了, 并且每个目录下已经建立了`cur`, `new`, `tmp`这三个子目录.

```
$ mkdir ~/.mail
$ cd ~/.mail
$ mkdir inbox mbox postponed sent
$ cd ~/.mail/inbox
$ mkdir cur new tmp
...
```

这里我们简要的介绍下各个目录的功能<sup>[[x]](http://dev.mutt.org/trac/wiki/MuttGuide/Folders)</sup>

| muttrc中变量 | 设置的目录 | 功能 |
| :--- | :--- | :--- |
| `$spoolfile` | inbox | 最基本的一个目录, 默认所有的新邮件都会存放在这个目录下 |
| `$mbox` | mbox | 已读邮件会被移动至该目录下(需要设置`$move`变量) |
| `$postponed` | postponed | 推迟发送的邮件会被保存在这里 |
| `$record` | sent | 已发送的邮件会被保存在这里 |

P.S. 在指定信箱目录的时候我们使用了`+`符号, 在muttrc文件中, `+`号代表`$folder`变量, 例如示例中设置了`$folder`变量为`~/.mail`目录, 那么`+inbox`表示的就是`~/.mail/inbox`目录<sup>[[x]](http://dev.mutt.org/trac/wiki/MuttGuide/Folders)</sup>.

### Mutt其它配置项

```
set move = ask-yes
```

`$move`变量有四个值, 分别为`yes`, `no`, `ask-yes`和`ask-no`. `$move`变量的意义在于指定是否将已读的邮件自动移动到`mbox`目录中. 默认是不移动, 此处将其设置为`ask-yes`, 即表示自动将已读邮件移动至`mbox`目录中, 但是在移动前会提示用户是否执行. 这样可以保持`inbox`目录的整洁, 避免每次打开mutt, 上来就看到一堆已经读过的邮件.

```
set charset = "utf-8"
set send_charset = "us-ascii:utf-8"
```

这里主要是字符集的设置, 因为本人使用Mutt来阅读以及回复邮件列表, 因此需要设置通用的字符集来避免收信人接受到乱码邮件的问题, 此处的设置是参考Linux内核邮件列表对邮件客户端的建议[设置](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/email-clients.txt?id=HEAD)

关于其它项的配置, 注释已经写的比较详细了, 就不再赘述了. 最后, 实际上Mutt是带有简单的收发邮件模块的, 因此有些用户可能需要在`.muttrc`文件中写入邮箱的账户和明文密码, 因此, 最好还是需要将`.muttrc`的权限设置为600.

```shell
$ chmod 600 ~/.muttrc
```

## 配置msmtp

msmtp的配置文件为`~/.msmtprc`.

```shell
$ vim ~/.msmtprc
```

一个基本的msmtprc文件如下:

```
defaults
tls off
tls_certcheck off
logfile ~/.mail/msmtp.log

account your_account
host $smtp.your_smtp_server.com
from $your_mailaddress@mail.com
port 25
user $username
password $password

# Set a default account
account default : $username
```

msmtp的配置相对比较简单, 文件中带`$`符号的都是需要根据具体情况修改的信息.

由于msmtp是负责发送邮件的, 而之前我们说过, 邮件发送涉及到的协议主要是SMTP, 因此, 需要告知msmtp SMTP服务器的host地址, 一般的邮箱STMP地址都比较简单, 例如新浪邮箱的host为`smtp.sina.com`, Gmail的host为`smtp.gmail.com`. 而大部分邮箱的SMTP服务端口号为默认的25, Gmail的话使用的是465和587, 具体的还请查看官方的介绍.

在知晓了SMTP服务器地址后, msmtp会访问SMTP服务器, 此时需要登录账户密码, 因此, 这些信息也要写入msmtprc以告知msmtp, 需要注意的是, 密码是明文保存的, 因此, 安全起见, 需要设置msmtprc文件的权限.

```shell
$ chmod 600 ~/.msmtprc
```

P.S. 由于本人使用的是公司的邮箱, 因此需要关闭TLS. 但是如果使用的Gmail则需要相关的设置, 具体方法网上随便一搜都是, 不再赘述.

在配置msmtp后, 不妨测试下在Mutt中能否正常发送邮件:

```shell
$ echo "测试邮件内容" | mutt -s "测试邮件标题" test_mail@address.com
```

如果在测试邮箱中收到测试邮件, 即表明msmtp配置成功, 并且Mutt能顺利调用msmtp来发送邮件.


## 配置getmail




# Misc:

配置procmail 配色 键位绑定



























