title: "fedora 17 更换登陆背景图片"
date: 2013-02-20 16:27:12
tags: [Fedora, Linux, gnome, maintenance]
---

新年各种繁琐的事情解决完，终于能有点时间折腾下电脑了。自从装完fedora 17后一直感觉fedora 17登陆界面的背景不怎么好看，遂换之。本文仅作备忘使用。

![Beefy Miracle 默认背景图片](/img/f17bg/beefy-miracle.png)

# 切换至root
```
$ su -
```

# 切换至背景配置目录下
```
# cd /usr/share/backgrounds/beefy-miracle/default/
```
注意路径中的`beefy-miracle`为fedora 17的代号，如果fedora的版本为15那么代号就为`Lovelock`，16的话就是`Verne`，根据自己的fedora版本而定。
有关fedora各版本的代号参见：[](http://zh.wikipedia.org/wiki/Fedora) 中 `发布历史` 一节。

在`default`目录下，我们可以发现有一个`beefy-miracle.xml`文件和`normalish`, `standard`, `wide` 三个文件夹。

![default目录](/img/f17bg/bmbg.png)

此处的`beefy-miracle.xml`文件记录了背景图片的配置信息。我们可以打开看一下里面的内容：

  
```bash
 <!-- Wide 16:10 -->
 /usr/share/backgrounds/beefy-miracle/default/wide/my_wallpaper.jpg
 <!-- Standard 4:3 -->
 /usr/share/backgrounds/beefy-miracle/default/standard/beefy-miracle.png
 <!-- Normalish 5:4 -->
 /usr/share/backgrounds/beefy-miracle/default/normalish/beefy-miracle.png
```
这里我们主要关注下`file`标签内的内容。可以清楚的看到, `size` 标签内的指定了不同屏幕长宽比下的背景图片的路径。

`wide`文件夹下的背景图像为16:10的, `standard`文件夹下为4:3的, `normalish`文件夹下为5:4 的。(等等，好像缺了点什么，16:9的到哪去了，我的笔记本是16:9的啊，这不科学= =)
那么，如果长宽比是16:9的同学，先暂且把16:9和16:10看成一样的吧，因为它们都是`wide`的嘛 :-D

# 替换背景图片
接下来思路就很清晰了，我们只要把对应文件夹下的图片替换成我们自定义的图片或者更改`beefy-miracle.xml`文件中对应的路径即可。

由于本人的屏幕长宽比为16:9，所以我选了一张个人认为还算好看的16:9壁纸，将其放到 `/usr/share/backgrounds/beefy-miracle/default/wide/`目录下，并命名为`my_wallpaper.jgp`(对，你没有看错，jpg文件也是可以的！) 然后再将`beefy-miracle.xml`文件中对应的路径作如下修改：
```
/usr/share/backgrounds/beefy-miracle/default/wide/my_wallpaper.jpg
```

# 重启X Windows
按`Ctrl + Alt + Backspace`重启 X Window，即可看到新的登陆背景啦～

最后，贴一下我的登陆背景图片作为本文结尾吧～

![](/img/f17bg/my_wallpaper.jpg)

(此壁纸来源于国外壁纸网站: ~~http://wallbase.cc~~(wallbase已经停止服务) http://alpha.wallhaven.cc 个人认为是个相当不错的壁纸网站，推荐之)