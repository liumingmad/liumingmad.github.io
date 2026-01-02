title: shell script
author: ming
date: 2020-08-17 10:56:00
tags:
---

# shell

### 0.man
```bash
# 1、Standard commands （标准命令）
# 2、System calls （系统调用）
# 3、Library functions （库函数）
# 4、Special devices （设备说明）
# 5、File formats （文件格式）
# 6、Games and toys （游戏和娱乐）
# 7、Miscellaneous （杂项）
# 8、Administrative Commands （管理员命令）
# 9 其他（Linux特定的）， 用来存放内核例行程序的文档。
```

### 1.type 查看是否是内建命令
```bash
#查看命令查找的顺序
type -a ls
type -a cd
```

### 2.echo 打印变量 
```bash
hello=kkk
echo $hello
```

### 3.export/unset 设置／取消环境变量
```bash
export hello
unset hello
```

### 4.反单引号
```bash
myver=kkk`uname -r`
myver=aaa$(uname -r)
```

### 5.set 查看所有变量

### 6.env/export 查看环境变量

### 7.声明一个int类型变量，并设置为10以内的随机数
```bash
declare -i num=$RANDOM%10
echo $num
```

### 8.当前shell的pid
```bash
echo $$
```

### 9.上一个命令的回传码
```bash
echo $?
```

### 10.等待用户输入
```bash
read myname
# -p 提示信息
# -t 等待秒数
read -p "kkk-->" -t 3 myname
```

### 11.声明某个类型的变量
```bash
#声明int类型
declare -i sum=1+2

#声明一个环境变量
declare -x sum

#声明为只读变量，只有注销再登陆，才能恢复
declare -r sum
sum=kkk
-bash: sum: readonly variable

#设置环境变量
declare -x sum
#取消环境变量
declare +x sum

#查看变量类型
declare -p sum

#定义数组
num[0]=a
num[1]=b
num[2]=c
echo ${num[2]}
```
### 12.设置配额
```bash
ulimit -a
```

### 13.变量内容的删除
```bash
#/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
#test=$PATH

#删除变量内容
#从前向后，最短匹配
echo ${test#/*bin:}
#从前向后，最长匹配
echo ${test##/*bin:}

#从后向前，最短匹配
echo ${test%:*sbin*}
#从后向前，最长匹配
echo ${test%%:*sbin*}

mypath=/var/spool/mail/root
#如果只想保留路径
echo ${mypath%/*}
#如果只想保留文件名
echo ${mypath##/*/}

```

### 14.变量内容的替换
```bash
#替换, 把kkk替换成aaa
echo ${mypath/kkk/aaa}
```

### 15.测试变量
```bash
#如果hello不存在或者是空，则打印默认值ming
echo ${hello:-ming}

#如果hello不存在或者为空，则把两个变量都改成默认值
echo ${hello:=ming}

#如果hello不存在或者为空，则把默认值输出到标准错误
echo ${hello:?ddd}
```

### 16.命令的别名
```bash
#新增别名
alias lm='ls -l | more'

#删除别名
unalias lm
```

### 17.history
```bash
#查看命令历史
history
```

### 18.登陆的欢迎信息
```bash
#/etc/issue
#/etc/issue.net
#/etc/motd
```

### 19.重定向(>, >>, <, <<)
```bash
# 标准输出重定向到11.log
find / -name .bashrc > 11.log

# 标准输出追加到11.log
find / -name .bashrc >> 11.log

# 标准输出追加到11.log, 标准错误重定向到err.log
find / -name .bashrc > 11.log 2>err.log

# 标准输出追加到11.log, 标准错误重定向到黑洞
find / -name .bashrc > 11.log 2> /dev/null

# 标准输出追加到11.log, 标准错误重定向到标准输出
find / -name .bashrc > 11.log 2>&1

# 标准输入从11.log中读取，标准输出重定向到22
cat > 22 < 11.log

# <<表示如果输入nani，则结束输入，相当于输入EOF
cat > 22 << "nani"
```

### 20.多条命令(;/,/&&/||)
```bash
# 按顺序执行多条命令
hello=aaa; echo $hello; echo ${hello}bb

# 如果ls aa返回0，则不执行创建目录，否则，执行创建
ls aa || mkdir aa
# 如果不存在，则把标准错误扔进黑洞
ls aa 2> /dev/null || mkdir aa 

# 如果aa存在，则在aa下创建22
ls aa && touch ./aa/22

# 无论bb是否存在，第三个命令都会执行
ls bb || mkdir bb && touch ./bb/11

# 如果cc存在，则显示exist, 否则显示not exist
ls cc 2>/dev/null && echo "exist" || echo "not exist"

```

