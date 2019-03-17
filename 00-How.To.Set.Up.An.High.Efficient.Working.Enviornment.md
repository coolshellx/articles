程序员是一个很懒的群体，总想着能够让代码为自己干活，他们不断地把工作生活中的一些事情用代码自动化了，从而让整个社会的效率运作地越来越高。所以，程序员在准备去优化这个世界的时候，都会先要优化自己的工作环境，是所谓“工欲善其事，必先利其器”。

我们每个程序员都应该打造一套让自己更为高效的工作环境。那怕就是让你少输入一次命令，少按一次键，少在鼠标和键盘间切换一次，都会让程序员的工作变得更为的高效。所以，程序员一般需要一台性能比较好，不会因为开了太多的网页或程序就卡得不行的电脑，还要配备多个显示器，一个显示器写代码，一个查文档，一个测试运行结果，而不必在各种窗口来来回回的切换……在大量的窗口间切换经常会迷路，而且也容易出错（分不清线上或测试环境）……

除了硬件上的装备，软件上也是能够得升程序员生产力的地方，下面我们就想在软件层面说一下可以提升程序员生产力的一些东西。

这篇文章在查阅了很多资料以及结合自身的实践，从四个方面来说明如何打造一套适合自己的工具，分别是**编辑器篇**、**命令行篇**、**Shell和脚本篇**、**版本管理篇**。



### 编辑器篇

对于程序员来说，日常与编辑器打交道的时间非常长。编辑器是值得我们投入大量时间去学习并且定制来满足我们的需要。

如何选择合适的编辑器呢？编辑器有很多，有的值得投资时间去学习，有的基本就不用花时间去学。应当去学习那些能够提升生产力，有很强的生命力，并且能够适应各类环境的编辑器。

**Vim** 和 **Emacs** 就是非常值得投资的编辑器，如果你决定开始学习 Vim/Emacs，刚开始的时候会感觉高很痛苦，甚至会伤害到自己的生产力，但是没关系，通过两三周的专注练习后，就可以通过学习编辑器的死亡之谷，很快就可以赚回票价了。

使用 Vim/Emacs 的另一个重要好处是，当你在处理一些线上问题的时候，有时候需要一个 IDE，你发现在一个Term 终端上是运行不了图形界面的编辑器了，你只有 Vim/Emacs 这样的选择。在 Coolshell 上有一篇《无插件Vim 编程技巧》（https://coolshell.cn/articles/11312.html）你可以移步一看。

为什么不学习 Notepad，因为，**使用高效编辑器的终极目标是抛弃鼠标，让手编辑的速度可以跟的上大脑思考的速度**。就我自己来说，从大二开始接触 Emacs 开始，已经使用 5 年了，通过不断的对 Emacs 进行定制，已经有了一套自己的使用模式了，每次打开 Emacs 就像是打开了自己才知道的秘籍，这是其他 IDE 都没有办法比的，通过不断定制自己的工具，可以激发自己创造的欲望。

但是我不参加编辑器的圣战，这些都是无意义的口水战。在一次接触了 Vim 之后，我反而喜欢上 Vim 的编辑模式了，于是在 Emacs 中使用上了 Vim 的编辑模式，实际的情况是这两者并不冲突（皓哥翻译的那篇 《简明 Vim 练级攻略》（https://coolshell.cn/articles/5426.html）文章非常不错）。

在 Emacs 上，有很多有用的特性和插件。 Emacs 的 **org-mode**，这个工具用来写作，还可用来做任务管理，任务计时器，甚至可以用来做表格的计算。功能上已经完全不是 markdown 能比的，但是遗憾的是目前只能在 Emacs 中使用（貌似有人在 Vim 上实现过）。

还有就是 Emacs 中的用来做文件管理的插件 **helm** 和 **ivy**，这两个插件完成的功能差不多，但是 helm 比较重量级一些，如果一个项目的代码量比较大时，文件的切换和搜索就没那么快了，所以我更喜欢轻量级的 ivy。

