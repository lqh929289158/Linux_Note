# Chapter 13. Shell Script.md

## 1. What is Shell script?

- Commands are read from up to down and from left to right.
- \[space\]\[tab\].etc will be ignored.
- Null line will be ignored too.
- Execute the command(a line or a series of) when a \[Enter\] was read.
- Use `\` to escape to next line.(The two lines are still one command.)
- Use `#` to comment.(For a line)
- Use `bash shell.sh` or `sh shell.sh` or `./shell.sh` to execute script.

### Write the 1st script

```
[root@www ~]# mkdir scripts; cd scripts
[root@www scripts]# vi sh01.sh
#!/bin/bash
# Program:
#       This program shows "Hello World!" in your screen.
# History:
# 2005/08/23	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "Hello World! \a \n"
exit 0
```

- Line 1: `#!/bin/bash` declares the shell used by this script.
- Line 2~4: Comments
  - 1. Contents and functions
  - 2. Version
  - 3. Author and contact
  - 4. Date
  - 5. History record
- Line 5~6: Environment variable declaration. Common variable `PATH` and `LANG`. So that you have no need to write absolute path.
- Line 7: `echo` command to print contents on screen.
- Line 8: return value.

> NOTE: Return value will be passed to a variable called `$?`. So if you call `echo $?` after executing the script, then you will see the return value.

Execute the script by
```
[root@www scripts]# sh sh01.sh
Hello World !
```
Or
```
[root@www scripts]# chmod a+x sh01.sh; ./sh01.sh
```

## 2. Simple practice of shell script

### Interactive script

Let the variable to be the value given by user input.

```

[root@www scripts]# vi sh02.sh
#!/bin/bash
# Program:
#	User inputs his first name and last name.  Program shows his full name.
# History:
# 2005/08/23	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input your first name: " firstname  # 提示使用者输入
read -p "Please input your last name:  " lastname   # 提示使用者输入
echo -e "\nYour full name is: $firstname $lastname" # 结果由萤幕输出
```
### Date

```
[root@www scripts]# vi sh03.sh
#!/bin/bash
# Program:
#	Program creates three files, which named by user's input 
#	and date command.
# History:
# 2005/08/23	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

# 1. 让使用者输入文件名称，并取得 fileuser 这个变量；
echo -e "I will use 'touch' command to create 3 files." # 纯粹显示资讯
read -p "Please input your filename: " fileuser         # 提示使用者输入

# 2. 为了避免使用者随意按 Enter ，利用变量功能分析档名是否有配置？
filename=${fileuser:-"filename"}           # 开始判断有否配置档名

# 3. 开始利用 date 命令来取得所需要的档名了；
date1=$(date --date='2 days ago' +%Y%m%d)  # 前两天的日期
date2=$(date --date='1 days ago' +%Y%m%d)  # 前一天的日期
date3=$(date +%Y%m%d)                      # 今天的日期
file1=${filename}${date1}                  # 底下三行在配置档名
file2=${filename}${date2}
file3=${filename}${date3}

# 4. 将档名创建吧！
touch "$file1"                             # 底下三行在创建文件
touch "$file2"
touch "$file3"
```

### Arithmetic Operation

Many ways to evaluate expression.

- 1. `$((expression))`
- 2. `declare -i var=[expression]`

```

[root@www scripts]# vi sh04.sh
#!/bin/bash
# Program:
#	User inputs 2 integer numbers; program will cross these two numbers.
# History:
# 2005/08/23	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "You SHOULD input 2 numbers, I will cross them! \n"
read -p "first number:  " firstnu
read -p "second number: " secnu
total=$(($firstnu*$secnu))
echo -e "\nThe result of $firstnu x $secnu is ==> $total"
```
Or we can use:
```
declare -i total=$firstnu*$secnu
```
Or
```
var=$((运算内容))
```

> NOTE: When you use `$((expression))`, there is no problem if you add \[space\] in expression.

### How to execute script

- sh xxx.sh
- source xxx.sh
  - **The environment variables will be left to the father program.**
- ./xxx.sh
  - the script will create a **new bash environment** to execute itself.
  - in another word, the script is executed in the bash of a subprogram.
  - The key point here: **The environment variables and results defined in subprogram will not pass to father program!**

