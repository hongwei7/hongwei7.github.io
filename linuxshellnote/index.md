# Linux Shell 笔记


此为做操作系统课程作业时,对shell语言的一些记录.<!-- more-->

## 常用命令

- `ls` 列出文件
  - `-F`  区分文件类型
  - `-a` 显示隐藏文件
  - `-R` 显示文件夹子文件夹
  - `-l` 显示更多信息，按行排列
  - 过滤器：通配符筛选文件
- `touch` 创建文件
  - `-a` 改变修改时间
- `cp` 复制文件 `mv` 移动文件（重命名）
  - `cp` ` source`  `destination`
  - `-i` 提醒是否覆盖
  - `-R` 复制并创建目录
- `ln` 链接文件
  - 符号链接 `ln -s data_file s1_data_file` 是两个独立的文件
  - 硬链接 `ln datafile linkfile` 两个相同的文件
  - 修改一个文件的时候，其硬链接和符号链接都会被修改
- `rm` 删除文件
  - `-i` 提示是否要删除
  - `-f` force删除
  - `-r` 递归删除子文件
- `mkdir` 创建目录
  - `-p` 同时创建多个目录和子目录
- `rmdir` 删除目录 只删除空目录
- `tree` 显示目录树
- `file` 查看文件类型
- `cat` 显示文本数据
  - `-n` 加上行号
  - `-b` 给有文字的加上行号
  - `-T` 不让制表符出现
- `more` 显示长文本
- `less` 更多功能的显示长文本工具
- `tail` 查看文件尾部 head 查看文件头部
- `ps` 查看进程
  - `-A` `-e` 显示所有进程
  - `-a` 显示除控制进程以外的和无终端进程外的所有进程
  - `-d` 显示除控制进程以外的所有进程
  - `-L` 显示进程中的线程
  - `-f` 完整输出
  - `--forest` 树状显示父子关系(linux)
- `top` 实时显示运行情况 检测资源使用
- `kill` 结束进程
  - `-s` 指定信号 `kill -s HUP 3940`
  - `killall` 通过进程名结束进程
- ![image-20200312193753491](/images/LinuxShell/image-20200312193753491.png)
- 挂载存储媒体 `mount`
  - 类型:`vfat ntfs ios9660`
  - `mount -t vfat /dev/sdb1 /media/disk`
  - 卸载 `umount`
- `df` 查看挂载硬盘情况 
- `du`显示该目录下所有文件
  - `-c` 显示文件大小
  - `-h` 用易读格式输出
- 排序数据 `sort XXfile`
  - `-n` 按照数值方式排序
  - `-M` 识别月份名排序
- `grep [option] pattern [file]` 在文件中查询某字符或模式
  - `-v` 反向搜索
  - `-c` 计行数
  - `-e` 多个模式
- 压缩单个文件`gzip`
  - `gzip` 压缩文件
  - `gzcat` 查看压缩文件内容
  - `gunzip` 解压文件
- `tar` 归档压缩数据
  - `-c` 创建一个新的tar文件
  - `-r` 追加文件
  - `-t` 显示文件内容
  - `-f` 输出结果到什么目录或什么文件
  - `-x` 提取文件
  - `-z` 将输出重定向给gzip解压内容
  - 下载了开源软件之后，你会经常看到文件名以.tgz结尾。这些是gzip压缩过的tar文件可以 用命令`tar -zxvf filename.tgz`来解压。
- `bash`
  - `-c string` 运行某个命令
  - `-i` 启动交互bash
  - `-l` 登陆shell形式启动
  - `-r` 目录受限制的bash
  - `-s` 从标准输入中获取命令

## 进程命令

- 在一行中加分号依次运行一系列命令
  - `pwd; ls; cd /etc; pwd`
- 加上括号会使其成为进程列表,生成一个子shell来只执行命令
  -  `(pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL)`
- 后台模式
  - 将命令在后面加上`&` 进入后台模式运行
  - 使用`ps` 或者` jobs`来查看后台作业
  - 也可以讲进程列表置入后台
- 协程
  - `coproc sleep 10`
  - 给进程起名字 `coproc MyJob { sleep 10; } `  (Linux) 可用于通信
- 内建命令与外部命令 可以使用`type`分辨
  - 外部命令会占据大量资源
- `history` 记录之前输入的命令
  - `!!` 上一条命令
  - `! num` 某一条命令
- `alias` 命令缩写输入工具
  - 创建自己的命令 `alias name=cmd`
  - 删除已有命名 `unalias name`

## 环境变量