### 21.管道
```bash
# 把前一个标准输出作为下一个命令的标准输入
ls -l /etc | grep sys* | less
```

### 22.cut, grep
```bash
#/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
# 输出第二段/usr/local/bin
echo $PATH | cut -d ":" -f 2

# 把环境变量，去掉declare -x
export | cut -d ' ' -f 3

# 功能同上
export | cut -c 12-

# 获取登陆列表中的第一段
last | cut -d ' ' -f 1

# 过滤包含ming的行
last | grep "ming"

# 取出环境变量中，包含PATH的行，并且只显示key=value部分
export | grep "PATH" | cut -d ' ' -f3

# 筛选文件11中，包含k或s的行
grep -n "[ks]" 11
```

### 23.sort, wc, uniq
```bash
# 对／etc/passwd文件按:分隔的第3列, 按数字大小排序，然后，输出包含bash的行
cat /etc/passwd | sort -t ":" -k 3 -n | grep "bash"

# 对登陆列表排序，去重，统计行数
last | cut -d " " -f 1 | sort | uniq -c

# 统计标准输入中的行数-l, 单词数-w, 字符数-m
cat 11 | wc -l

# 统计这个月登陆了几次
last | grep "Dec" | wc -l
last | grep $(date | cut -d " " -f 2) | grep "[a-zA-Z]" | grep -v "wtmp" | wc -l
```

### 24.双重重定向tee
```bash
# 保存一份标准输出到文件
ls -l | tee -a 11.log
```

### 25.字符删除或替换tr
```bash
# 小写改大写
last | tr "a-z" "A-Z"

# 把PATH中的:替换成换行
echo $PATH | tr -s ":" "\n"

# 然后删除所有的／
echo $PATH | tr -s ":" "\n" | tr -d "/"
```

### 26.grep
```bash
# 搜索包含the的行, 忽略大小写-i, 并显示行号-n
grep -ni "the" regular_express.txt

# 搜索正则，并显示行号
grep -n "t[ae]st" regular_express.txt 

# 搜索文件中，非空白行，且不是#号开头的行
cat /etc/systemd/user.conf | grep -v "^$" | grep -v "^#"

```

### 27.sed(新增，删除，替换)
```bash
# 删除文件中2-5行
nl /etc/passwd | sed "2,5 d"

# 删除第5到最后一行
nl /etc/passwd | sed "5,$ d"

# 在2-5行末尾追加what kkk
nl /etc/passwd | sed "2,5 a what kkk"

# 在第2行前面插入what kkk
nl /etc/passwd | sed "2 i what kkk"

# 把2-5行替换成一行bbbbbb
nl /etc/passwd | sed "2,5 c bbbbbb"

# 打印2-5行
nl /etc/passwd | sed -n "2,5 p"

# 把忽略大小写的the替换成THE
nl regular_express.txt | sed "s/[tT][hH][eE]/THE/g" | grep -i "THE"

# 截取ip
/sbin/ifconfig eth0 | grep "inet" | sed "s/^.*inet.//" | sed "s/ netmask.*//"

# 搜索文件中带有MAN的行，删掉#之后的注释，再删掉空白行
cat /etc/manpath.config | grep "MAN" | sed "s/#.*//g" | sed "/^$/d"

# 把末尾的.替换成!
sed -i "s/\.$/\!/g" regular_express.txt

# 在最后一行追加kkkkkbbb
sed -i "$ a kkkkkbbb" regular_express.txt
```

### 28.egrep, fgrep
```bash
# grep对+要转义才能当作正则的+
grep "go\+d" regular_express.txt

# egrep直接支持+
egrep "go+d" regular_express.txt

# fgrep完全不支持正则, 单纯搜索go+d
fgrep "go+d" regular_express.txt

# 搜索文件中包含*的文件的文件名
find / -type f | xargs -n 10 fgrep -l "*"
```

### 29.printf

### 30.awk
```bash
# 获取登陆信息的第1和3列
last | awk '{print $1 "\t" $ 3}'

# 获取第三列小于10的行
cat /etc/passwd | awk 'BEGIN {FS=":"} $3<10 {print $1 "\t" $3}'

```

### 31.diff
```bash
# 删除第4行，并替换第6行
cat /etc/passwd | sed -e "4 d" -e "6 s/^.*$/no six line/" > passwd

diff /etc/passwd passwd

cmp /etc/passwd passwd

```

### 32.date
```bash
# 格式化日期
date +"%Y%m%d"
```

### 33.test
```bash
# -e 测试文件是否存在
test -e test1.sh && echo "exist" || echo "not exist"
```