## 3. Judgement and Branch

Review:

- `$?` the return value of the last command.
- `cmd1 && cmd2` if cmd1(`$?` == 0) then cmd2
- `cmd1 || cmd2` if cmd1(`$?` != 0) then not cmd2
- `test -e /dmtsai` test whether `/dmtsai` exists or not

```
[root@www ~]# test -e /dmtsai && echo "exist" || echo "Not exist"
Not exist  <==结果显示不存在啊！
```

### test

```
test -e filename
```
| para | file exist and type |
| --- | --- |
| **-e** | **whether the name exists or not** |
| **-f** | **whether the name exists and is file** |
| **-d** | **whether the name exists and is directory** |
| -b | whether the name exists and is a block device |
| -c | whether the name exists and is a character device |
| -S | whether the name exists and is a Socket |
| -p | whether the name exists and is a FIFO pipe |
| -L | whether the name exists and is a Link file |

| para | file accessability |
| --- | --- |
| -r | can be read or not |
| -w | can be write or not |
| -x | can be execute or not |
| -u | SUID? |
| -g | SGID? |
| -k | Sticky bit? |
| -s | null file? |

```
test file1 -nt file2
```
| para | file comparasion |
| --- | --- |
| -nt | file1 newer than file2? |
| -ot | file1 older than file2? |
| -ef | file1 the same file with file2?(the same inode?) |

```
test n1 -eq n2
```
| para | integer comarasion |
| --- | --- |
| -eq | n1 == n2 |
| -ne | n1 != n2 |
| -gt | n1 > n2 |
| -lt | n1 < n2 |
| -ge | n1 >= n2 |
| -le | n1 <= n2 |

| cmd | string comparasion |
| --- | --- |
| test -z string | string =="" ? |
| test -n string | string !="" ? |
| test str1 = str2 | str1 == str2 ? |
| test str1 != str2 | str1 != str2 ? |

```
test -r filename -a -x filename
```
| para | multiple condition |
| --- | --- |
| -a | and |
| -o | or |
| ! | not `test ! -x file` not have execution right |

```

[root@www scripts]# vi sh05.sh
#!/bin/bash
# Program:
#	User input a filename, program will check the flowing:
#	1.) exist? 2.) file/directory? 3.) file permissions 
# History:
# 2005/08/25	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

# 1. 让使用者输入档名，并且判断使用者是否真的有输入字串？
echo -e "Please input a filename, I will check the filename's type and \
permission. \n\n"
read -p "Input a filename : " filename
test -z $filename && echo "You MUST input a filename." && exit 0
# 2. 判断文件是否存在？若不存在则显示信息并结束脚本
test ! -e $filename && echo "The filename '$filename' DO NOT exist" && exit 0
# 3. 开始判断文件类型与属性
test -f $filename && filetype="regulare file"
test -d $filename && filetype="directory"
test -r $filename && perm="readable"
test -w $filename && perm="$perm writable"
test -x $filename && perm="$perm executable"
# 4. 开始输出资讯！
echo "The filename: $filename is a $filetype"
echo "And the permissions are : $perm"
```

### `[]` judgement symbol

```
[root@www ~]# [ -z "$HOME" ] ; echo $?
```

> WARNING: Use **\[Space\]** to split each element in the `[]`

```
[  "$HOME"  ==  "$MAIL"  ]
```

- Split all elements with **\[Space\]**
- Use **""** for all variables.
- Use **""** for all constants.

```
[root@www scripts]# vi sh06.sh
#!/bin/bash
# Program:
# 	This program shows the user's choice
# History:
# 2005/08/25	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input (Y/N): " yn
[ "$yn" == "Y" -o "$yn" == "y" ] && echo "OK, continue" && exit 0
[ "$yn" == "N" -o "$yn" == "n" ] && echo "Oh, interrupt!" && exit 0
echo "I don't know what your choice is" && exit 0
```

### Default variable `$0,$1,...`

```
[root@www ~]# file /etc/init.d/syslog
/etc/init.d/syslog: Bourne-Again shell script text executable
# 使用 file 来查询后，系统告知这个文件是个 bash 的可运行 script 喔！
[root@www ~]# /etc/init.d/syslog restart
[root@www ~]# /etc/init.d/syslog stop
```