### 全局变量VS局部变量

全局环境变量对所有的Shell及子Shell都可见;而局部环境变量仅对创建其的Shell可见.在变量名前加`$`来调用变量的值.

- `env`或`printenv`查看全局变量,可用`echo $HOME` 查看某个变量的值
- `set`可以显示为某个进程设定的所有环境变量,包括全局变量、局部变量和用户定义变量.

### 变量设置

#### 设置用户变量

可以通过等号来对变量进行赋值

```bash
my_variable=hello
echo $my_variable
```

尽量遵循系统定义的变量使用大写,用户变量使用小写的规则.

#### 设置全局变量

通过`export`将用户变量导出到全局变量中(不需要加`$`)

```bash
my_variable=hello
export my_variable
```

此时`my_variable`将成为一个全局变量.修改子shell中的全局变量不会影响父shell中的值.

#### 删除环境变量

使用`unset`删除变量

```bash
unset my_variable
```

**$规则:使用变量时要加\$,操作变量时不需要用**.

除自之外,Bash有一些自带的环境变量.

![image-20200410214152339](/images/LinuxShell/image-20200410214152339.png)

#### 设置PATH环境变量

在shell输入一个外部命令时,shell会查找`PATH`环境变量中的命令和程序来执行.

```bash
echo $PATH
```

添加目录进入环境变量

```bash
PATH = $PATH:/myfolder #添加/myfloder路径进入环境变量
PATH = $PATH:. #添加当前目录进入环境变量
```

这种修改只能保持到本次关机或者退出之前.

#### 使得环境变量持久化

各个发行版下的shell有类似于`/etc/bash.bashrc`的配置文件,最常见的四种为

- `$HOME/.bash_profile`
- `$HOME/.bashrc`
- `$HOME/.bash_login`
- `$HOME/.profile`

这些是Bash的启动脚本,在之中加入自己的环境变量即可以使得变量持久化

### 数组变量

同样可以定义数组环境变量,例如

```bash
mytest=(one two three four five)
echo $mytest #输出整个数组(或者是第一个)
echo $mytest[*] #输出整个数组
echo $mytest[3] #输出第四个值
unset mytest[2] #删除第三个值
```

## Vim编辑器

### 基础操作

#### 普通模式下

##### 光标的移动

`h` 左移动光标

`j` 下移动光标

`k` 上移动光标

`l` 右移动光标

##### 翻页

`PageDown` 或 `Ctrl+F`  下翻一页

`PageUp` 或 `Ctrl+B` 上翻一页

`G` 移到缓冲区最后一行

`num G` 移动到缓冲区第num行

`gg` 移到缓冲区第一行

##### 命令行模式

`q` 未修改且退出

`q!` 取消修改并退出

`w `filename 保存到其他文件中

`wq` 保存并退出

##### 编辑数据

`x` 删除当前字符

`dd` 删除当前行

`dw` 删除当前单词

`d$` 删除至行尾

`J` 删除行尾的换行符 

`u` 撤销

`a` 追加数据

`A` 在行尾追加数据

`r char` 用`char`替换光标当前字符

`R text` 用`text`覆盖当前位置数据 直到按下ESC

##### 复制粘贴

`y` 复制

`p` 粘贴

##### 查找与替换

`/ text` 查找某文本

`/+Enter` 或 `n` 查找下一个

`:s/old/new/g` 替换所有old

` :n,ms/old/new/g` 替换n行到m行之间的所有old

`:%s/old/new/g` 替换整个文件中的所有old

` :%s/old/new/gc` 替换整个文件中的所有old，但在每次出现时提示。

## 构建脚本

### 基础

#### 脚本权限

对脚本赋予文件属主执行文件的权限 `chmod u+x test`

将脚本路径加入环境变量可以直接调用脚本

#### 显示消息

`echo` 输出字符串

`echo -n` 将字符串输出在同一行(仅限于Bash)  Mac可以通过`\c`实现

#### 变量的使用

##### 环境变量

在环境变量前加上`$`来使用环境变量,可以使用`${variable}`来严格规定变量名

##### 用户变量

使用等号赋值,在变量、等号和值之间不能出现空格.

```bash
var1=10
var2=20
var3=testing
var4="strill more testing"
```

#### 命令替换

使用反引号字符``或者$()来获取命令输出

```bash
debug=`date`
debug1=$(who)
```

命令替换获得日期并生成文件

```bash
#!/bin/bash
# copy the /usr/bin directory listing to a log file
today=$(date +%y%m%d)
ls /usr/bin -al > log.$today
```

