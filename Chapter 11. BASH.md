# Chapter 11. BASH

## 1. The BASH shell

### What is Shell?

All interfaces that can operate applications are called **shell**, including **Command Line** and **GUI**.

### Why command line?

- The command line(like **bash**) of almost all distributions are _the same_!
- Much faster when remote control(e.g. ssh).
- Advanced administration.

### Valid shell and /etc/shells

**Bourne SHell(sh)**, **Bourne Again SHell(bash)**.
- /bin/sh
- /bin/bash (Defualt Linux shell)
- /bin/ksh (Kornshell from AT&T Bell lab. compatible with bash)
- /bin/tcsh (advanced csh)
- /bin/csh (C Shell)
- /bin/zsh (based on ksh)

Different users correspond to different shell. The information is stored in _/etc/passwd_

e.g. root ~ /bin/bash     bin ~ /sbin/nologin

### Functions of BASH shell

- history: up to 1000 entries(:arrow_up: :arrow_down:)
- \[tab\]: **Command completion** and **File name completion**.
- \[tab\]\[tab\]: list all available **Command** and **File name**.
- alias: e.g. alias lm='ls -al'
- job control/foreground/background
- **_Shell Scripts_**
- Wildcard(Regular Expression?)

### BASH shell built-in command : type
```
type [-tpa] name
# -t : show the meaning of 'name'
#     file: external command
#     alias: configured alias command
#     builtin: bash built-in
# -p : if 'name' is external command, it will show complete file name.
# -a : list all command containing 'name' according to PATH.
```
e.g. check whether `ls` is a built-in command.
`type ls`

### Execute Command

Use **\\** to escape!

## 2. Shell Variable

All uppercase characters.

### echo

We use **echo** to get variable.

```
# echo $variable
echo $PATH
echo ${PATH}
```

### Rules of variable name.

- **No SPACE**
```
myname = Ruloch #WRONG!
myname = Ruloch L #WRONG!
```

- Letter beginning

`2myname=Ruloch #WRONG!`

- \" and \'
```
var="lang is $LANG"  # echo $var -> lang is en_US
var='lang is $LANG'  # echo $var -> lang is $LANG
```