```
# $n means the n-th parameter
/path/to/scriptname  opt1  opt2  opt3  opt4 
       $0             $1    $2    $3    $4
```

- `$0`: the name of the script executed
- `$n`: the n-th parameter
- `$#`: the number of parameters followed
- `$@`: the string of all parameters, splited by \[Space\].

```

[root@www scripts]# vi sh07.sh
#!/bin/bash
# Program:
#	Program shows the script name, parameters...
# History:
# 2009/02/17	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

echo "The script name is        ==> $0"
echo "Total parameter number is ==> $#"
[ "$#" -lt 2 ] && echo "The number of parameter is less than 2.  Stop here." \
	&& exit 0
echo "Your whole parameter is   ==> '$@'"
echo "The 1st parameter         ==> $1"
echo "The 2nd parameter         ==> $2"



[root@www scripts]# sh sh07.sh theone haha quot
The script name is        ==> sh07.sh            <==档名
Total parameter number is ==> 3                  <==果然有三个参数
Your whole parameter is   ==> 'theone haha quot' <==参数的内容全部
The 1st parameter         ==> theone             <==第一个参数
The 2nd parameter         ==> haha               <==第二个参数
```

### shift

left shift parameters. Maybe you can consume the parameters by how many you want, not to specify the sequence number of them.
```
[root@www scripts]# vi sh08.sh
#!/bin/bash
# Program:
#	Program shows the effect of shift function.
# History:
# 2009/02/17	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

echo "Total parameter number is ==> $#"
echo "Your whole parameter is   ==> '$@'"
shift   # 进行第一次『一个变量的 shift 』
echo "Total parameter number is ==> $#"
echo "Your whole parameter is   ==> '$@'"
shift 3 # 进行第二次『三个变量的 shift 』
echo "Total parameter number is ==> $#"
echo "Your whole parameter is   ==> '$@'"



[root@www scripts]# sh sh08.sh one two three four five six <==给予六个参数
Total parameter number is ==> 6   <==最原始的参数变量情况
Your whole parameter is   ==> 'one two three four five six'
Total parameter number is ==> 5   <==第一次偏移，看底下发现第一个 one 不见了
Your whole parameter is   ==> 'two three four five six'
Total parameter number is ==> 2   <==第二次偏移掉三个，two three four 不见了
Your whole parameter is   ==> 'five six'
```

## 4. Condition judgement and branch

### `if` and `then`
```
if [ 条件判断式 ]; then
	当条件判断式成立时，可以进行的命令工作内容；
fi   <==将 if 反过来写，就成为 fi 啦！结束 if 之意！
```

The logical relation **inside \[\]** use `-o -a`.
The logical relation **outside \[\]** use `|| &&`.

```
[ "$yn" == "Y" -o "$yn" == "y" ]
[ "$yn" == "Y" ] || [ "$yn" == "y" ]
```

```
[root@www scripts]# cp sh06.sh sh06-2.sh  <==用改的比较快！
[root@www scripts]# vi sh06-2.sh
#!/bin/bash
# Program:
#       This program shows the user's choice
# History:
# 2005/08/25    VBird   First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input (Y/N): " yn

if [ "$yn" == "Y" ] || [ "$yn" == "y" ]; then
	echo "OK, continue"
	exit 0
fi
if [ "$yn" == "N" ] || [ "$yn" == "n" ]; then
	echo "Oh, interrupt!"
	exit 0
fi
echo "I don't know what your choice is" && exit 0
```

### `if` and `then` and `elif` and `else`

