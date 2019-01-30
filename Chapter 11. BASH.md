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

## 4. Bash shell Environment

### Path and sequence of search

Sequence of searching command

1. Relative/Absolute path. e.g. `/bin/ls` or `./ls`
2. alias
3. bash builtin
4. Search `$PATH` sequently.

### Welcome info

`/etc/issue`

- \d: Local date.
- \l: No. of terminal port
- \m: Hardware level. (i386/i486....)
- \n: Host's network name.
- \o: Domain name.
- \r: OS version (uname -r)
- \t: Local time.
- \s: Name of OS.
- \v: Version of OS.

`/etc/motd` (Info displayed after login)

### Configuration file

- **login shell**: Opened by inputting account and password.
  - `/etc/profile`
    - Contents:
      - PATH: Decide whether `sbin` is included in `PATH` or not according to UID.
      - MAIL: Configure mailbox to `/var/spool/mail` according to account.
      - USER: Configure according to account.
      - HOSTNAME: Configure according to hostname.
      - HISTSIZE: Max history command number. Default 1000.
    - Extending call:
      - `/etc/inputrc`
      - `/etc/profile.d/*.sh`: You can put configuration script here to implement alias, etc.
      - `/etc/sysconfig/i18n`: called from `/etc/profile.d/lang.sh` to configure `$LANG`.
    
  - `~/.bash_profile`
    - Only read one of three files `~/.bash_profile`,`~/.bash_login`,`~/.profile`.
  - `~/.bashrc`: Read only when exist.
- **non-login shell**: Opened by GUI or `bash`.
  - `~/.bashrc`
  
#### source

```
source configuration_file
```

Activate configuration file immediately without logout.

### Terminal configuration (stty, set)

### Special Characters

| Symbol | Meaning |
| --- | --- |
| # | Comment symbol in scripts |
| \\ | Escape symbol |
| \| | pipe |
| ; | Multiple commands |
| ~ | Home path |
| $ | Load variable |
| & | Job control |
| ! | Logic NOT |
| / | Path symbol |
| >,>> | Data stream(output) replace,accumulation |
| <,<< | Data stream(input) |
| '' | Without variable-permutation |
| "" | With variable-permutation |
| \`\` | Command executed first |
| $() | Command executed first |
| () | Sub-shell |
| {} | Command-block |
  
## 5. Data Stream Redirection

- Standard output & standard error output
  - stdin: code 0, `<` or `<<`
  - stdout: code 1, `>` or `>>`
  - stderr: code 2, `2>` or `2>>`
 
 e.g.
 ```
 find /home -name .bashrc > list_right 2> list_error
 ```
 
- `/dev/null` blackhole for unnecessary info.

 e.g.
 ```
 find /home -name .bashrc 2> /dev/null
 ```

- `<` and `<<`
  - `<` get input from file instead of keyboard. e.g.`cat > catfile < ~/.bashrc`
  - `<<` define ending symbol. e.g. `cat > catfile <<"eof"`

### ; && ||

- `cmd;cmd`: execute commands sequently.
- `cmd1 && cmd2`: cmd2 only executed when cmd1 return 0(success)
- `cmd1 || cmd2`: cmd2 only executed when cmd1 return non-zero(fail)

> NOTE: The return value of sub-expression will pass forward.`cmd1 && cmd2 || cmd3`


## 6. Pipe

Command1(stdout) | (stdin)Command2(stdout) | (stdin) Command3

The latter command will take the stdout of the formmer as stdin. (**ONLY** standard output, not standard error)

And the latter command **must** be the type of accepting stdin.

### cut, grep

- `cut`: Process information in the unit of **line**.
  - -d: followed by _cutting character_. Combined with `-f`.
  - -f: get the nth slice.
  - -c: specify fixed interval.
```
范例一：将 PATH 变量取出，我要找出第五个路径。
[root@www ~]# echo $PATH
/bin:/usr/bin:/sbin:/usr/sbin:/usr/local/bin:/usr/X11R6/bin:/usr/games:
# 1 |    2   |  3  |    4    |       5      |     6        |    7

[root@www ~]# echo $PATH | cut -d ':' -f 5
# 如同上面的数字显示，我们是以『 : 』作为分隔，因此会出现 /usr/local/bin 
# 那么如果想要列出第 3 与第 5 呢？，就是这样：
[root@www ~]# echo $PATH | cut -d ':' -f 3,5

范例二：将 export 输出的信息，取得第 12 字符以后的所有字符串
[root@www ~]# export
declare -x HISTSIZE="1000"
declare -x INPUTRC="/etc/inputrc"
declare -x KDEDIR="/usr"
declare -x LANG="zh_TW.big5"
.....(其他省略).....
# 注意看，每个数据都是排列整齐的输出！如果我们不想要『 declare -x 』时，
# 就得这么做：

[root@www ~]# export | cut -c 12-
HISTSIZE="1000"
INPUTRC="/etc/inputrc"
KDEDIR="/usr"
LANG="zh_TW.big5"
.....(其他省略).....
# 知道怎么回事了吧？用 -c 可以处理比较具有格式的输出数据！
# 我们还可以指定某个范围的值，例如第 12-20 的字符，就是 cut -c 12-20 等等！

范例三：用 last 将显示的登陆者的信息中，仅留下用户大名
[root@www ~]# last
root   pts/1    192.168.201.101  Sat Feb  7 12:35   still logged in
root   pts/1    192.168.201.101  Fri Feb  6 12:13 - 18:46  (06:33)
root   pts/1    192.168.201.254  Thu Feb  5 22:37 - 23:53  (01:16)
# last 可以输出『账号/终端机/来源/日期时间』的数据，并且是排列整齐的

[root@www ~]# last | cut -d ' ' -f 1
# 由输出的结果我们可以发现第一个空白分隔的字段代表账号，所以使用如上命令：
# 但是因为 root   pts/1 之间空格有好几个，并非仅有一个，所以，如果要找出 
# pts/1 其实不能以 cut -d ' ' -f 1,2 喔！输出的结果会不是我们想要的。
```

- grep: search info line by line.
  - -a: search binary file as text file.
  - -c: count match.
  - -i: igore lowercase or uppercase.
  - -n: output line No.
  - -v: output lines **NOT** containing match.
  - --color=auto: display match with special color.
  
### sort, wc, uniq

- sort
  - -f: ignore case.
  - -b: ignore leading spaces.
  - -M: sort by Month Name sequence.
  - -n: sort by number.
  - -r: reverse sort
  - -u: only one delegate for same lines.
  - -t: _cutting characters_, default \[tab\].
  - -k: sort by which field.
- uniq: only display once for the repeated data.
  - -i: ignore case.
  - -c: count
- wc: 
  - -l: only list how many lines
  - -w: only list how many letters
  - -m: list how many characters

### tee

Save intermediate info to files and pass it to next pipe.

- -a: appending mode.

### tr, col, join, paste, expand

- tr: delete or replace
  - -d: delete the followed string from the info.
  - -s: replace repeated character
- col: usually used to convert \[tab\] to \[space\].
  - -x: convert \[tab\] to \[space\]
  - -b: Only keep character after '/'.
- join: merge files
  - -t: _cutting character_. default compare the first field.
  - -i: ignore case.
  - -1: the analyzed field of file1
  - -2: the analyzed field of file2
- paste: combine lines in two files with \[tab\]
  - -d: _cutting character_
- expand: convert \[tab\] to \[space\]
  - -t: convert \[tab\] to #No. \[space\]

### split

### xargs
