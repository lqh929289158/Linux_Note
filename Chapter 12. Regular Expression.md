# Chapter 12. Regular Expression

## 1.What is Regular Expression?

## 2.Basic Regular Expression

| Symbol | Meaning |
| --- | --- |
| \[:alnum:\] | 0-9,A-Z,a-z |
| \[:alpha:\] | A-Z,a-z |
| \[:digit:\] | 0-9 |
| \[:lower:\] | a-z |
| \[:upper:\] | A-Z |
| \[:blank:\] | \[space\],\[tab\] |
| \[:punct:\] | punctuation symbol(",',?,!,#,$...) |
| \[:print:\] | any character that can be printed |
| \[:upper:\] | A-Z |
| \[:xdigit:] | 0-9, A-F, a-f |
| \[:cntrl:\] | CR,LF,Tab,Del... |

### Advanced usage of *grep*

**grep** is a command that search string line by line. So, the results printed is of line-unit.

```
[root@www ~]# grep [-A] [-B] [--color=auto] '搜寻字串' filename
选项与参数：
-A ：后面可加数字，为 after 的意思，除了列出该行外，后续的 n 行也列出来；
-B ：后面可加数字，为 befer 的意思，除了列出该行外，前面的 n 行也列出来；
-n ：显示行号
-v ：反向选择
-i ：忽略大小写
--color=auto 可将正确的那个撷取数据列出颜色
```

Basic regular expression used in *grep*:

```
[root@www ~]# grep -n 't[ae]st' regular_express.txt
8:I can't finish the test.
9:Oh! The soup taste good.


[root@www ~]# grep -n 'oo' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
9:Oh! The soup taste good.
18:google is the best tools for search keyword.
19:goooooogle yes!


[root@www ~]# grep -n '[^g]oo' regular_express.txt
2:apple is my favorite food.
3:Football game is not use feet only.
18:google is the best tools for search keyword.
19:goooooogle yes!


[root@www ~]# grep -n '[^[:lower:]]oo' regular_express.txt
# 那个 [:lower:] 代表的就是 a-z 的意思！请参考前两小节的说明表格

[root@www ~]# grep -n '[[:digit:]]' regular_express.txt
```

Line beginning and line ending symbol ^ $
`^` in the `[]` and out of the `[]` is DIFFERENT!
```
# No beginning 'the'
[root@www ~]# grep -n '^the' regular_express.txt
12:the symbol '*' is represented as start.

# No beginning lower letters
[root@www ~]# grep -n '^[a-z]' regular_express.txt
2:apple is my favorite food.
4:this dress doesn't fit me.
10:motorcycle is cheap than car.
12:the symbol '*' is represented as start.
18:google is the best tools for search keyword.
19:goooooogle yes!
20:go! go! Let's go.

# No beginning letters
[root@www ~]# grep -n '^[^a-zA-Z]' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
21:# I am VBird
# 命令也可以是： grep -n '^[^[:alpha:]]' regular_express.txt
```

```
# . has special meaning that will be talked below, so we have to use escape symbol(\) here
[root@www ~]# grep -n '\.$' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
4:this dress doesn't fit me.
10:motorcycle is cheap than car.
11:This window is clear.

# Find blank line
[root@www ~]# grep -n '^$' regular_express.txt
22:

# cascade grep to make readable
[root@www ~]# grep -v '^$' /etc/syslog.conf | grep -v '^#'
# 结果仅有 10 行，其中第一个『 -v '^$' 』代表『不要空白行』，
# 第二个『 -v '^#' 』代表『不要开头是 # 的那行』喔！
```

**Any** character `.` and **repeat** character `*`.

```
# Matching g??d
[root@www ~]# grep -n 'g..d' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
9:Oh! The soup taste good.
16:The world <Happy> is the same with "glad".

# Matching at least one continuous `o`
[root@www ~]# grep -n 'goo*g' regular_express.txt
18:google is the best tools for search keyword.
19:goooooogle yes
```

> WARNING:  `grep -n 'o*' regular_express.txt` will grep all data in the file because `o*` means that null character is OK!

```
# Find all strings begin with 'g' and end with 'g'
[root@www ~]# grep -n 'g.*g' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
14:The gd software is a library for drafting programs.
18:google is the best tools for search keyword.
19:goooooogle yes!
20:go! go! Let's go.

# Find all number
[root@www ~]# grep -n '[0-9][0-9]*' regular_express.txt
5:However, this dress is about $ 3183 dollars.
15:You are the best is mean you are the no. 1.
```

Specific repeat times `{}`

```
# Since {} has special meaning in shell, we have to use escape symbol(\)

# Find two 'o'
[root@www ~]# grep -n 'o\{2\}' regular_express.txt
1:"Open Source" is a good mechanism to develop programs.
2:apple is my favorite food.
3:Football game is not use feet only.
9:Oh! The soup taste good.
18:google is the best tools for search keyword.
19:goooooogle yes!

# Find 2~5 'o'
[root@www ~]# grep -n 'go\{2,5\}g' regular_express.txt
18:google is the best tools for search keyword.

# Find 2~INF 'o'
[root@www ~]# grep -n 'go\{2,\}g' regular_express.txt
18:google is the best tools for search keyword.
19:goooooogle yes!
```

| Symbol | Meaning |
| --- | --- |
| ^word | word at begnning of a line |
| word$ | word at end of a line |
| . | any character |
| \ | escape character |
| * | repeat the character before it zero or more times |
| \[list\] | one of the element in list |
| \[n1-n2] | one of the element in the range(mainly 0-9, a-z, A-Z) |
| \[^list] | except all elements in list |
| \{n,m\} | repeat the character before for n~m times |

> WARNING: Special character in bash, like `*` is **DIFFERENT** from the `*` in RE.

> `*` in bash: 0~INF characters

> `*` in RE: Repeat the character before for 0~INF times!

```
# For example, 

#list all filenames begin with a
ls -l a*

#list all files of any name
ls -l | grep 'a*'

#list all filenames begin with a
ls -l | grep '^a.*'
```

### sed
```
[root@www ~]# sed [-nefr] [动作]
选项与参数：
-n  ：使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 
      的数据一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过
      sed 特殊处理的那一行(或者动作)才会被列出来。
-e  ：直接在命令列模式上进行 sed 的动作编辑；
-f  ：直接将 sed 的动作写在一个文件内， -f filename 则可以运行 filename 内的 
      sed 动作；
-r  ：sed 的动作支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)
-i  ：直接修改读取的文件内容，而不是由萤幕输出。

动作说明：  [n1[,n2]]function
n1, n2 ：不见得会存在，一般代表『选择进行动作的行数』，举例来说，如果我的动作
         是需要在 10 到 20 行之间进行的，则『 10,20[动作行为] 』

function 有底下这些咚咚：
a   ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
c   ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d   ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
i   ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
p   ：列印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
s   ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配
      正规表示法！例如 1,20s/old/new/g 就是啦！
```

#### Add/Delete functions in the unit of line.

```
范例一：将 /etc/passwd 的内容列出并且列印行号，同时，请将第 2~5 行删除！
[root@www ~]# nl /etc/passwd | sed '2,5d'
     1  root:x:0:0:root:/root:/bin/bash
     6  sync:x:5:0:sync:/sbin:/bin/sync
     7  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
.....(后面省略).....

# Delete line 3 to last line.
 nl /etc/passwd | sed '3,$d' 
```
> NOTE: Must use `'` for the actions after the `sed`

```
范例二：承上题，在第二行后(亦即是加在第三行)加上『drink tea』字样！
[root@www ~]# nl /etc/passwd | sed '2a drink tea'
     1  root:x:0:0:root:/root:/bin/bash
     2  bin:x:1:1:bin:/bin:/sbin/nologin
drink tea
     3  daemon:x:2:2:daemon:/sbin:/sbin/nologin
.....(后面省略).....

# add line into the front of the current line.
 nl /etc/passwd | sed '2i drink tea'
 
范例三：在第二行后面加入两行字，例如『Drink tea or .....』与『drink beer?』
[root@www ~]# nl /etc/passwd | sed '2a Drink tea or ......\
> drink beer ?'
     1  root:x:0:0:root:/root:/bin/bash
     2  bin:x:1:1:bin:/bin:/sbin/nologin
Drink tea or ......
drink beer ?
     3  daemon:x:2:2:daemon:/sbin:/sbin/nologin
.....(后面省略).....
# Use \ to insert many lines
```

#### Replace/Display function in the unit of line.

```
范例四：我想将第2-5行的内容取代成为『No 2-5 number』呢？
[root@www ~]# nl /etc/passwd | sed '2,5c No 2-5 number'
     1  root:x:0:0:root:/root:/bin/bash
No 2-5 number
     6  sync:x:5:0:sync:/sbin:/bin/sync
.....(后面省略).....


范例五：仅列出 /etc/passwd 文件内的第 5-7 行
[root@www ~]# nl /etc/passwd | sed -n '5,7p'
     5  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
     6  sync:x:5:0:sync:/sbin:/bin/sync
     7  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
```

#### Search and replace partially.

```
sed 's/[string to be replaced]/[string replaced to]/g'
```

#### Modify files directly

```
sed -i '/\.$/\!/g' regular_express.txt
sed -i '$a # This is a test' regular_express.txt
```

## 3. Extended Regular Expression

```
egrep
```
| Symbol | Meaning |
| --- | --- |
| \| | meaning: or. to find one or more strings to match. `gd|good` means `gd` or `good` |
| + | repeat one or more times for the character before |
| ? | zero or one the character before |
| () | meaning: or(partially). `g(la|oo)d` means `glad` or `good` |
| ()+ | group repeat one or more times `A(xyz)+C` means `Axyzxyz....xyzC`

> NOTE: '!','>' are not special character.

## 4. Format of files

### printf

Almost the same as **printf** in C. Followed by variable to be printed.

| Symbol | Meaning |
| --- | --- |
| %ns | n means the number of byte(character) |
| %ni | n means the total digit of integer |
| %N.nf | N:the digit number of all. n: the digit number after dot. |

```

范例一：将刚刚上头数据的文件 (printf.txt) 内容仅列出姓名与成绩：(用 [tab] 分隔)
[root@www ~]# printf '%s\t %s\t %s\t %s\t %s\t \n' $(cat printf.txt)
Name     Chinese         English         Math    Average
DmTsai   80      60      92      77.33
VBird    75      55      80      70.00
Ken      60      90      70      73.33


范例二：将上述数据关於第二行以后，分别以字串、整数、小数点来显示：
[root@www ~]# printf '%10s %5i %5i %5i %8.2f \n' $(cat printf.txt |\
> grep -v Name)
    DmTsai    80    60    92    77.33
     VBird    75    55    80    70.00
       Ken    60    90    70    73.33

范例三：列出 16 进位数值 45 代表的字节为何？
[root@www ~]# printf '\x45\n'
E
```

### awk

Compared with `sed` that deal with data line by line, `awk` prefer to deal with data unit by unit.

The segment symbols of unit are usually `\t` ` `

```
[root@www ~]# awk '条件类型1{动作1} 条件类型2{动作2} ...' filename

[root@www ~]# last -n 5 <==仅取出前五行
root     pts/1   192.168.1.100  Tue Feb 10 11:21   still logged in
root     pts/1   192.168.1.100  Tue Feb 10 00:46 - 02:28  (01:41)
root     pts/1   192.168.1.100  Mon Feb  9 11:41 - 18:30  (06:48)
dmtsai   pts/1   192.168.1.100  Mon Feb  9 11:41 - 11:41  (00:00)
root     tty1                   Fri Sep  5 14:09 - 14:10  (00:01)


[root@www ~]# last -n 5 | awk '{print $1 "\t" $3}'
root    192.168.1.100
root    192.168.1.100
root    192.168.1.100
dmtsai  192.168.1.100
root    Fri
```

| Symbol | Meaning |
| --- | --- |
| NF | the unit number of each line |
| NR | the current line |
| FS | segment symbol(default \[space\]) |

```
[root@www ~]# last -n 5| awk '{print $1 "\t lines: " NR "\t columns: " NF}'
root     lines: 1        columns: 10
root     lines: 2        columns: 10
root     lines: 3        columns: 10
dmtsai   lines: 4        columns: 10
root     lines: 5        columns: 9
# 注意喔，在 awk 内的 NR, NF 等变量要用大写，且不需要有钱字号 $ 啦！
```

Logical expression symbol

| Symbol | Meaning |
| --- | --- |
| > | . |
| < | . |
| >= | . |
| <= | . |
| == | . |
| != | . |

```
[root@www ~]# cat /etc/passwd | \
> awk '{FS=":"} $3 < 10 {print $1 "\t " $3}'
root:x:0:0:root:/root:/bin/bash
bin      1
daemon   2
....(以下省略)....
```

> NOTE: The configuration of `{FS=":"}` will come into effect after line 1!!! You need `BEGIN` keyword to configure environment variable before deal with data.

```
[root@www ~]# cat /etc/passwd | \
> awk 'BEGIN {FS=":"} $3 < 10 {print $1 "\t " $3}'
root     0
bin      1
daemon   2
......(以下省略)......
```

Arithmetic calculation
```

Name    1st     2nd     3th
VBird   23000   24000   25000
DMTsai  21000   20000   23000
Bird2   43000   42000   41000

[root@www ~]# cat pay.txt | \
> awk 'NR==1{printf "%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total" }
NR>=2{total = $2 + $3 + $4
printf "%10s %10d %10d %10d %10.2f\n", $1, $2, $3, $4, total}'
      Name        1st        2nd        3th      Total
     VBird      23000      24000      25000   72000.00
    DMTsai      21000      20000      23000   64000.00
     Bird2      43000      42000      41000  126000.00
```

> NOTE: Use `;` or \[Enter\] to split multiple commands. (Including the actions in \{\}.)


### diff

Compare between files line by line. Usually used for different versions of the a file.

```

[root@www ~]# diff [-bBi] from-file to-file
选项与参数：
from-file ：一个档名，作为原始比对文件的档名；
to-file   ：一个档名，作为目的比对文件的档名；
注意，from-file 或 to-file 可以 - 取代，那个 - 代表『Standard input』之意。

-b  ：忽略一行当中，仅有多个空白的差异(例如 "about me" 与 "about     me" 视为相同
-B  ：忽略空白行的差异。
-i  ：忽略大小写的不同。

范例一：比对 passwd.old 与 passwd.new 的差异：
[root@www test]# diff passwd.old passwd.new
4d3    <==左边第四行被删除 (d) 掉了，基准是右边的第三行
< adm:x:3:4:adm:/var/adm:/sbin/nologin  <==这边列出左边(<)文件被删除的那一行内容
6c5    <==左边文件的第六行被取代 (c) 成右边文件的第五行
< sync:x:5:0:sync:/sbin:/bin/sync  <==左边(<)文件第六行内容
---
> no six line                      <==右边(>)文件第五行内容
# 很聪明吧！用 diff 就把我们刚刚的处理给比对完毕了！
```

### patch

Used to update the old version of file.

Make patch file.
```
范例一：以 /tmp/test 内的 passwd.old 与 passwd.new  制作补丁文件
[root@www test]# diff -Naur passwd.old passwd.new > passwd.patch
[root@www test]# cat passwd.patch
--- passwd.old  2009-02-10 14:29:09.000000000 +0800 <==新旧文件的资讯
+++ passwd.new  2009-02-10 14:29:18.000000000 +0800
@@ -1,9 +1,8 @@   <==新旧文件要修改数据的界定范围，旧档在 1-9 行，新档在 1-8 行
 root:x:0:0:root:/root:/bin/bash
 bin:x:1:1:bin:/bin:/sbin/nologin
 daemon:x:2:2:daemon:/sbin:/sbin/nologin
-adm:x:3:4:adm:/var/adm:/sbin/nologin      <==左侧文件删除
 lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
-sync:x:5:0:sync:/sbin:/bin/sync           <==左侧文件删除
+no six line                               <==右侧新档加入
 shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
 halt:x:7:0:halt:/sbin:/sbin/halt
 mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
 ```
 
 Use the patch file to update old version.
 ```
[root@www ~]# patch -pN < patch_file    <==升级
[root@www ~]# patch -R -pN < patch_file <==还原
选项与参数：
-p  ：后面可以接『取消几层目录』的意思。
-R  ：代表还原，将新的文件还原成原来旧的版本。

范例二：将刚刚制作出来的 patch file 用来升级旧版数据
[root@www test]# patch -p0 < passwd.patch
patching file passwd.old
[root@www test]# ll passwd*
-rw-r--r-- 1 root root 1929 Feb 10 14:29 passwd.new
-rw-r--r-- 1 root root 1929 Feb 10 15:12 passwd.old <==文件一模一样！

范例三：恢复旧文件的内容
[root@www test]# patch -R -p0 < passwd.patch
[root@www test]# ll passwd*
-rw-r--r-- 1 root root 1929 Feb 10 14:29 passwd.new
-rw-r--r-- 1 root root 1986 Feb 10 15:18 passwd.old
# 文件就这样恢复成为旧版本罗
```
 