还有两个我最喜欢的插件，**evil** 和 **magit**，evil 可以让 Emacs 变身成为一个 Vim，大多数的 Vim 操作都可以覆盖上。magit 是一个可以使 Emacs 变身成为 Git 的插件。当然其他的各种语言的支持插件就是常规操作了，不多说。 

下面推荐一些 Emacs 和 Vim 的学习资源:

**Emacs**

- **一年成为 Emacs 高手**，当年我也是靠着这一篇文章真正入门 emacs,

  https://github.com/redguardtoo/mastering-emacs-in-one-year-guide/blob/master/guide-zh.org

- 陈斌 (一年成为 Emacs 高手的作者)的 Emacs 配置，是一个很不错的配置，我现在使用的配置要就是从这个演化来的 https://github.com/redguardtoo/emacs.d
- Steve Purcell 大牛的配置，很适合 Web 开发者：https://github.com/purcell/emacs.d
- Spacemacs 配置，适合新手，有着非常完善的文档：https://github.com/syl20bnr/spacemacs
- reddit 的 Emacs 频道，最前沿的 Emacs 技巧：https://www.reddit.com/r/emacs/
- Emacs org-mode 的文档，从这个文档中你可以发现纯文本的魔力：https://orgmode.org/ 
- Emacs Wiki，上面有大量的 Emacs 插件的实践，虽然质量参差不齐，但是也有很多优秀的插件 https://www.emacswiki.org/emacs/EmacsWiki

**Vim**

- Doist 创业公司 CEO amix 的 Vim 配置，被称之为最强 vimrc：https://github.com/amix
- junegunn 是韩国的一个大牛，擅长写 Vim 插件，他的 Vim 插件看起来总是令人赏心悦目：https://github.com/junegunn/vim-plug
- SpaceVim，一个开箱即用的 Vim 配置，对新手很友好，和 Spacemacs 一样，有着很完善的文档：https://github.com/SpaceVim/SpaceVim
- reddit 的 Vim 频道，有很多最前沿的 Vim 技巧:https://www.reddit.com/r/vim/ 



### 命令行篇

一个好的、可定制的命令行环境可以给工作效率带来很大的提升。如果你还在使用这个，那就差点意思了：

