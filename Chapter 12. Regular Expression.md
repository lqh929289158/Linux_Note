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