##### 警告

命令替换会创建一个子shell来运行对应的命令。子shell(subshell)是由运行该脚本的shell 所创建出来的一个独立的子shell(child shell)。正因如此，**由该子shell所执行命令是无法 使用脚本中所创建的变量的**。 在命令行提示符下使用路径./运行命令的话，也会创建出子shell;要是运行命令的时候 **不加入路径，就不会创建子shell**。如果你使用的是内建的shell命令，并不会涉及子shell。 在命令行提示符下运行脚本时一定要留心!

#### 重定向输入输出

##### 输出重定向

`>`完成将命令的输出发送到一个文件中,`>>`表示将内容追加到文件中

```bash
$ who > test6
$ cat test6
user     pts/0    Feb 10 17:55
```

##### 输入重定向

输入重定向恰好相反 `command < inputfile`

```bash
$ wc < test6 # wc : wordcount
2 11 60
$
```

##### 内联输入重定向

`command << marker`

在命令行上使用内联输入重定向时，shell会用PS2环境变量中定义的次提示符(参见第6章) 来提示输入数据。下面是它的使用情况。

```bash
$ wc << EOF
> test string 1
> test string 2
> test string 3
> EOF
```

#### 管道(Pipe)

有时需要**将一个命令的输出作为另一个命令的输入**。这可以用重定向来实现，只是有些笨拙。

```bash
$ rpm -qa > rpm.list
$ sort < rpm.list 
abrt-1.1.14-1.fc14.i686 
abrt-addon-ccpp-1.1.14-1.fc14.i686 
abrt-addon-kerneloops-1.1.14-1.fc14.i686 
abrt-addon-python-1.1.14-1.fc14.i686 
abrt-desktop-1.1.14-1.fc14.i686 
abrt-gui-1.1.14-1.fc14.i686 
abrt-libs-1.1.14-1.fc14.i686 
abrt-plugin-bugzilla-1.1.14-1.fc14.i686 
abrt-plugin-logger-1.1.14-1.fc14.i686 
abrt-plugin-runapp-1.1.14-1.fc14.i686 
acl-2.2.49-8.fc14.i686
...
```

管道连接可以解决这个问题,`|`将一个命令的输出重定向到另一个命令中.

`command1 | command2`

Linux会同时运行两个命令,在系统内部连接起来,在第一个命令产生输出的同时，输出会被立即送给第二个命令。**数据传输不会用到任何中间文件或缓冲区**。

`$ rpm -qa | sort | more`

到目前为止，管道最流行的用法之一是将命令产生的大量输出通过管道传送给more命令。 这对ls命令来说尤为常见.

<img src="/images/LinuxShell/image-20200424121839819.png" alt="image-20200424121839819" style="zoom:80%;" />

#### 数学运算

##### expr命令

使用expr命令可以进行一些简单的数学运算

```bash
$expr 1 + 5
6
```

<img src="/images/LinuxShell/image-20200424122302385.png" alt="image-20200424122302385" style="zoom:80%;" />

尽管标准操作符在expr命令中工作得很好，但在脚本或命令行上使用它们时仍有问题出现。 许多expr命令操作符在shell中另有含义(比如星号)。当它们出现在在expr命令中时，会得到一 些诡异的结果。

##### 使用方括号

使用`$[]`处理数学表达式会更加方便

```bash
$ cat test8
    #!/bin/bash
    var1=100
    var2=45
    var3=$[$var1 / $var2]
    echo The final result is $var3
$ chmod u+x test8
$ ./test8
The final result is 2 
$
```

但是bash仅支持整数运算,zshell(zsh)提供了完整的浮点数算术操作,或者也可以使用bc来进行浮点运算.

#### 退出脚本

##### 查看退出状态码

shell中运行的每个命令都使用退出状态码(exit status)告诉shell它已经运行完毕。退出状态 码是一个0~255的整数值，在命令结束运行时由命令传给shell。可以捕获这个值并在脚本中使用。

Linux提供了一个专门的变量`$?`来保存上个已执行命令的退出状态码。对于需要进行检查的 命令，必须在其运行完毕后立刻查看或使用`$?`变量。它的值会变成由shell所执行的最后一条命令的退出状态码。

```bash
$ date
Sat Jan 15 10:01:30 EDT 2014
$ echo $?
0
$
```

<img src="/images/LinuxShell/image-20200424123347544.png" alt="image-20200424123347544" style="zoom:80%;" />

##### exit 命令

使用`exit`可以自行返回状态码

