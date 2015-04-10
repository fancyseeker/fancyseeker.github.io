title: "Vim(2) -- Vim插件介绍及配置"
date: 2014-05-16 11:16:53
tags: [vim]
---

单独的Vim本身在文本编辑方面已经足够强大，但是针对一些特殊的场合还有很多可以提高和改进的空间，这也是插件的意义所在。你用Vim写脚本吗？你用Vim写C/C++吗？你用Vim写HTML、CSS、javascript吗？你用Vim写Ruby吗？你用Vim看项目代码吗......那么各式各样的Vim插件总有一款或几款可以满足你的需求。<!-- more -->

# 前言

掌握Vim确实没有什么捷径，如果非要说有的话，那便是唯手熟而。当你渐渐习惯用hjkl来移动光标，自如的在各个模式下切换，会用一些基本的指令，心里还不那么排斥Vim的话，那么我想接下来要做的就是在实际使用过程中进一步打磨你的Vim，让Vim真正融入到你的生活工作中去了。

单独的Vim本身在文本编辑方面已经足够强大，但是针对一些特殊的场合还有很多可以提高和改进的空间，这也是插件的意义所在。你用Vim写脚本吗？你用Vim写C/C++吗？你用Vim写HTML、CSS、javascript吗？你用Vim写Ruby吗？你用Vim看项目代码吗......那么各式各样的Vim插件总有一款或几款可以满足你的需求。由于插件不属于“标准的”Vim集合，且每个人的工作内容不同因此不同人拥有不同的插件集也是很正常的事情，也正因此，网络上才会有花样繁多的Vim配置方案，多到让人眼花缭乱却无从下手。

本人的主要工作环境是Linux Mint，采用C/C++进行开发附带python和一些shell脚本，这里以本人的Vim插件配置举例，讲解在配置Vim插件过程中的一般性和部分具体性方法。首先上张图来看看吧～
![My Vim](/img/vim2/Screenshot_for_my_vim.png)

相比于原始的Vim，安装了插件后的Vim界面显得非常的丰(hua)富(shao)，这些插件中有一部分是可以显示在窗口上的，一部分则是在使用过程中才能感受到的。从上图中可以看到，整个Vim窗口被划分成了两列，左边为侧边栏，包含文件浏览器以及下方的程序符号列表，在符号列表很清晰的划分了宏定义，变量和函数等信息。窗口的上方还将使用中的buffer显示出来(默认情况下buffer是隐藏的)。右侧则为主要的编辑区，编辑区下方的为酷炫的状态栏～

**Vim Buffer简介**

所谓的buffer，实际上是Vim用于保存工作中的文件信息的，每个打开的文件都对应一个buffer，对文件的操作会保存在buffer中，当关闭文件的时候，根据用户的选择，可以将buffer中的修改内容写入硬盘，或是丢弃buffer中的修改内容。其实文件buffer的概念随处可见，最经常看到的就是你打开文本文档或者Word并输入一些内容后，在没有保存的情况下就关闭程序，会出现是否保存或者放弃保存直接退出的提示，而这些修改的内容就是暂存在buffer中的。只是在windows下，文本文档和word的文件缓存是看不到的(word会定时将缓存写入隐藏的临时文件来保存进度防止意外断电)，而对于Vim，我们则称文件缓存为Buffer。

从上图中的buffer显示条中可以看到，Vim一共打开了4个文件，即有4个对应的buffer，每个buffer有自己的编号(这个编号是Vim定的，保证唯一，但是可能不是顺序的)。

---

# 插件概览

本人安装的Vim插件包括以下几个：