```
# 多个条件判断 (if ... elif ... elif ... else) 分多种不同情况运行
if [ 条件判断式一 ]; then
	当条件判断式一成立时，可以进行的命令工作内容；
elif [ 条件判断式二 ]; then
	当条件判断式二成立时，可以进行的命令工作内容；
else
	当条件判断式一与二均不成立时，可以进行的命令工作内容；
fi
```
```
[root@www scripts]# cp sh06-2.sh sh06-3.sh
[root@www scripts]# vi sh06-3.sh
#!/bin/bash
# Program:
#       This program shows the user's choice
# History:
# 2005/08/25    VBird   First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input (Y/N): " yn

if [ "$yn" == "Y" ] || [ "$yn" == "y" ]; then
	echo "OK, continue"
elif [ "$yn" == "N" ] || [ "$yn" == "n" ]; then
	echo "Oh, interrupt!"
else
	echo "I don't know what your choice is"
fi
```
```

[root@www scripts]# vi sh09.sh
#!/bin/bash
# Program:
#	Check $1 is equal to "hello"
# History:
# 2005/08/28	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

if [ "$1" == "hello" ]; then
	echo "Hello, how are you ?"
elif [ "$1" == "" ]; then
	echo "You MUST input parameters, ex> {$0 someword}"
else
	echo "The only parameter is 'hello', ex> {$0 hello}"
fi
```

### Using `case ... esac`

```

case  $变量名称 in   <==关键字为 case ，还有变量前有钱字号
  "第一个变量内容")   <==每个变量内容建议用双引号括起来，关键字则为小括号 )
	程序段
	;;            <==每个类别结尾使用两个连续的分号来处理！
  "第二个变量内容")
	程序段
	;;
  *)                  <==最后一个变量内容都会用 * 来代表所有其他值
	不包含第一个变量内容与第二个变量内容的其他程序运行段
	exit 1
	;;
esac                  <==最终的 case 结尾！『反过来写』思考一下！
```

```

[root@www scripts]# vi sh09-2.sh
#!/bin/bash
# Program:
# 	Show "Hello" from $1.... by using case .... esac
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

case $1 in
  "hello")
	echo "Hello, how are you ?"
	;;
  "")
	echo "You MUST input parameters, ex> {$0 someword}"
	;;
  *)   # 其实就相当於万用字节，0~无穷多个任意字节之意！
	echo "Usage $0 {hello}"
	;;
esac
```

### Use `function`

```
function fname() {
	程序段
}
```

```
[root@www scripts]# vi sh12-2.sh
#!/bin/bash
# Program:
#	Use function to repeat information.
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

function printit(){
	echo -n "Your choice is "     # 加上 -n 可以不断行继续在同一行显示
}

echo "This program will print your selection !"
case $1 in
  "one")
	printit; echo $1 | tr 'a-z' 'A-Z'  # 将参数做大小写转换！
	;;
  "two")
	printit; echo $1 | tr 'a-z' 'A-Z'
	;;
  "three")
	printit; echo $1 | tr 'a-z' 'A-Z'
	;;
  *)
	echo "Usage $0 {one|two|three}"
	;;
esac
```

Function has its own local variable, also specified by `$0,$1,...`. `$0` is the name of the function?. The `$1,$2,...` are the parameters.

```
[root@www scripts]# vi sh12-3.sh
#!/bin/bash
# Program:
#	Use function to repeat information.
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

function printit(){
	echo "Your choice is $1"   # 这个 $1 必须要参考底下命令的下达
}

echo "This program will print your selection !"
case $1 in
  "one")
	printit 1  # 请注意， printit 命令后面还有接参数！
	;;
  "two")
	printit 2
	;;
  "three")
	printit 3
	;;
  *)
	echo "Usage $0 {one|two|three}"
	;;
esac
```

## 5. Loop 

### `while..do..done..until..do..done`

```
while [ condition ]  <==中括号内的状态就是判断式
do            <==do 是回圈的开始！
	程序段落
done          <==done 是回圈的结束

until [ condition ]
do
	程序段落
done
```

```
[root@www scripts]# vi sh13.sh
#!/bin/bash
# Program:
#	Repeat question until user input correct answer.
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

while [ "$yn" != "yes" -a "$yn" != "YES" ]
do
	read -p "Please input yes/YES to stop this program: " yn
done
echo "OK! you input the correct answer."


[root@www scripts]# vi sh13-2.sh
#!/bin/bash
# Program:
#	Repeat question until user input correct answer.
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

until [ "$yn" == "yes" -o "$yn" == "YES" ]
do
	read -p "Please input yes/YES to stop this program: " yn
done
echo "OK! you input the correct answer."
```

