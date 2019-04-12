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
| Parameter | Meaning |
| --- | --- |
| **-e** | **whether the name exists or not** |
| **-f** | **whether the name exists and is file** |
| **-d** | **whether the name exists and is directory** |
| -b | whether the name exists and is a block device |
| -c | whether the name exists and is a character device |
| -S | whether the name exists and is a Socket |
| -p | whether the name exists and is a FIFO pipe |
| -L | whether the name exists and is a Link file |