|Plugin|Description|
|:-|:-|
|[vundle](https://github.com/gmarik/Vundle.vim)| 一款用于管理插件的插件，操作简单方便，是主流的管理插件。|
|[ctrlp](https://github.com/kien/ctrlp.vim)| 一款非常好用的模糊查找文件插件。|
|[vim-airline](https://github.com/bling/vim-airline)| B格很高的一个状态栏增强插件，同时带有buffer显示条(tabline)，并集成了多款主流插件。|
|[vim-Bbye](https://github.com/moll/vim-bbye)| 解决了当关闭当前buffer时导致的窗口关闭以及布局混乱等问题。|
|[nerdcommenter](https://github.com/scrooloose/nerdcommenter)| 快速注释工具，同时根据文件类型自动选择对应注释。|
|[nerdtree](https://github.com/scrooloose/nerdtree)| 一款用于浏览文件或目录的插件，效果相当于嵌入Vim内的资源管理器。|
|[taglist](https://github.com/vim-scripts/taglist.vim)| 用于显示程序中的符号列表，并进行简单的查看插件。|
|[winmanager](https://github.com/vim-scripts/winmanager)| Vim窗口管理插件，主要用于将nerdtree和taglist整齐的安放在一个“侧边栏”中。|
|[a.vim](https://github.com/vim-scripts/a.vim)| 用于快速的在*.c/*.cpp和*.h文件之间进行切换。|
|[OmniCppComplete](https://github.com/vim-scripts/OmniCppComplete)| 老牌C/C++自动补全插件，不过似乎存在更好的插件，未研究。|
 
---

# 插件安装

插件的安装方式大致上分为两种：传统的安装方式和基于包管理的插件安装方式。

## 传统的安装方式(不推荐)

我们先来介绍一下传统的安装方式是什么样的。插件实际上是由vim脚本和少数其它类型的文件组成，插件一般位于用户家目录的 `.vim` 文件夹下，即 `～/.vim/` (如果没有，请自行创建)。

安装的流程：

1. 从[www.vim.org/scripts/](http://www.vim.org/scripts/)或者[github.com](https://github.com/)上找到对应的插件并下载。
2. 解压下载得到的插件压缩包，得到若干个文件及文件夹。
3. 将解压得到的文件或文件夹放到～/.vim/ 文件夹下，如果遇到和之前插件同名的文件夹，则选择合并。
4. 重启Vim,插件生效。

总结一下传统的安装方法，实际上就是找到`插件--下载--解压--放到.vim文件夹下--完成`。(不过，对于某些特定的插件，可能有不同的安装方式，有的直接提供了可执行的安装脚本，有的则要求需要一些库的支持或者需要进行一定的配置后才能使用。好在这些都是少数情况，大部分的插件的安装都是相当简单的。)

**注意：我们并不提倡用传统的方式安装插件**

传统的插件安装方式存在一些弊端。首先，所有插件不加区分和归类的直接丢到 `～/.vim/` 文件夹下，非常混乱；其次，插件的管理不方便，比如要想删除一个插件，那么必须要到 `～/.vim/` 文件夹下找到插件对应的文件一一删除，如果想要更新插件，那么就必须重新下载新版本的插件再次安装。

让我们举个例子直观感受下传统的插件安装方式：以下是`nerdtree`和`ctrlp`插件解压得到的文件和文件夹。
```
nerdtree
|--autoload/
|--doc/
|--lib/
|--nerdtree_plugin/
|--plugin/
|--syntax/
|--README.markdown
```

```
ctrlp
|--autoload/
|--doc/
|--plugin/
|--readme.md
[/ezcol_1half_end]
```

当安装这两个插件时候，二者都放到了 `～/.vim/` 文件夹下，其中 `autoload`，`doc`，`plugin` 目录整合到了一起。当要删除 `ctrlp` 插件的时候，需要到删除 `ctrlp` 对应的文件，就要到 `autoload`，`doc`，`plugin` 目录中一个个找属于 `ctrlp` 插件的文件进行删除，可想而知是多么的麻烦。同时无法自动更新插件，需要通过下载覆盖来进行手动更新。

仔细观察不难看出，传统的插件安装方式的弊端主要源自于其混乱的插件管理方式。那么很自然的，人们就会希望能够将插件各自分开来进行存放，同时尽量避免手动的管理插件。这也是为什么后来会出现多款优秀的基于包(bundle)的插件管理工具的原因。

 

## 插件管理工具--Vundle

出于传统Vim插件安装及管理方式的弊端，目前有许多插件管理工具，pathogen、NeoBundle、Vundle、VAM等。pathogen我用过，之后转投Vundle了，至于NeoBundle和VAM没有用过，就不发言了，功能应该大致一样。这里选用目前最主流的Vundle来进行讲解。

Vundle本身也是一个Vim插件，主要干了两件事情：

1.将不同插件放置到 `～/.vim/` 下的对应文件夹进行分别管理。
2.集成了git操作，自动安装更新或卸载插件。

### Vundle的优点
- 其中插件存放在 `～/.vim/bundle/` 目录下，例如`nerdtree`存放在 `～/.vim/bundle/nerdtree/` 目录下，`taglist` 存放在 `～/.vim/bundle/taglist/` 目录下，这样各个插件的目录就非常的清晰明了。

- 对于安装更新卸载插件，只需要在vimrc文件中写入要安装的插件，通过对应的Vim命令即可自动从github或vim.org上自动获取或更新该插件，要卸载时只需将vimrc文件中对应的插件名删除，再运行对应命令即可自动删除插件。

*注：可能有同学git和github比较陌生，git是一个一个分布式的版本控制工具，简单点说就是可以随时把代码放进数据库并加以管理，同时可以随时提取出某个时候放进数据库的代码的一个代码管理工具，除了本地数据库，git还可以从远程服务器上获取或提交代码，而github.com就是一个提供这种代码托管服务的网站，上面存放着成千上万的代码库。基本上我们知道的vim插件源码在github上都能找到，而Vundle的插件安装更新操作就是采用git从github上把对应插件拖到你本地的电脑上，因此Vundle需要git的支持，也就是说你的电脑上需要事先安装有git，现在主流的Linux发行版都是默认安装有git的。如果对git和github比较感兴趣的同学，可以看看[Pro Git](http://git-scm.com/book/zh/v1) 和[GotGitHub](http://www.worldhello.net/gotgithub/)。*

### 安装配置Vundle

关于Vundle的最新信息以及安装方法具体请参见Vundle的github主页：[github.com/gmarik/Vundle.vim](https://github.com/gmarik/Vundle.vim) 。以下对Vundle的安装过程新进适当的描述。

1. 安装Vundle前，我们需要一个干净的 `～/.vim/` 目录。如果你之前采用传统的插件安装方式将插件安装在 `～/.vim/` 目录下，那么你可以备份原先的 `~/.vim/` 目录：`$ mv ~/.vim ~/.vim.bak` 。然后在新建一个空的`.vim`目录：`mkdir ～/.vim` ，这样我们就得到了一个空的 `～/.vim/` 目录，便于之后的Vundle安装。

2. 通过git下载并安装Vundle(实际上就是通过git工具将最新的Vundle下载到对应的目录下)：
```
$ git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

3. 接下来，对Vundle进行配置。Vundle的配置过程是在vimrc文件中添加相应的语句。以下是插件官方给出的示例：
```
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'gmarik/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Avoid a name conflict with L9
Plugin 'user/L9', {'name': 'newL9'}

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList          - list configured plugins
" :PluginInstall(!)    - install (update) plugins
" :PluginSearch(!) foo - search (or refresh cache first) for foo
" :PluginClean(!)      - confirm (or auto-approve) removal of unused plugins
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
```
对于Vundle的配置语句，其实没多少，上面这么长的一段只是详细的讲解了Vundle是如何配置和使用的。这里我们需要修改的就只是`call vundle#begin()` 和 `call vundle#end()` 之间的部分，我们需要在这里添加我们需要的插件信息，以便Vundle可以根据这些信息为我们自动安装插件。Vundle支持4种插件搜索添加方式，分别为托管在github上的插件、存放在vim-scripts.org上的插件、未托管在github上但是可以通过git协议访问到的插件、本地(正在开发的)插件。由于我们需要的插件要不是托管在github上，要不就是在github上有镜像，简单点说就是在github上都能找得到，因此我们只要在vimrc文件中给出github上插件的对应信息就可以了，具体格式为`Plugin '开发者用户名/项目名字'` 例如，我想安装vim-airline插件，那可以在github.com上搜索vim-airline项目，可以找到项目主页的链接为：https://github.com/bling/vim-airline ，则我只需要在vimrc的对应行中写入`Plugin 'bling/vim-airline'` 。

明白了Vundle配置的具体方法，让我们打开 `～/.vimrc` 文件，在 `.vimrc` 文件的顶部(也可以是任意位置)，添加如下语句：
```
" Vundle设置
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'gmarik/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'scrooloose/nerdtree'
Plugin 'scrooloose/nerdcommenter'
Plugin 'vim-scripts/taglist.vim'
Plugin 'bling/vim-airline'
Plugin 'vim-scripts/winmanager'
Plugin 'vim-scripts/a.vim'
Plugin 'kien/ctrlp.vim'
Plugin 'vim-scripts/OmniCppComplete'
Plugin 'moll/vim-bbye'
" plugin from http://vim-scripts.org/vim/scripts.html
" Plugin 'L9'
" Git plugin not hosted on GitHub
" Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
" Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
" Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Avoid a name conflict with L9
" Plugin 'user/L9', {'name': 'newL9'}

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
```
这样，Vundle就知道了我们所需要的全部插件。

 安装或更新插件。打开vim，输入`:PluginInstall`  即可将vimrc中配置的插件下载安装，如果之前已经安装过，则进行更新。或者直接在终端输入 `vim +PluginInstall +qall` 效果也相同。是不是很方便～如果想卸载插件，只要将vimrc中插件对应的行删除，然后打开vim，输入 `:PluginClean` 即可。
 注：在安装或更新过程中Vundle需要从github上下载数据，因此Vim界面无法进行正常操作，但下方的状态栏会显示正在处理，所以只需耐心等待即可。

至此，插件就已经安装完成了。接下来还需要对每个插件进行单独的配置。

注：就插件管理而言，Vundle已经做的足够优秀，但是事情往往可以变得更好。尽管可能遇到的机会不多，但是试想一下，如果你新买了一台电脑，或是电脑硬盘故障导致数据丢失了又或者你在操作一台不常用的电脑，是不是每次都要安装Vundle，更要命的是每次都要修改vimrc文件，要知道vimrc文件里面出了基本的vim设置外还有相当大一部分的插件设置，对Vim环境进行恢复的代价还是略大了点。因此，我们总希望我们的Vim配置能够保存在云端，以便我们随时可以通过一条简单的命令就可以把一切恢复到熟悉的模样。要想完成这点，需要在配置完Vim和插件后，将 `./.vim/` 目录以及 `./.vimrc` 文件都托管到自己github的上，具体方法在接下来的文章会讲到，现在，让我们先专注于把插件配置好: - )

---

# 插件配置及使用

在使用插件的过程中，往往会涉及到很多插件的具体命令，而这些命令往往比较长，并且夹杂着大小写，输入起来比较繁琐，因此往往使用键位绑定和映射来解决这个问题，简单点说就是设置快捷键。而Vim中的操作相当的多，不用说单个的按键了，即便是像 `ctrl+××`，`alt+××` 或是 `ctrl+shift+××` 这样的组合键都有对应的操作，因此我们会经常看到vimrc文件中采用 `<leader>`键和其它按键配合进行键盘映射。vimrc文件中的 `<leader>`实际上也是一个按键，默认为回车键上方的`\`键，可以根据用户需要修改。本人就将 `<leader>` 设置成了逗号`,`。修改 `<leader>`键只需要在vimrc文件中添加：
```
" leader键
let mapleader=","
let g:mapleader=","
```
注：插件的具体配置信息一般都是记录在vimrc文件中，以下介绍插件的配置语句全是添加在 `～/.vimrc` 文件中的。

## nerdtree

在 `~/.vimrc` 文件中添加以下内容：
```
" nerdtree设置
" 控制当光标移动超过一定距离时，是否自动将焦点调整到屏中心
let NERDTreeAutoCenter=1
" 指定鼠标模式（1.双击打开；2.单目录双文件；3.单击打开）
" let NERDTreeMouseMode=2
" 是否默认显示书签列表
" let NERDTreeShowBookmarks=1
" 是否默认显示文件
let NERDTreeShowFiles=1
" 是否默认显示隐藏文件
" let NERDTreeShowHidden=1
" 是否默认显示行号
" let NERDTreeShowLineNumbers=1
" 窗口位置（'left' or 'right'）
" let NERDTreeWinPos='left'
" 窗口宽
let NERDTreeWinSize=31
```
上面只添加了一些常用的设置，更具体的设置请在vim中输入 `:help nerdtree` 查看。

nerdtree基本操作

|key|Description|
|:-|:-|
|`o`|在已有窗口中打开文件、目录或书签，并跳到该窗口|
|`go`|在已有窗口 中打开文件、目录或书签，但不跳到该窗口|
|`x`|合拢选中结点的父目录|
|`J` (大写)|跳到当前目录下同级的最后一个结点|
|`I` (大写)|切换是否显示隐藏文件|
|`m`|显示系统菜单|
|`C` (大写)|设置当前目录为root|
|`u`|返回到当前root的上级父目录并设置为root|
|`q`|关闭NerdTree窗口|

## taglist

taglist是一个用于显示定位程序中各种符号的插件，例如宏定义、变量名、结构名、函数名这些东西我们将其称之为符号(symbols)，而在taglist中将其称之为tag。显然，要想将程序文件中的tag显示出来，需要事先了解全部tag的信息，并将其保存在一个文件中，然后去解析对应的tag文件。taglist做的仅仅是将tag文件中的内容解析完后显示在Vim上而已。tag扫描以及数据文件的生成则是由[ctags(Exuberant Ctags)](http://ctags.sourceforge.net/)这一工具完成的，所以在使用taglist之前，你的电脑需要装有ctags(Vim默认已安装)。

taglist的设置：
```
" taglist设置
let Tlist_Show_One_File=1 " 0为同时显示多个文件函数列表,1则只显示当前文件函数列表
let Tlist_Auto_Update=1
let Tlist_File_Fold_Auto_Close=1 " 非当前文件，函数列表折叠隐藏
let Tlist_Exit_OnlyWindow=1 "如果taglist是最后一个窗口，则退出vim 
let Tlist_Auto_Update=1            "Automatically update the taglist to include newly edited files.
"把taglist窗口放在屏幕的右侧，缺省在左侧
"let Tlist_Use_Right_Window=1 
"显示taglist菜单
"let Tlist_Show_Menu=1
"启动vim自动打开taglist
"let Tlist_Auto_Open=1
" ctags, 指定tags文件的位置,让vim自动在当前或者上层文件夹中寻找tags文件
set tags=tags
" 添加系统调用路径
set tags+=/usr/include/tags
"键绑定，刷新tags
nmap tg :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q *<CR>:set tags+=./tags<CR>
```

我们已经稍微解释了下taglist和ctags的关系，在真正使用过程中，需要在项目文件夹下面运行命令 `ctags -R --c++-kinds=+p --fields=+iaS --extra=+q *<CR>:set tags+=./tags<CR>` ，这个命令的主要任务是对当前目录下的全部文件递归扫描，然后生成一个“tags”文件，里面记录整个项目的全部符号信息，同时将当前目录下的这个tags文件告知taglist，taglist根据这个tags文件显示项目的tag。(对于ctags命令中的各个开关，可以通过man ctags查看具体作用，此处不再赘述)

接下来，稍微解释下部分配置内容：
`set tags=tags` 表示tags文件所在位置，此处设定为当前目录下查找tags文件，当然，你也可以添加其它路径下的tags文件。`set tags+=/usr/include/tags` 就将系统调用的tags文件也包含进来了，这样如果程序中包含系统的一些调用，那么也能被正确解析。此外，我们还将生成tags的ctags命令进行了键盘映射：`nmap tg :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q *<CR>:set tags+=./tags<CR>`  ，这样就可以直接在Vim的Normal模式下输入`tg`直接在当前工作目录下产生tags文件而不用先在终端运行ctags命令后在到Vim显示了。同时还需要注意的一点是，一旦程序代码发生改动，则相应的“tag”也会发生变化，这时需要重新利用ctags命令生成tags文件，这个对于我们设置完键位映射后显得比较简单，再输入`tg`即可。

taglist的基本操作：

|key|Description|
|:-|:-|
|`ctrl+]`|查找当前光标所在处tag的定义|
|`ctrl+t`|返回到之前执行查找tag时的位置|
|`o`|在一个新打开的窗口中显示光标下tag|
|`Space`|显示光标下tag的原型定义|
|`x`|taglist窗口放大和缩小，方便查看较长的tag|
|`q`|关闭taglist窗口|

关于taglists的具体配置以及使用，请在Vim下输入`:help taglist `查看。同时，这里推荐一个关于ctags介绍的[视频](http://happycasts.net/episodes/25?autoplay=true)。

**cscope**

说到ctags，就不得不提下cscope。ctags最大的优点在于通过 `ctrl+]`就可以快速定位到程序符号所在的位置进行查看，之后再通过 `ctrl+t` 或者 `ctrl+o` 返回之前的位置，用两组按键来总结ctags，那就是 `ctrl+]` 和 `ctrl+t` 。ctags虽然已经相当不错了，但是仍然无法解决一些问题，例如查看函数被哪些函数调用，函数调用了哪些函数，查找该头文件被哪些文件包含等等，而解决这些问题，就需要用到cscope了。cscope不是Vim插件，而是一个工具，因此需要在终端进行安装。cscope的原理其实和ctags一样，也需要事先扫描项目目录，生成相关信息文件，方便之后查找的时候使用，cscope生成的数据文件为 `cscope.out` 和 `cscope.in.out`。

cscope设置：

```
" Cscope 设置
if has("cscope")
    set csprg=/usr/bin/cscope   "指定用来执行cscope的命令
    set csto=0                  " 设置cstag命令查找次序：0先找cscope数据库再找标签文件；1先找标签文件再找cscope数据库"
    set cst                     " 同时搜索cscope数据库和标签文件"
    set cscopequickfix=s-,c-,d-,i-,t-,e-    " 使用QuickFix窗口来显示cscope查找结果"
    set nocsverb
    if filereadable("cscope.out")    " 若当前目录下存在cscope数据库，添加该数据库到vim
        cs add cscope.out
    elseif $CSCOPE_DB != ""            " 否则只要环境变量CSCOPE_DB不为空，则添加其指定的数据库到vim
        cs add $CSCOPE_DB
    endif
    set csverb
endif
map <F4>:!cscope -Rbq<CR>:cs add ./cscope.out .<CR><CR><CR> :cs reset<CR>
"对:cs find c等Cscope查找命令进行映射
nmap <leader>s :cs find s <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <leader>g :cs find g <C-R>=expand("<cword>")<CR><CR>
nmap <leader>d :cs find d <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>
nmap <leader>c :cs find c <C-R>=expand("<cword>")<CR><CR>:copen<CR><CR>
nmap <leader>t :cs find t <C-R>=expand("<cword>")<CR><CR>:copen<CR><CR>
nmap <leader>e :cs find e <C-R>=expand("<cword>")<CR><CR>:copen<CR><CR>
nmap <leader>f :cs find f <C-R>=expand("<cfile>")<CR><CR>
nmap <leader>i :cs find i <C-R>=expand("<cfile>")<CR><CR> :copen<CR><CR>
" 设定是否使用 quickfix 窗口来显示 cscope 结果
set cscopequickfix=s-,c-,d-,i-,t-,e-
```

cscope生成数据文件的命令为：`cscope -Rbq` ，`-R` 表示在生成索引文件时，搜索子目录树中的代码；`-b` 表示只生成索引文件，不进入cscope的界面；`-q` 表示生成 `cscope.in.out` 和 `cscope.po.out` 文件，加快cscope的索引速度。我们的配置文件将`<F4>`键映射成了cscope命令，同时将生成的cscope.out文件加入到当前索引文件中。(由于cscope和ctags往往配合一同使用，而且都需要事先生成数据文件，所以本人直接将ctags和cscope生成数据文件的操作组合起来，绑定到了一组组合键上：`nmap tg :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q *<CR> :set tags+=./tags<CR>:!cscope -Rbq<CR>:cs add ./cscope.out .<CR>` ，键位绑定这东西，仁者见仁，智者见智，根据自己的喜好来。)

cscope基本操作：

|key|Description|
|:-|:-|
|`:cs find s`|查找这个C符号|
|`:cs find g`|查找这个定义|
|`:cs find d`|查找被这个函数调用的函数（们）|
|`:cs find c`|查找调用这个函数的函数（们）|
|`:cs find t`|查找这个字符串|
|`:cs find e`|查找这个egrep匹配模式|
|`:cs find f`|查找这个文件|
|`:cs find i`|查找#include这个文件的文件（们）|

find可以简写为f, 所以命令可以写成`:cs f s symbol_name`

参考上面cscope的配置语句，我们将 `:cs find` 操作映射成了 `<leader>` ，这样方便很多。`nmap <leader>s :cs find s <C-R>=expand("<cword>")<CR><CR> :copen<CR><CR>` ，这里我们主要关注下这条语句后面的内容，其中 `<C-R>=expand("<cword>")` 代表当前光标所在地方的单词，而 `:copen` 则是打开Vim中内置的一个称之为 `quickfix` 的窗口用于显示查找的结果。

对我而言，我看代码时最常用的就是 `ctrl+]` ，`ctrl+o` 和 `:cs find c` 了。

关于cscope的详细信息，请在终端运行 `man cscope` 。

## winmanager

我们上面介绍了用于文件浏览的nerdtree以及浏览程序符号的taglist，这两个插件都会以窗口的形式出现在Vim的窗口中，那么如何合理的安排它们，这就是winmanager的作用。

这里，我们利用winmanager将nerdtree和taglist放到同一个"侧边栏"中，nerdtree在上方，taglist在下方，效果如下图所示：

![](/img/vim2/winmanager_with_nerdtree_and_taglist.png)

winmanager设置：
```
" WinManager设置
" NERD_Tree集成到WinManager
let g:NERDTree_title="[NERDTree]" 
function! NERDTree_Start()
    exec 'NERDTree'
endfunction

function! NERDTree_IsValid()
    return 1
endfunction

" 键盘映射，同时加入防止因winmanager和nerdtree冲突而导致空白页的语句
nmap wm :if IsWinManagerVisible() <BAR> WMToggle<CR> <BAR> else <BAR> WMToggle<CR>:q<CR> endif <CR><CR>
" 设置winmanager的宽度，默认为25
let g:winManagerWidth=30 
" 窗口布局
let g:winManagerWindowLayout='NERDTree|TagList'
" 如果所有编辑文件都关闭了，退出vim
let g:persistentBehaviour=0
```
这里唯一要说的就是我将wm映射成了winmanager的开关，这意味着，在Vim的Normal模式下，我输入`wm`即可打开“侧边栏”，再次输入`wm`即可关闭“侧边栏”。

## vim-airline

接下来介绍一下个人非常喜欢的一个插件vim-airline。这是一款状态栏增强插件，可以让你的Vim状态栏非常的美观，同时包括了buffer显示条扩展smart tab line以及集成了一些插件。vim-airline-demo

![airline展示](/img/vim2/vim-airline-demo.gif)

(关于状态栏增强插件，你可能还听说过Powerline，airline是受Powerline的启发而开发的。和airline不同的是，Powerline不是纯粹的Vim脚本编写的，需要python的支持，同时airline的代码远少于powerline，这意味着airline更轻更快。)

### vim-airline字体补丁

虽然我们在本文的上半部分中讲解了通过Vundle来安装插件，但是airline有点特殊，还需要进行些额外的操作才能达到最佳的显示效果。这是因为airline中的三角形箭头实际上是通过特殊的字符而非图片实现的，但是系统原先的字体库中并没有包含这类特殊的图形字符，因此，想要达到上图所示的显示效果，需要对系统字库“打补丁”，即添加特殊图形字符。具体步骤如下：

1. 下载已经打过补丁的字体

在家目录下新建一个.fonts文件夹：`$ mkdir ~/.fonts` 然后 `$ cd ~/.fonts` 进入文件夹，执行 `$git clone git@github.com:Lokaltog/powerline-fonts.git` 下载已经打过补丁的字体文件。此时，`~/.fonts/` 目录下已经存在打过补丁的常用系统字体文件了。

2. 向系统中安装打过补丁的字体

运行 `$ fc-cache -vf ~/.fonts` 安装patched fonts到系统中。

3. 设置终端字体为打过补丁的字体

![](/img/vim2/set_patched_font_for_terminal.png)

我这里将终端字体设置成了DejaVu Sans Mono for Powerline。(打过补丁的字体名字后面都会有个for Powerline，选择自己喜欢的就好)。

### vim-airline设置
```
" airline设置
set laststatus=2
" 使用powerline打过补丁的字体
let g:airline_powerline_fonts = 1
" 开启tabline
let g:airline#extensions#tabline#enabled = 1
" tabline中当前buffer两端的分隔字符
let g:airline#extensions#tabline#left_sep = ' '
" tabline中未激活buffer两端的分隔字符
let g:airline#extensions#tabline#left_alt_sep = '|'
" tabline中buffer显示编号
let g:airline#extensions#tabline#buffer_nr_show = 1
" 映射切换buffer的键位
nnoremap [b :bp<CR>
nnoremap ]b :bn<CR>
```
关于上述的配置文件没什么好说的，注释都解释的比较明白了。这里我只想提一下vim-airline中自带的smart tab line扩展，smart tab line效果如下

![airline中的tabline展示](/img/vim2/tabline_in_airline.gif)

我们都知道在Vim中可以同时操作多个文件，多文件操作主要有两个显示手段，一个是buffer方式，一个tab的方式。tab方式我想不用多说，就相当于标签页，每个标签页位于不同的Vim窗口。因此，当切换tab的时候，原先的窗口布局会发生混乱，这个是我们不愿意看到的，而buffer则是在同一个窗口下的，便不存在这个问题。实际使用过程中，我更倾向于使用buffer来管理打开的多个文件。但是Vim对于buffer，默认的是隐藏的，也就是说，你打开了A、B、C三个文件，Vim并不将这三个文件的buffer显示出来，你只能通过 `:ls` 来查看buffer，然后通过 `:bn` (buffer next)和 `:bp` (buffer previous)，或者 `:b num` (打开编号为num的buffer)这样的命令来切换不同文件。如果能把buffer显示在Vim的窗口上，那么buffer切换起来就相对比较轻松，同时也能明白自己打开的文件都有哪些。

对于buffer可视化以及buffer管理的插件，比较著名的有 `Minibufexplorer` 和 `Bufexplorer`，大多数人选择使用Minibufexplorer(Bufexplorer我没用过)。Minibufexplorer整体上来说不错，但是存在一个个人认为非常致命的缺陷，那就是和winmanager的兼容性不好，表现在当打开多个buffer的时候，winmanager中的nerdtree显示部分会被压扁，无法还原，同时关闭文件buffer时，窗体的布局有时候会发生混乱。正因为这样的原因导致我最后放弃了Minibufexplorer。而airline中自带的smart tab line的主要功能是将buffer显示出来，同时当文件名太长的时候，会根据设定的模式进行适当的缩写。可能没有Minibufexplorer功能多，但是这样对我而言就已经足够了，更何况tab line还如此的美观呢 :-D

因为tab line只是一个显示buffer的扩展，并不提供操作buffer的功能，所以想要操作buffer还是要通过Vim自带的命令实现，从上面的配置语句中可以看到，我将切换到前一个buffer的命令 `:bp` 映射成了 `[b` ，将切换至后一个buffer的命令 `:bn` 映射成了 `]b` (这实际上是airline作者的映射)

 ## vim-Bbye

vim-Bbye主要是用于解决在关闭buffer时候导致的窗口混乱问题。(如果你在关闭buffer的时候没有任何问题的话，可以无视此插件= =)问题主要表现为，在Vim中，如果当前窗口里存在多个buffer，而你用 `:bd` 命令删除当前操作的buffer的话，会导致整个Vim窗口都被关闭的问题。vim-Bbye很好的解决了这个问题。

vim-Bbye涉及到的主要命令就是 `:Bd`,采用这个命令关闭当前buffer不会导致整个窗口被关闭的问题。因此我将这条命令映射成了bd ，在vimrc中的配置如下：

```
" Bbye设置
" 由于原生的:bd在删除当前buffer时会将整个窗口关闭，故使用Bbye的:Bd
nnoremap bd :Bd<CR>
```

 ## ctrlp

ctrlp是一款相当棒的插件，主要的功能是对文件进行模糊的查找，如果你的project目录结构复杂，或者你正在阅读一个较大的项目的话，那么ctrlp可以帮你快速的定位到你想要文件而不必在终端不断的cd、ls。

例如想打开文件 `linux/arch/x86/mm/mmap.c` , 只需要在vim中通过 `ctrl+p` , 打开ctrlp, 输入 `laxmm` 或者 `lucxmp` (诸如此类的, 即路径中顺序出现的字符组合), ctrlp就能帮你定位到对应的 `mmap.c` 文件, 回车打开即可.

ctrlp在阅读大项目时特别有用, 很多时候,  我们只记得要找的文件大概在某个文件夹下, 不确定中间还有哪些文件夹, 这时ctrlp就可以极大程度的提高你定位文件的效率.

这里提供一个介绍ctrlp不错的[视频](http://happycasts.net/episodes/64?autoplay=true)给大家。

## nerdcommenter

如果你是一个酷爱写注释的程序员的话，那么你一定要用一下nerdcommenter，即便你不热衷于写注释，你也应该关注下nerdcommenter这款插件。

nerdcommenter基本操作：

|key|Description|
|:-|:-|
|`<leader>cc`|注释当前行|
|`<leader>c+space`|注释选定代码行|
|`<leader>cu`|取消注释|
|`<leader>ca`|在可选的注释方式之间切换，比如C/C++ 的块注释/* */和行注释//|
|`<leader>cA`|在当前行尾添加注释符，并进入Insert模式|

nerdcommenter和Vim的Visual模式结合可以快速的注释/取消注释多行代码，同时在行尾追加注释并自动进入Insert模式可以方便的进行行内注释。

关于nerdcommenter的更多信息，请在Vim中执行 `:help nerdcommenter` 。

## a.vim

a.vim的主要功能是快速的在`*.c` / `*.cpp` 和 `*.h`之间进行转换，如果你正在用C/C++进行开发或者正在阅读C/C++项目的话，那么你可能会需要用到a.vim的。

a.vim的基本操作：

|key|Description|
|:-|:-|
|`:A`|头文件／源文件切换|
|`:AN`|在多个匹配文件间循环切换|
|`<Leader>ih`|切换至光标所在文件|
|`<Leader>is`|切换至光标所在处(单词所指)文件的配对文件(如光标所在处为foo.h，则切换至foo.c/foo.cpp...)|
|`<Leader>ihn`|在多个匹配文件间循环切换|

关于a.vim的更多信息请参考a.vim插件目录下的README文件。

## OmniCppComplete

老牌的C/C++自动补全插件了，目前似乎存在更好的自动补全插件，例如 `YouCompleteMe` 和 `Neocomplete`。这里就不再对OmniCppComplete进行叙述了。

---

# Vim及插件集的备份

经过了上面这么多道复杂的工序后，我们现在手里的Vim似乎更加完善了。在讲解的最后部分，让我们来聊聊如何对Vim的环境来进行备份。

目前我们对Vim的配置主要涉及vimrc文件和插件集，对于有多台电脑或者经常四处跑的同学来说，肯定能理解同步环境的意义，如果能将我们在本地配置好的Vim环境都保存在某个服务器上，那么不管在哪里，用哪台电脑，只要能联网，我们就可以通过网络迅速的恢复/搭建好我们的Vim环境。这里不得不称赞一下github，实在是广大程序员的福音，即便不是程序员，合理的使用github也能省去不少麻烦。利用git，我们可以在github服务器上建立一个项目，专门用来存放我们的Vim配置文件以及插件，然后将本地配置好的Vim环境全都发送至服务器端，等需要的时候，直接从服务器端取回即可，不需要再进行额外的配置，可以说是非常的方便。我们接下来讲讲具体的步骤。

## 建立Github帐号

如何申请github账号，这里就不说了。

## 新建Github项目

在你的github账号上新建一个项目

![新建github项目](/img/vim2/github_create_repo.png)

这里，不要勾选 `Initialize this repository with a README` 。点击 `create repository` 来创建一个属于你的项目。这一步的目的是在github服务器上建立用于存放Vim配置及插件的仓库，以便之后可以将本地上的文件提交到这个仓库中保存。

## 操作本地仓库

在本地 `～/.vim/` 目录进行git操作

`$ cd ~/.vim` 移动到 `~/.vim/` 文件夹下，我们的Vim插件都在这个文件夹下，因此我们要将这个目录作为git操作的根目录，之后才能将这个目录的内容提交到服务器上。

Vim环境除了插件外，还有 `～/.vimrc` 文件，因为我们备份的目录是 `~/.vim/` 并不包含 `～/.vimrc` 文件，因此我们 `$ mv ~/.vimrc ~/.vim/` 将 `.vimrc` 移动到 `~/.vim/` 目录下，同时 `ln -sf ～/.vim/.vimrc ~/.vimrc` 在家目录下创建一个符号链接。

接下来是本地的git操作：

对于第一次使用git的同学而言，你需要告知git工具你的用户名和邮箱信息，以便在你执行git操作的时候可以知道是谁作出的改动。
```
$ git config --global user.name "你的名字" 
$ git config --global user.email "你的邮箱"
```
在～/.vim/文件夹下初始化git项目信息。
```
$ git init
```

将～/.vim/文件夹的内容和隐藏文件.vimrc全都加入git的追踪列表里
```
$ git add * .vimrc 
```
(git add . 是常用的添加命令, 但是不包括隐藏文件, 这也是为什么单独添加.vimrc的原因)。

提交追踪列表里的文件，同时给出本次提交操作的相关信息。
```
$ git commit -m 'first commit for my vim evironment' 
```
这样我们 `～/.vim/` 文件夹下的全部内容就全都提交到本地的git仓库中了。接下来要做的就是将本地的git仓库发送(push)到远端的github服务器的对应项目上。

## 同步本地仓库至远端仓库

接下来是将本地仓库push到github上的操作：

push操作实际上是将已经提交给git的本地仓库发送到远端服务器上，这里我们将上面提交好的 `～/.vim/`目录提交到特定账号上的特定项目中。这个过程需要对你的github账户上的项目进行写操作，显然只有用户自身才有权限对其账号下的项目执行写操作，因此在提交之前，需要完成github对应用户的验证过程。

github采用SSH的验证方式，即用户需要在本地电脑上生成一对SSH密钥，将公钥保存在github上，自己本地电脑上保留私钥，在连接github时进行私钥和公钥的验证，匹配则被认定为用户登陆成功(关于SSH的连接原理，请参见[SSH连接认证原理概述]()。所以，第一次向github提交项目的同学，需要执行：

`$ ssh-keygen` 一路回车，这条命令会在 `~/.ssh/` 目录下产生私钥文件 `id_rsa` 以及公钥文件 `id_rsa.pub`，我们需要将公钥文件`id_rsa`中的内容填入到github账户下的SSH key中。

在github.com上点击 `账户设定 -- SSH-Keys -- add SSH key`
![](/img/vim2/add_sshkey_for_github.png)

可以看到我已经上传了一个公钥(Public Key)了，在这里，你需要给公钥一个名字，便于区别，同时在Title下方的Key文本框内，将 `~/.ssh/` 目录下的 `id_rsa.pub` 文件中的内容粘贴进来(复制粘贴的时候注意格式，公钥不包含回车换行的，同时不要在末尾留有空格)。点击Add key，就可以成功添加公钥了。这样，就完成了github账户和本地电脑的SSH密钥配对。

`$ ssh -T git@github.com` 在终端对github来进行连接，这里的用户名为git，是所有github用户统一的用户名。如果连接成功，终端会返回 `Hi 你的用户名! You've successfully authenticated, but GitHub does not provide shell access.` 。

`$ cd ~/.vim/` 再次回到 `~/.vim/` 目录，接下来将本地的git仓库推送到github的对应项目上：
```
$ git remote add origin git@github.com:你的用户名/你的项目名.git
$ git push origin master
```
现在，到github上看看，`.vim`文件夹下的内容是不是都出现在上面了: - )

自此，备份过程结束。如果想要恢复备份的话，只需要 `$ git clone https://github.com/你的用户名/你的vim项目.git` 将github上的vim项目拷贝至本地, 之后再 `mv 你的vim项目名 ~/.vim` 即可。如果你又给你的Vim做了修改或者添加了新的插件的话，那么只要重新执行上述过程中的git add、git commit、git push操作即可。

对于git的基本操作，可以参见这个[视频](http://happycasts.net/episodes/7?autoplay=true)。

---

# 总结

Vim真的是一款非常强大的文本编辑工具，搭配上适当的插件之后更是如虎添翼，但是过多的插件又会让Vim变得非常的臃肿，甚至反应速度会变慢，而且也违背Unix的KISS(keep it simple, stupid)原则。在网上也有部分人认为不应该使用Vim插件，避免对插件过渡依赖。就我个人而言，我需要的只是一个高效的工具，来帮我完成特定的任务，至于这个工具是复杂，是简单，是不是需要恪守一些原则我不是很在意，我真正关心的是这个工具能否高效的完成我的任务。正如白猫黑猫的道理一样。因此，对于Vim，我一直本着让Vim来适应我的习惯和需求而非让我去适应Vim的想法来使用它的，如果不添加任何插件，那我想在很多时候Vim就不是那么的好用，如果添加了许多的插件，那么Vim的响应速度会很慢而且其中大部分插件只是会偶尔的用到或根本不用。这是一个trade-off的过程，总之自己用的顺手，用的舒坦才是最重要的。

这篇文章亦是我心血来潮之作，介绍Vim及其插件的文章可谓成千上万，而且Vim本身以及绝大部分的插件都正在不断的发展和完善中，可能现在流行的插件，过不了多久就被另外一个新兴的插件给替代了，同样的，现在写的关于Vim插件的相关文章也会因为时间的流逝而慢慢过时而变得没有意义。即便如此，我依然还是很开心的把这一大篇的文章给写完了，为何如此？只是最近使用Vim开代码，添加新的插件的时候，回想起自己当初刚接触Vim时茫然无措的情景，而上网找到的资料要不是早就过时的，要不就是存在错误或者轻描淡写草草带过的，只有极少数负责任的博主的几篇优秀的博文真正帮到了我。而我所能做的，也只是将我目前所知道，尽可能认真的阐述出来，或许能帮到一些人。学习、记录、分享，不亦乐乎。