```bash
exit 127
```

### 结构化命令

#### if-then

bash中最基本的结构化语句为

```bash
if command
then
		commands
fi
```

若`if`后的*command*返回状态码为0,则*commands*会运行,若返回状态码非0,则会跳过`then`中的内容.

同时,也有`else`语句,也有`elif`语句

```bash
if command then
commands
else
commands
fi
```

```bash
if command1 then
commands 
elif command2 
then
more commands 
fi
```

#### test 命令

##### 检查变量是否有内容

在if语句中看到的都是普通shell命令。你可能想问，if-then语句是否能测试 命令退出状态码之外的条件。

答案是不能。但在bash shell中有个好用的工具可以帮你通过`if-then`语句测试其他条件。 test命令提供了在`if-then`语句中测试不同条件的途径。如果`test`命令中列出的条件成立， `test`命令就会退出并返回退出状态码0。这样if-then语句就与其他编程语言中的`if-then`语句

以类似的方式工作了。如果条件不成立，`test`命令就会退出并返回非零的退出状态码，这使得 if-then语句不会再被执行。

`test`命令的格式非常简单。` test condition`

`condition`是`test`命令要测试的一系列参数和值。当用在`if-then`语句中时，`test`命令看 起来是这样的。

```bash
if test condition
then 
		commands
fi
```

用`test`可以确定变量中是否有内容

也可以用`[]`替代`test`.

##### 数值比较

<img src="/images/LinuxShell/image-20200425010537599.png" alt="image-20200425010537599" style="zoom:80%;" />

```bash
value1=10
value2=2
if [ $value1 -lt $value2 ]
then echo " $value1 < $value2 " 
else echo " $value1 >= $value2 " 
fi
```

##### 字符串比较

通常需要添加转义字符

<img src="/images/LinuxShell/image-20200425011431504.png" alt="image-20200425011431504" style="zoom:80%;" />

```bash
if [ $value1 \< $value2 ]
then echo " $value1 < $value2 " 
else echo " $value1 >= $value2 " 
fi
```

在比较测试中，大写字母被认为是小于小写字母的。但sort命令恰好相反。当你将同样的 字符串放进文件中并用sort命令排序时，小写字母会先出现。这是由各个命令使用的排序技术 不同造成的。

##### 文件比较

文件比较是最常用,功能最强大的比较形式.

<img src="/images/LinuxShell/image-20200425011634167.png" alt="image-20200425011634167" style="zoom:80%;" />

#### 复合条件

可以用布尔逻辑`&&` `||`来组合条件

```bash
$ cat test22.sh
#!/bin/bash
# testing compound comparisons
#
if [ -d $HOME ] && [ -w $HOME/testing ] then
   echo "The file exists and you can write to it"
else
   echo "I cannot write to the file"
fi
$
$ ./test22.sh
I cannot write to the file $
$ touch $HOME/testing $
$ ./test22.sh
The file exists and you can write to it $
```

#### If-then 高级特性

`(())` 双括号命令允许在比较过程中使用高级数学表达式.

<img src="/images/LinuxShell/image-20200425012126938.png" alt="image-20200425012126938" style="zoom:80%;" />

```bash
$ cat test23.sh
#!/bin/bash
# using double parenthesis #
val1=10
#
if (( $val1 ** 2 > 90 )) then
       (( val2 = $val1 ** 2 ))
       echo "The square of $val1 is $val2"
fi
$
$ ./test23.sh
The square of 10 is 100 
```



`[[]]` 双方括号允许在比较过程中使用字符串的高级特性.可以利用正则表达式来匹配模式.

```bash
$ cat test24.sh
#!/bin/bash
# using pattern matching 
if [[ $USER =~ r* ]]
then
echo "Hello $USER"
else 
   echo "Sorry, I do not know you"
fi
$ ./test24.sh 
Hello rich
$
```

#### case 命令

`case`命令在bash中格式为

```bash
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

也类似于其他语言中的`case`

```bash
$ cat test26.sh #!/bin/bash
# using the case command #
case $USER in
rich | barbara)
       echo "Welcome, $USER"
       echo "Please enjoy your visit";;
testing)
       echo "Special testing account";;
jessica)
       echo "Do not forget to log off when you're done";;
*)
       echo "Sorry, you are not allowed here";;
esac
$
$ ./test26.sh
Welcome, rich
Please enjoy your visit 
$
```


---

> 作者: [hongwei](https://github.com/hongwei7)  
> URL: https://hongwei7.online/linuxshellnote/  