- Escape with \\ (turn $,\\, \[SPACE\], \' to normal character)
- Use **\`_Command_\`** or **\$\(_Command_\)** to get result from a command
```
version=$(uname -r)
# echo $version -> x.x.xx-xxx
```

- Extend variable with **\"\$_VAR_\"** or **\$\{_VAR_\}**
```
PATH="$PATH":/home/bin
```

- Use _export_ to turn varible as environment variable

`export PATH`

- Usually, uppercase for system default variable, lowercase for user variable.

- Delete variable with _unset_
```
unset myname
```

### Functions of environment variable

#### _env_

Use `env` to list all environment variable.

- HOME: home directory `~`
- SHELL: which program is used by SHELL now? `/bin/bash`
- HISTSIZE: how many entries of command will be recorded. Default: 1000.
- MAIL: which mailbox is used now?
- PATH: the paths that will be traversed when executing command. Sequence is IMPORTANT.
- LANG: language system.
- RANDOM: program that will be used to generate random number. 
  - `/dev/random`.
  - For BASH, random number is between 0~32767.
  - To generate customized random number.
  ```
  declare -i number=$RANDOM*10/32768 ; echo $number
  ```
#### _set_

Use `set` to list all variable(including environment variable and self-defined variable).

- **PS1** (root@www ~)
  - \\d : Date (e.g. Mon Feb 2)
  - \\H : Host name (kkk.vbird.tsai)
  - \\h : First segment of host name (kkk)
  - \\t : 24-h time (HH:MM:SS)
  - \\T : 12-h time (HH:MM:SS)
  - \\A : 24-h time (HH:MM)
  - \\@ : 12-h time (HH:MM:SS am/pm)
  - \\u : Account name
  - \\v : BASH version information
  - \\w : Complete working directory.
  - \\W : Only display last directory name
  - \\# : Number of command.
  - \\$ : root ~ #  other users ~ $
- $ (Shell PID)
```
echo $$ # to get the PID of current shell
```
- ? (Return value of last command)
  - Success : return 0
  - Error : return error number
- OSTYPE (Operating system type. e.g. linux-gnu)
- HOSTTYPE (Installed software type. e.g. i686)
- MACHTTYPE (Machine type. e.g. i686-redhat-linux-gnu)

#### _export_

Motivation: Make child-program to inherit custom-defined variables. (Child-program only inherit environment variables from father-program)

```
export variableName
```

If no variable name followed, `export` will list all environment variables.

```
export

declare -x HISTSIZE="1000"
declare -x HOME="/root"
declare -x HOSTNAME="www.vbird.tsai"
declare -x INPUTRC="/etc/inputrc"
declare -x LANG="en_US"
declare -x LOGNAME="root"
...
```

### Language configuration _locale_

List all supported languages.

```
locale -a

zh_TW
zh_TW.big5     
zh_TW.euctw
zh_TW.utf8
zu_ZA
zu_ZA.iso88591
zu_ZA.utf8
```

List all variables you can configure.

```
locale

LANG=en_US                   <==主语言的环境
LC_CTYPE="en_US"             <==字符(文字)辨识的编码
LC_NUMERIC="en_US"           <==数字系统的显示信息
LC_TIME="en_US"              <==时间系统的显示数据
LC_COLLATE="en_US"           <==字符串的比较与排序等
LC_MONETARY="en_US"          <==币值格式的显示等
LC_MESSAGES="en_US"          <==信息显示的内容，如菜单、错误信息等
LC_ALL=                      <==整体语系的环境
```

If you configured `LANG` or `LC_ALL`, all other variables will be filled by this two.

All language files are stored in `/usr/lib/locale/`.

### Range of variable. 

**Child program only inherits environment variables from father program**.

- environment variable = global variable
- custom-defined variabl = local variable

### _read_, _array_, _declare_

#### _read_  Reading variable from user-input.

```
read [-pt] variable
```

- \-p : Reminder string
- \-t : Waiting time 

#### _declare_ / _typeset_ Declaring variable's type

```
declare [-aixr] variable
```

- \-a : array-type
- \-i : integer-type
- \-x : to environment variable
- \-r : read-only type, can't be changed or `unset`
- \-p : list type of a variable
> NOTE: Default type of variables is _string_.

> NOTE: Computation in bash environment is limited in integer range.

Turn `-` to `+` to cancel type.

e.g. remove a variable from environment variable list.

```
declare +x variable
```

> NOTE: You can not cancel read-only type unless you re-login.

#### _array_ 

```
var[index]=content
```

```
var[1]="small min"
var[2]="big min"
var[3]="nice min"

echo "${var[1]},${var[2]},${var[3]}"
# small min, big min, nice min

echo "${var}"
# small min, big min, nice min
```

### _ulimit_ 

Limit available resources for each users.

```
ulimit [-SHacdfltu] [quota]
```

- -H: hard limit
- -S: soft limit (can over the limitation, but there will be warning)
- -a: no arguments after, list all limitations.
- -c: max volume for **core file**(the files that was created from crashed program)
- -f: max file size that can be created.(KB)
- -d: max segment in memory.
- -l: available **lock** in memory.
- -t: max CPU time that can be used. (s)
- -u: max process number for a user.

Simplest way to recover _ulimit_ is to relogin.

> Warning: You can not increase file size when you have configured by `ulimit -f`


### Delete, replacement, change of variables

#### Delete

```
${variable#/*:}
${variable##/*:}
${variable%/*:}
${variable%%/*:}
```
- `${...}` is necessary.
- `variable` is the variable name
- Direction
  - `#` from left to right and only delete the shortest match.
  - `##` from left to right and only delete the longest match.
  - `%` from right to left and only delete the shortest match.
  - `%%` from right to left and only delete the longest match.
- Regular Expression.

e.g.
```

范例一：先让小写的 path 自定义变量配置的与 PATH 内容相同
[root@www ~]# path=${PATH}
[root@www ~]# echo $path
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:
/usr/sbin:/usr/bin:/root/bin  <==这两行其实是同一行啦！

范例二：假设我不喜欢 kerberos，所以要将前两个目录删除掉，如何显示？
[root@www ~]# echo ${path#/*kerberos/bin:}
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin


${variable#/*kerberos/bin:}
   上面的特殊字体部分是关键词！用在这种删除模式所必须存在的

${variable#/*kerberos/bin:}
   这就是原本的变量名称，以上面范例二来说，这里就填写 path 这个『变量名称』啦！

${variable#/*kerberos/bin:}
   这是重点！代表『从变量内容的最前面开始向右删除』，且仅删除最短的那个

${variable#/*kerberos/bin:}
   代表要被删除的部分，由于 # 代表由前面开始删除，所以这里便由开始的 / 写起。
   需要注意的是，我们还可以透过通配符 * 来取代 0 到无穷多个任意字符

   以上面范例二的结果来看， path 这个变量被删除的内容如下所示：
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:
/usr/sbin:/usr/bin:/root/bin  <==这两行其实是同一行啦！


范例三：我想要删除前面所有的目录，仅保留最后一个目录
[root@www ~]# echo ${path#/*:}
/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:
/root/bin     <==这两行其实是同一行啦！
# 由于一个 # 仅删除掉最短的那个，因此他删除的情况可以用底下的删除线来看：
# /usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:
# /usr/sbin:/usr/bin:/root/bin  <==这两行其实是同一行啦！

[root@www ~]# echo ${path##/*:}
/root/bin
# 嘿！多加了一个 # 变成 ## 之后，他变成『删除掉最长的那个数据』！亦即是：
# /usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:
# /usr/sbin:/usr/bin:/root/bin  <==这两行其实是同一行啦！
```

#### Replacement

```
${variable/oldString/newString}
${variable//oldString/newString}
```

- `/` Replace the first old string to new string.
- `//` Replace all old strings to new string.

#### Test and change

| Configuration | str is NULL | str is empty string | str is non-empty string |
| --- | --- | --- | --- |
| `var=${str-expr}` | var=expr | var= | var=$str |
| `var=${str:-expr}` | var=expr | var=expr | var=$str |
| `var=${str+expr}` | var= | var=expr | var=expr |
| `var=${str:+expr}` | var= | var= | var=expr |
| `var=${str=expr}` | str=var=expr | str=$str,var= | str=$str,var=$str |
| `var=${str:=expr}` | str=var=expr | str=var=expr | str=$str,var=$str |
| `var=${str?expr}` | stderr<<expr | var= | var=$str |
| `var=${str:?expr}` | stderr<<expr | stderr<<expr | var=$str |

## 3. Alias and History

### alias, unalias

```
alias newCommand='command'
```
e.g.
```
alias lm='ls -al | more
alias rm='rm -i'
```

```
unalias newCommand
```
e.g.
```
unalias lm
```

### history

```
history [n]
history [-c]
history [-raw] histfiles
```

- n: list last n lines of commands.
- -c: clear all history commands of current shell.
- -a: add new history command of current shell to `histfiles` or default `~/.bash_history`.
- -r: read history commands from `histfiles` to current shell.
- -w: write current history to `histfiles`

```
!number
!command
!!
```

- number: execute nth command.
- command: execute the most recent command beginning with `command`
- !: execute the last command.