![](http://rayjun.oss-cn-beijing.aliyuncs.com/shell/Image.png)

在不同的操作系统下，都有着很不错的命令行工具，比如 Mac 下的 **Iterm2**，Linux 下的原生命令行，如果你是在 Windows 下工作，问题也不大，因为 Windows 下现在有了 **WSL**。WSL 提供了一个由微软开发的Linux兼容的内核接口（不包含Linux内核代码），然后可以在其上运行GNU用户空间，例如 Ubuntu，openSUSE，SUSE Linux Enterprise Server，Debian和Kali Linux。这样的用户空间可能包含 Bash shell 和命令语言，使用本机 GNU/Linux 命令行工具（sed，awk 等），编程语言解释器（Ruby，Python 等），甚至是图形应用程序（使用主机端的X窗口系统）。

使用命令行可以完成所有日常的操作，新建文件夹（mkdir）、新建文件（touch）、移动（mv）、复制（cp）、删除（rm）等等。而且使用 Linux/Unix 命令行最好的方式是可以用 awk、sed、grep、xargs、find、sort 等等这样的命令，然后用管道把其串起来，就可以完成一个你想要的功能，尤其是一些简单的数据统计功能。这是Linux命令行不可比拟的优势。比如：

- 查看连接你服务器 top10 用户端的 IP 地址：

`netstat -nat | awk  '{print  $5}' | awk -F ':' '{print $1}' | sort | uniq -c | sort -rn | head -n 10`

- 查看一下你最常用的10个命令：

`cat .bash_history | sort | uniq -c | sort -rn | head -n 10 (or cat .zhistory | sort | uniq -c | sort -rn | head -n 10`

在命令行中使用 **alias**  可以将使用频率很高命令或者比较复杂的命令合并成一个命令，或者修改原生的命令。

下面这几个命令，可能是你天天都在敲的。所以，你应该设置成 alias 来提高效率

```
alias nis="npm install --save "
alias svim='sudo vim'
alias mkcd='foo(){ mkdir -p "$1"; cd "$1" }; foo '
alias install='sudo apt get install'
alias update='sudo apt-get update && sudo apt-get upgrade'
alias ..="cd .."
alias ...="cd ..; cd .."
alias www='python -m SimpleHTTPServer 8000'
alias sock5='ssh -D 8080 -q -C -N -f user@your.server' 
```
你还可以参考如下的一些文章，看看别人是怎么用好 `alias` 的

- https://www.cyberciti.biz/tips/bash-aliases-mac-centos-linux-unix.html
- https://www.digitalocean.com/community/questions/what-are-your-favorite-bash-aliases
- https://www.linuxtrainingacademy.com/23-handy-bash-shell-aliases-for-unix-linux-and-mac-os-x/
- https://brettterpstra.com/2013/03/31/a-few-more-of-my-favorite-shell-aliases/


命令行中除了原生的命令之外，还有很多可以提升使用体验的工具。下面罗列一些很不错的命令，把原生的命令增强地很厉害:

- **fasd** 增强了 `cd` 命令 (https://github.com/clvv/fasd)。

- **bat** 增强了 `cat` 命令 (https://github.com/sharkdp/bat)。如果你想要有语法高亮的 `cat`，可以试试 **ccat**(https://github.com/jingweno/ccat)

- **exa** 增强了 `ls` 命令(https://github.com/ogham/exa），如果你需要在很多目录上浏览各种文件 ，**ranger** 命令可以比 `cd` 和 `cat` 更有效率（https://github.com/ranger/ranger)，甚至可以在你的终端预览图片。

- **fd** 是一个比 `find` 更简单更快的命令(https://github.com/sharkdp/fd)，他还会自动地忽略掉一些你配置在 `.gitignore` 中的文件，以及 `.git` 下的文件。


- **fzf** 是另外一个很好用的文件搜索神器 (https://github.com/junegunn/fzf)，其主要是搜索当前目录以下的文件，还可以使用 `fzf --preview 'cat {}'` 边搜索文件边浏览内容。

- `grep` 是一个上古神器，然而，**ack** (https://beyondgrep.com/)、**ag** (https://github.com/ggreer/the_silver_searcher) 和 **rg** (https://github.com/BurntSushi/ripgrep) 是更好的grep，和上面的fd一样，在递归目录匹配的时候，会忽略到你配置在 .gitignore 中的规则。

- `rm` 是一个危险的命令，尤其是各种 `rm -rf …`，所以，**trash**（https://github.com/andreafrancia/trash-cli/) 是一个更好的删除命令。

- `man` 命令是好读文档的命令，但是man的文档有时候太长了，所以，你可以试式 **tldr**（https://github.com/tldr-pages/tldr) 命令，把文档上的一些示例整出来给你看。

- 如果你想要一个图示化的`ping`，你可以试试 **prettyping** (https://github.com/denilsonsa/prettyping) 。

- 如果你想搜索以前打过的命令，不要再用 Ctrl +R 了，你可以使用加强版的 **hstr** (https://github.com/dvorka/hstr) 。

- **htop** (https://hisham.hm/htop/） 是 top 的一个加强版。然而，还有很多的各式各样的top，比如：用于看IO负载的 **iotop** (http://guichaz.free.fr/iotop/)，网络负载的 **iftop** (http://www.ex-parrot.com/~pdw/iftop/), 以及把这些top都集成在一起的 **atop** (https://github.com/Atoptool/atop)。

- **ncdu** (https://dev.yorhel.nl/ncdu) 比 du 好用多了用。另一个选择是 nnn（https://github.com/jarun/nnn) 。

- 如果你想把你的命令行操作建录制成一个 SVG 动图，那么你可以尝试使用 **asciinema** (https://asciinema.org/) 和 **svg-trem** (https://github.com/marionebl/svg-term-cli) 。

- **httpie**(https://github.com/jakubroztocil/httpie) 是一个可以用来替代 curl 和 wget 的 http 客户端，httpie 支持 json 和语法高亮，可以使用简单的语法进行 http 访问: `http -v github.com`。

- **tmux** (https://github.com/tmux/tmux) 在需要经常登录远程服务器工作的时候会很有用，可以保持远程登录的会话，还可以在一个窗口中查看多个 shell 的状态。

- **Taskbook**(https://github.com/klaussinani/taskbook) 是可以完全在命令行中使用的任务管理器 ，支持 ToDo 管理，还可以为每个任务加上优先级。

- **sshrc** (sshrc：https://github.com/Russell91/sshrc ) 是个神器，在你登录远程服务器的时候也能使用本机的 shell 的 rc 文件中的配置。

- **goaccess** (https://github.com/allinurl/goaccess) 这个是一个轻量级的分析统计日志文件的工具，主要是分析各种各样的 access log。



关于这些增加命令，参考自下面的这些文章

1. https://dev.to/_darrenburns/10-tools-to-power-up-your-command-line-4id4
2. https://dev.to/_darrenburns/tools-to-power-up-your-command-line-part-2-2737
3. https://dev.to/_darrenburns/power-up-your-command-line-part-3-4o53 
4. https://darrenburns.net/posts/tools/
5. https://hacker-tools.github.io/



### Shell 和脚本篇

shell 是可以与计算机进行高效交互的文本接口。shell 提供了一套交互式的编程语言（脚本），shell的种类很多，比如 **sh**、**bash**、**zsh** 等。

shell 的生命力很强，在各种高级编程语言大行其道的今天，很多的任务依然离不开 shell。比如可以使用 shell 来执行一些编译任务，或者做一些批处理任务，初始化数据、打包程序等等。

写一个脚本很简单，**touch zsh-script.sh**：

```shell
#!/bin/zsh
echo Hello shell
```

一个脚本就写完了，**#!/bin/zsh** 表示使用的是哪种 shell。写完之后需要给脚本加上执行的权限:

```shell
# 给脚本执行的权限
$ chmod +x zsh-script.sh
# 执行脚本
$ ./zsh-script.sh
```

脚本的语法很简单，而且可以在脚本中使用命令行的所有命令。还有很多其他有意思的玩法，比如在后台运行：

```shell
$ ./zsh-script.sh &
```

还有可以定时执行，在 **crontab** 加入：

```shell
* * * * * /home/ray/zsh-script.zsh
```

crontab 是 Linux 中的一个定时器，可以定制执行任务，上面的表示每分钟执行一次脚本。

我最喜欢的就是 **zsh** + **oh-my-zsh** + **zsh-autosuggestions** 的组合，你也可以试试看。其中 zsh 和 oh-my-zsh 算是常规操作了，但是 zsh-autosuggestions 特别有用，可以超级快速的帮你补全你输入过的命令，让命令行的操作更加高效。 

你也许会说，用 Python 脚本或 PHP 来写脚本会比 Shell 更好更没有 bug，但我要申辩一下:

- 其一，如果你有一天要维护线上机器的时候，或是到了银行用户的系统（与外网完全隔离，而且服务器上没有安装 Python/PHP 或是他们的的高级库，那么，你只有 Shell 可以用了）。

- 其二，而且，如果要跟命令行交互很多的话，Shell 是不二之选，试想一下，如果你要去 100 台远程的机器上查access.log 日志中有没有某个错误，完成这个工作你是用 PHP/Python 写脚本快还是用 Shell 写脚本快呢？

要写好一个脚本并不容易，下面有一些小模板供你参考：

处理命令行参数的一个样例

```
while [ "$1" != "" ]; do
    case $1 in
        -s  )   shift	
		SERVER=$1 ;;  
        -d  )   shift
		DATE=$1 ;;
	--paramter|p ) shift
		PARAMETER=$1;;
        -h|help  )   usage # function call
                exit ;;
        * )     usage # All other parameters
                exit 1
    esac
    shift
done
```

命令行菜单的一个样例

```

#!/bin/bash
# Bash Menu Script Example

PS3='Please enter your choice: '
options=("Option 1" "Option 2" "Option 3" "Quit")
select opt in "${options[@]}"
do
    case $opt in
        "Option 1")
            echo "you chose choice 1"
            ;;
        "Option 2")
            echo "you chose choice 2"
            ;;
        "Option 3")
            echo "you chose choice $REPLY which is $opt"
            ;;
        "Quit")
            break
            ;;
        *) echo "invalid option $REPLY";;
    esac
done
```

颜色定义，你可以使用 `echo -e "${Blu}blue ${Red}red ${RCol}etc...."` 进行有颜色文本的输出

```
RCol='\e[0m'    # Text Reset

# Regular           Bold                Underline           High Intensity      BoldHigh Intens     Background          High Intensity Backgrounds
Bla='\e[0;30m';     BBla='\e[1;30m';    UBla='\e[4;30m';    IBla='\e[0;90m';    BIBla='\e[1;90m';   On_Bla='\e[40m';    On_IBla='\e[0;100m';
Red='\e[0;31m';     BRed='\e[1;31m';    URed='\e[4;31m';    IRed='\e[0;91m';    BIRed='\e[1;91m';   On_Red='\e[41m';    On_IRed='\e[0;101m';
Gre='\e[0;32m';     BGre='\e[1;32m';    UGre='\e[4;32m';    IGre='\e[0;92m';    BIGre='\e[1;92m';   On_Gre='\e[42m';    On_IGre='\e[0;102m';
Yel='\e[0;33m';     BYel='\e[1;33m';    UYel='\e[4;33m';    IYel='\e[0;93m';    BIYel='\e[1;93m';   On_Yel='\e[43m';    On_IYel='\e[0;103m';
Blu='\e[0;34m';     BBlu='\e[1;34m';    UBlu='\e[4;34m';    IBlu='\e[0;94m';    BIBlu='\e[1;94m';   On_Blu='\e[44m';    On_IBlu='\e[0;104m';
Pur='\e[0;35m';     BPur='\e[1;35m';    UPur='\e[4;35m';    IPur='\e[0;95m';    BIPur='\e[1;95m';   On_Pur='\e[45m';    On_IPur='\e[0;105m';
Cya='\e[0;36m';     BCya='\e[1;36m';    UCya='\e[4;36m';    ICya='\e[0;96m';    BICya='\e[1;96m';   On_Cya='\e[46m';    On_ICya='\e[0;106m';
Whi='\e[0;37m';     BWhi='\e[1;37m';    UWhi='\e[4;37m';    IWhi='\e[0;97m';    BIWhi='\e[1;97m';   On_Whi='\e[47m';    On_IWhi='\e[0;107m';
```

取当前运行脚本绝对路径的示例：（注：Linux下可以用 `dirname $(readlink -f $0)` ）

```
FILE="$0"
while [[ -h ${FILE} ]]; do
    FILE="`readlink "${FILE}"`"
done
pushd "`dirname "${FILE}"`" > /dev/null
DIR=`pwd -P`
popd > /dev/null
```

如何在远程服务器运行一个本地脚本

```
#无参数
ssh user@server 'bash -s' < local.script.sh

#有参数
ssh user@server ARG1="arg1" ARG2="arg2" 'bash -s' < local_script.sh
```

如何检查一个命令是否存在，用 `which` 吗？最好不要用，因为很多操作系统的 `which` 命令没有设置退出状态码，这样你不知道是否是有那个命令。所以，你应该使用下面的方式。

```
# POSIX 兼容:
command -v <the_command>


# bash 环境:
hash <the_command> 
type <the_command> 

# 示例：
gnudate() {
    if hash gdate 2>/dev/null; then
        gdate "$@"
    else
        date "$@"
    fi
}
```
然后，如果要写出健壮性更好的脚本，下面是一些相关的技巧：


- 使用 `-e` 参数，如：`set -e` 或是 `#!/bin/sh -e`，这样设置会让你的脚本出错就会停止运行，这样一来可以防止你的脚本在出错的情况下还在拼拿地干活停不下来。
- 使用 `-u` 参数，如： `set -eu`，这意味着，如果你代码中有变量没有定义，就会退出。
- 对一些变理，你可以使用默认值。如：`${FOO:-'default'}`
- 处理你代码的退出码。这样方便你的脚本跟别的命令行或脚本集成。
- 尽量不要使用 `;` 来执行多个命令，而是使用 `&&`，这样会在出错的时候停止运行后续的命令。
- 对于一些字符串变量，使用引号括起，避免其中有空格或是别的什么诡异字符。
- 如果你的脚有参数，你需要检查脚本运行是否带了你想要的参数，或是，你的脚本可以在没有参数的情况下安全的运行。
- 为你的脚本设置 `-h` 和 `--help` 来显示帮助信息。千万不要把这两个参数用做为的功能。
- 使用 `$()` 而不是 `` 来获得命令行的输出，主要原因是易读。
- 小心不同的平台，尤其是 MacOS 和 Linux 的跨平台。
- 对于 `rm -rf` 这样的高危操作，需要检查后面的变量名是否为空，比如：`rm -rf $MYDIDR/*` 如果 `$MYDIR`为空，结果是灾难性的。
- 考虑使用 "find/while" 而不是 “for/find”。如：`for F in $(find . -type f) ; do echo $F; done` 写成 `find . -type f | while read F ; do echo $F ; done` 不但可以容忍空格，而且还更快。
- 防御式编程，在正式执行命令前，把相关的东西都检查好，比如，文件目录有没有存在。



最后推荐一些 Shell 和脚本的参考资料。

各种有意思的命令拼装，一行命令走天涯:

- http://www.bashoneliners.com/ 
- http://www.shell-fu.org/
- http://www.commandlinefu.com/

下面是一些脚本集中营，你可以在里面淘到各种牛X的脚本：

- http://www.shelldorado.com/scripts/
- https://snippets.siftie.com/public/tag/bash/
- https://bash.cyberciti.biz/
- https://github.com/alexanderepstein/Bash-Snippets
- https://github.com/miguelgfierro/scripts
- https://github.com/epety/100-shell-script-examples
- https://github.com/ruanyf/simple-bash-scripts 


甚至写脚本都可以使用框架:

- 写bash脚本的框架  https://github.com/Bash-it/bash-it

Google的Shell脚本的代码规范：

- https://google.github.io/styleguide/shell.xml

最后，别忘了几个和shell有关的索引资源：

- https://github.com/alebcay/awesome-shell
- https://github.com/awesome-lists/awesome-bash
- https://terminalsare.sexy/


### 版本管理篇

版本管理的工具对我来说已经不仅仅不是管理代码的工具了。任何需要不断优化，不断修改的内容都需要进行版本管理。

版本管理的工具很多。现在还有好些人还不喜欢 Git 还在用 svn，那是因为他们并不知道 Git 的强大之处，这种脱机的版本管理可以让你在没有网的情况下提效代码变更，再加上 Git 切换 branch 快得不行，merge branch 时会把 branch 的改动情况一同 merge 了，这样可以让你看到整个历史。再有 stash，cherry-pick 等等这样的黑魔法加持，你的工作效率真的很爽的。如今最好用的应该就是 **Git** 了，在加上最近 **GitHub** 私有仓库的开放，让这一优势继续扩大。

Git 这么好用的原因来源于其底层数据结构的设计，非常的有意思，如果你接触过区块链，你会发现 Git 底层的数据结构与区块链的数据结构有异曲同工之处。

Git 除了可以完成通常的版本管理之外，它还拥有一些很神奇的技能：

- 帮你找 bug 的命令: **git bisect**，通过二分搜索的方式来帮助你定位到引入 bug 的 commit。
- 可以帮助你洞察一切的 **git blame** 可以给你文件的每行信息都进行注释，然后就可以看到关于该行修改的每一次 commit 的哈希标签、作者和提交日期。
- 可以帮你恢复一切的 **git reflog**，通常我们 **git reset** 命令都是慎用的，要不然就坏事了，但是 git relog 在你将变化提交之前，可以帮助你回到任何修改之前，包括 git reset。但是 reflog 只是保存在本地，而且不是永久保存，有一个可以配置的过期时间。

除此之外，还有一些 Git 的小技巧：

- 如果你当前修改的文档在你本地的master上，然后你想把这个修改迁移到一个branch上，这样方便你做PR，如果这些修改还没有被add，那么你可以使用 `git checkout -b new_branch_name` 来完成。
- 如果你commit后，还没有push，你想修改一下这前的commit，你可以使用 `git commit --amend`
- 使用 `git pull --rebase` 可以避免出现一个 merge buble。
- 如果你要删除远程分支，可以使用 `git push origin --delete <remote-branchname>`
- 使用 `git add -p` 可以让你挑选修改，这样你可以把你的一次大改动变成多个提交。
- `git checkout -p` 和 `git add -p` 类似，如果让你挑选你想checkout的修改。
- 你知道你可以使用时间在你的git命令行上吗？比如：`git diff HEAD@{'2 months ago'}` 或 `git diff HEAD@{yesterday}` 或 `git diff HEAD@{'2010-01-01 12:00:00'}` 

有时候，设置一下 Git 的别名也是可以让你更高效工作的一件事，比如：

使用如下的 `git lg` 别名

```
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --"
```

就可以把原来的 `git log` 的输出

![](../../Desktop/git.log_.01.png)

变成下面这个样子

![](../../Desktop/git.log_.02.png)

除此之外，Git 还有好些比较有用的别名，下面这几个收集库，可以让你更高效更赏心悦目地使用 Git

- https://github.com/momeni/gittify 
- https://github.com/GitAlias/gitalias
- https://gist.github.com/mwhite/6887990

Git 有多种用法，可以使用原生的 Git 命令行，原生的 Git 命令加上 shell 脚本的包装就可以做到很高效了。 Git 也有图形化的界面，比如 Git 安装包自带的 gitk 或者 TortoiseGit，因为比起命令行来太慢了。还有就是 magit，这是 Emacs 的一个插件，这个插件将所有 Git 的操作都融入到 Emacs 中了，只需要使用使用快捷键就能够完成 Git 的所有操作，但是同时又带有一点图形化的感觉，这么说有点苍白无力，看图: 

![](http://rayjun.oss-cn-beijing.aliyuncs.com/emacs/emacs-git.jpg)

我现在日常会使用 Git 来进行代码管理、博客文章的管理和自己的知识库的管理。这些内容使用版本管理起来的好处是可以看见自己的成长过程，每一次修改的内容，每一个想法的进化。

下面推荐一下 Git 的学习资料:

- Progit2，最好的深入学习 Git 的教材，而且是开源的https://github.com/progit/progit2
- Magit，Git 在 Emacs 上的打开方式：https://magit.vc/
- Vim-fugitive，Git 在 Vim 上的打开方式：https://github.com/tpope/vim-fugitive
- Git 相关的 shell 提示： https://github.com/magicmonty/bash-git-prompt

Happy hacking！

(完)