### 34.shell script
```bash
#!/bin/bash
# author:ming

#echo 'hello world'

#read -p "please input your name:" filename
#touch "${filename}_$(date +'%Y%m%d')"

#read -p "please input first value:" first
#read -p "please input second value:" second
#echo $(( $first * $second ))

#pwd
#read -p "please input filename:" filename
##test ! -e $filename && echo "$filename does not exist" && exit 0
#test -e $filename || (echo "$filename does not exist" ; exit 0)
#test -d $filename && echo "$filename is directory" || echo "$filename is regular file"
#test -x $filename && echo "executable" || echo "no execute"

#str1=ming
#str2=ming11
#[ "$str1" == "$str2" ]
#echo $?

#read -p "Please choise Y/N:" res
#[ "$res" == "Y" -o "$res" == "y" ] && echo "OK, continue" && exit 0
#[ "$res" == "N" -o "$res" == "n" ] && echo "Oh, interrupt!" && exit 0
#echo "I don't know what your choice is" && exit 0

#echo "program name:$0"
#echo "args count:$#"
#echo "args count:$@"
#shift 2
#[ "$#" -lt "2" ] && echo "args too short"
#echo "args:$*"
#echo "first:$1"
#echo "second:$2"

read -p "Please chose Y/N:" res
if [ "$res" == "Y" ] || [ "$res" == "y" ]; then
	echo "OK, continue"
elif [ "$res" == "N" ] || [ "$res" == "n" ]; then
	echo "Oh, interrupt"
else
	echo "I don't know what your choice is"
fi


exit 0
```

### 35.用户管理
```bash
# 创建用户
useradd -g 初始组 -G 次要组 用户名

# 设置密码
passwd ming2

# 查看当前用户所属的组
groups

# 切换有效组, exit可以切换回上一次的结果
newgrp ming
exit

# 删除用户, -r 删除用户主文件夹
userdel -r ming

# 用户指纹
finger ming

```

### 36.定时任务at
```bash
# 启动atd服务
/etc/init.d/atd start

# 5分钟后，执行任务
at now + 5 minutes

# 查看任务列表
atq

# 删除4号任务
atrm 4
```

### 37.循环定时任务crontab
```bash
# 查看服务进程是否启动
/etc/init.d/cron status

# 查看所有任务
crontab -l

# 编辑任务
crontab -e

# 每隔一分钟，追加一行日志文件
*/1 * * * * echo "$(date)" >> /home/ming/test/corn.log
```

### 38.前后台程序jobs/fg
```bash
# 在后台执行tar命令, 标准错误重定向到标准输出，标准输出写入到tar.log中
tar -cvf test.tar test/ > tar.log 2>&1 &

# 查看当前shell及其子进程
ps -l

# ctrl + z可以把vim 11.log切换到后台并暂停
vim 11.log

# 查看所有后台进程
jobs -l

# 把号码为2的后台进程切换到前台
fg %2

# 查看信号的值
kill -l

# 正常方式终止1号任务
kill -15 %1

```

### 39.nohup
```bash
# 忽略父进程的信号
nohup ping www.google.com &

# 杀掉ping的后台进程
kill -2 $(ps -aux | grep ping | awk '{ print $2 }' | head -n 1)


```

### 40.进程管理
```bash
# 查看所有进程
ps -aux

# 查看进程树
ps axjf

# 查看当前的shell进程之下的子进程
ps -l

# top 命令，每隔2秒刷新一次，$$ bash pid
top -d 2 -p $$

# 查看最耗资源的进程
top -d 2
shift + p

# 查找僵尸进程
pstree -Aup
```

### 41.查看系统信息
```bash
# 查看系统信息
uname -a

# 查看系统负载
uptime

```

### 42.查看网络状态
```bash
# 查看tcp的连接
netstat -aptl

# 查看udp的连接 
netstat -apul
```

### 43.查看内核产生的信息
```bash
dmesg | more
```

### 44.查看正在使用当前文件的进程
```bash
fuser -uv .
```

### 45.查看正在被使用的文件
```bash
lsof -u ming -Ua
```

### 46.linux启动流程
```bash
# 1.BIOS
# 2.grub
# 3.kernel
# 4.systemd
```

### 47.systemd
```bash
# http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html

```

### 48.chown
```bash
# 修改 用户:组
chown ming:ming android/
```

### 49.tar
```bash
# 压缩
tar -cvf test.tar test/

# 解压到当前目录
tar -xvf test.tar

# 不解压, 只列出压缩包内的文件列表
tar -tvf test.zip
```