```
[root@www scripts]# vi sh14.sh
#!/bin/bash
# Program:
#	Use loop to calculate "1+2+3+...+100" result.
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

s=0  # 这是加总的数值变量
i=0  # 这是累计的数值，亦即是 1, 2, 3....
while [ "$i" != "100" ]
do
	i=$(($i+1))   # 每次 i 都会添加 1 
	s=$(($s+$i))  # 每次都会加总一次！
done
echo "The result of '1+2+3+...+100' is ==> $s"
```

### `for...do...done`

```
for var in con1 con2 con3 ...
do
	程序段
done
```

```
[root@www scripts]# vi sh15.sh
#!/bin/bash
# Program:
# 	Using for .... loop to print 3 animals
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

for animal in dog cat elephant
do
	echo "There are ${animal}s.... "
done
```
```
[root@www scripts]# vi sh16.sh
#!/bin/bash
# Program
#       Use id, finger command to check system account's information.
# History
# 2009/02/18    VBird   first release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
users=$(cut -d ':' -f1 /etc/passwd)  # 撷取帐号名称
for username in $users               # 开始回圈进行！
do
        id $username
        finger $username
done
```
```
[root@www scripts]# vi sh18.sh
#!/bin/bash
# Program:
#	User input dir name, I find the permission of files.
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

# 1. 先看看这个目录是否存在啊？
read -p "Please input a directory: " dir
if [ "$dir" == "" -o ! -d "$dir" ]; then
	echo "The $dir is NOT exist in your system."
	exit 1
fi

# 2. 开始测试文件罗～
filelist=$(ls $dir)        # 列出所有在该目录下的文件名称
for filename in $filelist
do
	perm=""
	test -r "$dir/$filename" && perm="$perm readable"
	test -w "$dir/$filename" && perm="$perm writable"
	test -x "$dir/$filename" && perm="$perm executable"
	echo "The file $dir/$filename's permission is $perm "
done
```

### seq
```
[root@www scripts]# vi sh17.sh
#!/bin/bash
# Program
#       Use ping command to check the network's PC state.
# History
# 2009/02/18    VBird   first release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
network="192.168.1"              # 先定义一个网域的前面部分！
for sitenu in $(seq 1 100)       # seq 为 sequence(连续) 的缩写之意
do
	# 底下的程序在取得 ping 的回传值是正确的还是失败的！
        ping -c 1 -w 1 ${network}.${sitenu} &> /dev/null && result=0 || result=1
	# 开始显示结果是正确的启动 (UP) 还是错误的没有连通 (DOWN)
        if [ "$result" == 0 ]; then
                echo "Server ${network}.${sitenu} is UP."
        else
                echo "Server ${network}.${sitenu} is DOWN."
        fi
done
```

### `for...do...done` data process

```
for (( 初始值; 限制值; 运行步阶 ))
do
	程序段
done
```
```
[root@www scripts]# vi sh19.sh
#!/bin/bash
# Program:
# 	Try do calculate 1+2+....+${your_input}
# History:
# 2005/08/29	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input a number, I will count for 1+2+...+your_input: " nu

s=0
for (( i=1; i<=$nu; i=i+1 ))
do
	s=$(($s+$i))
done
echo "The result of '1+2+3+...+$nu' is ==> $s"
```

## 6. Track and debug

```
[root@www ~]# sh [-nvx] scripts.sh
选项与参数：
-n  ：不要运行 script，仅查询语法的问题；
-v  ：再运行 sccript 前，先将 scripts 的内容输出到萤幕上；
-x  ：将使用到的 script 内容显示到萤幕上，这是很有用的参数！

范例一：测试 sh16.sh 有无语法的问题？
[root@www ~]# sh -n sh16.sh 
# 若语法没有问题，则不会显示任何资讯！

范例二：将 sh15.sh 的运行过程全部列出来～
[root@www ~]# sh -x sh15.sh 
+ PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:/root/bin
+ export PATH
+ for animal in dog cat elephant
+ echo 'There are dogs.... '
There are dogs....
+ for animal in dog cat elephant
+ echo 'There are cats.... '
There are cats....
+ for animal in dog cat elephant
+ echo 'There are elephants.... '
There are elephants....
```
