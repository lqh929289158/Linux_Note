# Chapter 14. Account Management

# 1. Account and Group

## Account and Group

### UID and GID

User ID and account name stored in `/etc/passwd`

Group ID and account name stored in `/etc/group`

> WARNING: If the User ID in `/etc/passwd` is changed, the user name in file's profile will be changed to number because it can not find corresponding name in `/etc/passwd`. If we reboot the PC without change back, we can't enter `~/` because the home directory's owner ID can not be found in `/etc/passwd` and we are logging in with a new User ID now!

### UID

What first do we do before using Linux?

- First we need to log in by the terminal **tty1~tty7**. Or by **ssh** on the Internet.

- Second, find whether user name exists in `/etc/passwd`. If yes, find out **UID** and **GID**(from `/etc/group`). Also, read out `~/` and shell configuration.

- Third, check password in `/etc/shadow` by your UID.

- Last, if authentication succeeds, enter shell.

### structure of `/etc/passwd`

One account per line(including system account!).

```
[root@www ~]# head -n 4 /etc/passwd
root:x:0:0:root:/root:/bin/bash  <==等一下做为底下说明用
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
```

1. Account name. `root`
2. Password. `x` hidden because stored in `/etc/shadow`
3. UID. 
  - 0: System administor. Account with UID 0 has **root** rights.
  - 1~499: System accoutns. Actually, no different right with **root**. But they can not be logged in`/sbin/nologin`. 
    - 1~99: Accounts created by **Distributions**.
    - 100~499: Accounts available when user need.
  - 500~65535: User account. Can be logged in.
4. GID. `/etc/group` map GID to Group name.
5. User information.
6. home directory. `~/`
7. Shell. Used to specify shell environment. For example, default **BASH** environment is specified here. `/sbin/nologin` means no shell!

### structure of `/etc/shadow`

```
[root@www ~]# head -n 4 /etc/shadow
root:$1$/30QpE5e$y9N/D0bh6rAACBEz.hqo00:14126:0:99999:7:::  <==底下说明用
bin:*:14126:0:99999:7:::
daemon:*:14126:0:99999:7:::
adm:*:14126:0:99999:7:::
```
1. Account name. Must be the same as the name in `/etc/passwd`.
2. Password. Encrypted. Only read and written by **root**.
3. Last date of changing password. The days from 1970/1/1.
4. How many days the password can not be changed. Based on **#3**.
5. How many days you need to change password in.
6. How many days warning occurs before the password change deadline.
7. How many days you can still use your account before the password out of date.
8. Ouf of date.
9. Reserved.

### structure of `/etc/group`

4 columns, each line for a group.
```
[root@www ~]# head -n 4 /etc/group
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
sys:x:3:root,bin,adm
```
1. Group name.
2. Group password. Stored in `/etc/gshadow`
3. GID.
4. Account names in this group.

### Effective group and initial group

**Initial group**: The **GID** in `/etc/passwd`. The group the user is in when log in.

```
[root@www ~]# usermod -G users dmtsai  <==先配置好次要群组
[root@www ~]# grep dmtsai /etc/passwd /etc/group /etc/gshadow
/etc/passwd:dmtsai:x:503:504::/home/dmtsai:/bin/bash
/etc/group:users:x:100:dmtsai  <==次要群组的配置
/etc/group:dmtsai:x:504:       <==因为是初始群组，所以第四字段不需要填入账号
/etc/gshadow:users:::dmtsai    <==次要群组的配置
/etc/gshadow:dmtsai:!::
```

- `dmtsai` belongs to group `dmtsai` and `users`.
- `dmtsai` 's initial group is `dmtsai` with GID 504. So, no need to add account name `dmtsai` to group `dmtsai` in `/etc/group`
- `dmtsai` can **read/write/execute** files which group `dmtsai` and `users` have rights.
- When `dmtsai` creates a file, the file belongs to the **effective group** of `dmtsai`!

**Effective group**: The file belongs to the effective group of the creator.

Use `groups` to check which groups you are belonging to. The first one is **effective group**!

Use `newgrp [group name]` to switch effective group. **Group name must be the group you have already belong to!**

> NOTE: `newgrp` open a new shell with id \[user,newgroup\]. Although, the environment configuration will not change, the permission to files will be updated. And you have to `exit` to go back to the previous environment.

How to add user to groups?

- Let **root** use `usermod`.
- Let **group administrator** use `gpasswd`.

### `/etc/gshadow`

Main function: Specify group administrator to manage group members.

```
[root@www ~]# head -n 4 /etc/gshadow
root:::root
bin:::root,bin,daemon
daemon:::root,bin,daemon
sys:::root,bin,adm
```

1. Group name.
2. Password. `!` means no valid password(no group administrator).
3. Name of group administrator. (Details in `gpasswd`)
4. Group members.(The same as `/etc/group`)

> NOTE: Since we have convenient tools like `sudo`, group administrator is rarely used now.

# 2. Account Management

## Add and delete user

### `useradd`

```
[root@www ~]# useradd [-u UID] [-g 初始群组] [-G 次要群组] [-mM]\
>  [-c 说明栏] [-d 家目录绝对路径] [-s shell] 使用者账号名
选项与参数：
-u  ：后面接的是 UID ，是一组数字。直接指定一个特定的 UID 给这个账号；
-g  ：后面接的那个组名就是我们上面提到的 initial group 啦～
      该群组的 GID 会被放置到 /etc/passwd 的第四个字段内。
-G  ：后面接的组名则是这个账号还可以加入的群组。
      这个选项与参数会修改 /etc/group 内的相关数据喔！
-M  ：强制！不要创建用户家目录！(系统账号默认值)
-m  ：强制！要创建用户家目录！(一般账号默认值)
-c  ：这个就是 /etc/passwd 的第五栏的说明内容啦～可以随便我们配置的啦～
-d  ：指定某个目录成为家目录，而不要使用默认值。务必使用绝对路径！
-r  ：创建一个系统的账号，这个账号的 UID 会有限制 (参考 /etc/login.defs)
-s  ：后面接一个 shell ，若没有指定则默认是 /bin/bash 的啦～
-e  ：后面接一个日期，格式为『YYYY-MM-DD』此项目可写入 shadow 第八字段，
      亦即账号失效日的配置项目啰；
-f  ：后面接 shadow 的第七字段项目，指定口令是否会失效。0为立刻失效，
      -1 为永远不失效(口令只会过期而强制于登陆时重新配置而已。)

范例一：完全参考默认值创建一个用户，名称为 vbird1
[root@www ~]# useradd vbird1
[root@www ~]# ll -d /home/vbird1
drwx------ 4 vbird1 vbird1 4096 Feb 25 09:38 /home/vbird1
# 默认会创建用户家目录，且权限为 700 ！这是重点！

[root@www ~]# grep vbird1 /etc/passwd /etc/shadow /etc/group
/etc/passwd:vbird1:x:504:505::/home/vbird1:/bin/bash
/etc/shadow:vbird1:!!:14300:0:99999:7:::
/etc/group:vbird1:x:505:    <==默认会创建一个与账号一模一样的群组名
```
What the `useradd [Account name]` will do?
- `/etc/passwd`: A new line with **UID/GID/Home Directory**.
- `/etc/shadow`: A new line without password.
- `/etc/group`: A new group name same with the account name by default.
- `/etc/gshadow`: A line for the new group without password by default.
- `/home`: A new directory with the same name with account name. Access control: **700**.

What the `useradd` will refer to?(Default configuration)
- `/etc/default/useradd`
- `/etc/login.defs`
- `/etc/skel/*`

### `useradd` manual

Show the default configuration of `useradd`(`/etc/default/useradd`).
```
[root@www ~]# useradd -D
GROUP=100		<==默认的群组
HOME=/home		<==默认的家目录所在目录
INACTIVE=-1		<==口令失效日，在 shadow 内的第 7 栏
EXPIRE=			<==账号失效日，在 shadow 内的第 8 栏
SHELL=/bin/bash		<==默认的 shell
SKEL=/etc/skel		<==用户家目录的内容数据参考目录
CREATE_MAIL_SPOOL=yes   <==是否主动帮使用者创建邮件信箱(mailbox)
```

- `GROUP=100`. The GID of **default group**.
  - **Private group strategy**. Each account has its own group and home directory with **700** access control. Example distributions: RHEL, Fedora, **CentOS**.
  - Public group strategy. Each account belongs to group **users** and home directory with **755** access control. Example distributions: SuSE.
- `HOME=/home`: Base directory of home directory.
- `INACTIVE=-1`: The password will be invalid or not after out of date.
  - 0: Invalid soon after due.
  - -1: Never invalid.
  - n: Valid in n days after due.
- `EXPIRE=`: Date of due.
- `SHELL=/bin/bash`: Default program name of shell.
  - `/bin/bash`: Normal.
  - `/sbin/nologin`: The new user can not login but he can only use email function.(Mail server)
- `SKEL=/etc/skel`: Reference directory of home directory. All files in `SKEL` will be copied to the new created home directory.
- `CREATE_MAIL_SPOOL=yes`: Create mailbox for the user or not. Check in `/var/spool/mail/`

### `/etc/login.defs`
```

MAIL_DIR        /var/spool/mail	<==用户默认邮件信箱放置目录

PASS_MAX_DAYS   99999	<==/etc/shadow 内的第 5 栏，多久需变更口令日数
PASS_MIN_DAYS   0	<==/etc/shadow 内的第 4 栏，多久不可重新配置口令日数
PASS_MIN_LEN    5	<==口令最短的字符长度，已被 pam 模块取代，失去效用！
PASS_WARN_AGE   7	<==/etc/shadow 内的第 6 栏，过期前会警告的日数

UID_MIN         500	<==使用者最小的 UID，意即小于 500 的 UID 为系统保留
UID_MAX       60000	<==使用者能够用的最大 UID
GID_MIN         500	<==使用者自定义组的最小 GID，小于 500 为系统保留
GID_MAX       60000	<==使用者自定义组的最大 GID

CREATE_HOME     yes	<==在不加 -M 及 -m 时，是否主动创建用户家目录？
UMASK           077     <==用户家目录创建的 umask ，因此权限会是 700
USERGROUPS_ENAB yes     <==使用 userdel 删除时，是否会删除初始群组
MD5_CRYPT_ENAB yes      <==口令是否经过 MD5 的加密机制处理
```

### `passwd` configure password

```
[root@www ~]# passwd [--stdin]  <==所有人均可使用来改自己的口令
[root@www ~]# passwd [-l] [-u] [--stdin] [-S] \
>  [-n 日数] [-x 日数] [-w 日数] [-i 日期] 账号 <==root 功能
选项与参数：
--stdin ：可以透过来自前一个管线的数据，作为口令输入，对 shell script 有帮助！
-l  ：是 Lock 的意思，会将 /etc/shadow 第二栏最前面加上 ! 使口令失效；
-u  ：与 -l 相对，是 Unlock 的意思！
-S  ：列出口令相关参数，亦即 shadow 文件内的大部分信息。
-n  ：后面接天数，shadow 的第 4 字段，多久不可修改口令天数
-x  ：后面接天数，shadow 的第 5 字段，多久内必须要更动口令
-w  ：后面接天数，shadow 的第 6 字段，口令过期前的警告天数
-i  ：后面接『日期』，shadow 的第 7 字段，口令失效日期

范例一：请 root 给予 vbird2 口令
[root@www ~]# passwd vbird2
Changing password for user vbird2.
New UNIX password: <==这里直接输入新的口令，屏幕不会有任何反应
BAD PASSWORD: it is WAY too short <==口令太简单或过短的错误！
Retype new UNIX password:  <==再输入一次同样的口令
passwd: all authentication tokens updated successfully.  <==竟然还是成功修改了!
```

> WARNING: When you are **root**, use `passwd [account]` to configure password for others and `passwd` for itself. When you are a normal user, use `passwd` to configure password for yourself!

### `chage`

```
[root@www ~]# chage [-ldEImMW] 账号名
选项与参数：
-l ：列出该账号的详细口令参数；
-d ：后面接日期，修改 shadow 第三字段(最近一次更改口令的日期)，格式 YYYY-MM-DD
-E ：后面接日期，修改 shadow 第八字段(账号失效日)，格式 YYYY-MM-DD
-I ：后面接天数，修改 shadow 第七字段(口令失效日期)
-m ：后面接天数，修改 shadow 第四字段(口令最短保留天数)
-M ：后面接天数，修改 shadow 第五字段(口令多久需要进行变更)
-W ：后面接天数，修改 shadow 第六字段(口令过期前警告日期)

范例一：列出 vbird2 的详细口令参数
[root@www ~]# chage -l vbird2
Last password change                               : Feb 26, 2009
Password expires                                   : Apr 27, 2009
Password inactive                                  : May 07, 2009
Account expires                                    : never
Minimum number of days between password change     : 0
Maximum number of days between password change     : 60
Number of days of warning before password expires  : 7
```
### `usermod`

```

[root@www ~]# usermod [-cdegGlsuLU] username
选项与参数：
-c  ：后面接账号的说明，即 /etc/passwd 第五栏的说明栏，可以加入一些账号的说明。
-d  ：后面接账号的家目录，即修改 /etc/passwd 的第六栏；
-e  ：后面接日期，格式是 YYYY-MM-DD 也就是在 /etc/shadow 内的第八个字段数据啦！
-f  ：后面接天数，为 shadow 的第七字段。
-g  ：后面接初始群组，修改 /etc/passwd 的第四个字段，亦即是 GID 的字段！
-G  ：后面接次要群组，修改这个使用者能够支持的群组，修改的是 /etc/group 啰～
-a  ：与 -G 合用，可『添加次要群组的支持』而非『配置』喔！
-l  ：后面接账号名称。亦即是修改账号名称， /etc/passwd 的第一栏！
-s  ：后面接 Shell 的实际文件，例如 /bin/bash 或 /bin/csh 等等。
-u  ：后面接 UID 数字啦！即 /etc/passwd 第三栏的数据；
-L  ：暂时将用户的口令冻结，让他无法登陆。其实仅改 /etc/shadow 的口令栏。
-U  ：将 /etc/shadow 口令栏的 ! 拿掉，解冻啦！
```

### `userdel`

What will it delete?

- Entry in `/etc/passwd` `/etc/shadow`
- Entry in `/etc/group` `/etc/gshadow`
- Data in `/home/username` `/var/spool/mail/username` ...

```
[root@www ~]# userdel [-r] username
选项与参数：
-r  ：连同用户的家目录也一起删除

范例一：删除 vbird2 ，连同家目录一起删除
[root@www ~]# userdel -r vbird2
```

> WARNING: You can just disable an account by setting the expire date to **0** in `/etc/shadow`. And all data will be kept in this case. If you really want to delete all data of an account, you are recommened to use `find / -user [username]` to confirm all files you will delete.

## User function

### `finger`

```
[root@www ~]# finger [-s] username
选项与参数：
-s  ：仅列出用户的账号、全名、终端机代号与登陆时间等等；
-m  ：列出与后面接的账号相同者，而不是利用部分比对 (包括全名部分)

范例一：观察 vbird1 的用户相关账号属性
[root@www ~]# finger vbird1
Login: vbird1                           Name: (null)
Directory: /home/vbird1                 Shell: /bin/bash
Never logged in.
No mail.
No Plan.
```

- Login: Account name. (1st part in `passwd`)
- Name: Full name. (5th part in `passwd`)
- Directory: Home.
- Shell: Shell program directory.
- Never logged in: 
- No mail: `/var/spool/mail`. 
- No plan: `~vbird1/.plan`

```
范例二：利用 vbird1 创建自己的计划档
[vbird1@www ~]$ echo "I will study Linux during this year." > ~/.plan
[vbird1@www ~]$ finger vbird1
Login: vbird1                           Name: (null)
Directory: /home/vbird1                 Shell: /bin/bash
Never logged in.
No mail.
Plan:
I will study Linux during this year.

范例三：找出目前在系统上面登陆的用户与登陆时间
[vbird1@www ~]$ finger
Login     Name       Tty      Idle  Login Time   Office     Office Phone
root      root       tty1           Feb 26 09:53
vbird1               tty2           Feb 26 15:21
```

### `chfn` modify finger

### `chsh` modify shell

```

[vbird1@www ~]$ chsh [-ls]
选项与参数：
-l  ：列出目前系统上面可用的 shell ，其实就是 /etc/shells 的内容！
-s  ：配置修改自己的 Shell 啰

范例一：用 vbird1 的身份列出系统上所有合法的 shell，并且指定 csh 为自己的 shell
[vbird1@www ~]$ chsh -l
/bin/sh
/bin/bash
/sbin/nologin  <==所谓：合法不可登陆的 Shell 就是这玩意！
/bin/tcsh
/bin/csh       <==这就是 C shell 啦！
/bin/ksh
# 其实上面的信息就是我们在 bash 中谈到的 /etc/shells 啦！

[vbird1@www ~]$ chsh -s /bin/csh; grep vbird1 /etc/passwd
Changing shell for vbird1.
Password:  <==确认身份，请输入 vbird1 的口令
Shell changed.
vbird1:x:504:505:VBird Tsai test,Dic in Ksu. Tainan,06-2727175#356,06-1234567:
/home/vbird1:/bin/csh

[vbird1@www ~]$ chsh -s /bin/bash
# 测试完毕后，立刻改回来！

[vbird1@www ~]$ ll $(which chsh)
-rws--x--x 1 root root 19128 May 25  2008 /usr/bin/chsh
```

### `id` mainly for UID/GID

```

[root@www ~]# id [username]

范例一：查阅 root 自己的相关 ID 信息！
[root@www ~]# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),
10(wheel) context=root:system_r:unconfined_t:SystemLow-SystemHigh
# 上面信息其实是同一行的数据！包括会显示 UID/GID 以及支持的所有群组！
# 至于后面那个 context=... 则是 SELinux 的内容，先不要理会他！

范例二：查阅一下 vbird1 吧～
[root@www ~]# id vbird1
uid=504(vbird1) gid=505(vbird1) groups=505(vbird1) context=root:system_r:
unconfined_t:SystemLow-SystemHigh

[root@www ~]# id vbird100
id: vbird100: No such user  <== id 这个命令也可以用来判断系统上面有无某账号！
```

## Add and delete group

### `groupadd`

```
[root@www ~]# groupadd [-g gid] [-r] 组名
选项与参数：
-g  ：后面接某个特定的 GID ，用来直接给予某个 GID ～
-r  ：创建系统群组啦！与 /etc/login.defs 内的 GID_MIN 有关。

范例一：新建一个群组，名称为 group1
[root@www ~]# groupadd group1
[root@www ~]# grep group1 /etc/group /etc/gshadow
/etc/group:group1:x:702:
/etc/gshadow:group1:!::
# 群组的 GID 也是会由 500 以上最大 GID+1 来决定！
```

> NOTE: It is recommended to use `groupadd -r [group name]` to create groups nothing to do with private groups.

### `groupmod`

```
[root@www ~]# groupmod [-g gid] [-n group_name] 群组名
选项与参数：
-g  ：修改既有的 GID 数字；
-n  ：修改既有的组名

范例一：将刚刚上个命令创建的 group1 名称改为 mygroup ， GID 为 201
[root@www ~]# groupmod -g 201 -n mygroup group1
[root@www ~]# grep mygroup /etc/group /etc/gshadow
/etc/group:mygroup:x:201:
/etc/gshadow:mygroup:!::
```

### `groupdel`

```
[root@www ~]# groupdel [groupname]

范例一：将刚刚的 mygroup 删除！
[root@www ~]# groupdel mygroup

范例二：若要删除 vbird1 这个群组的话？
[root@www ~]# groupdel vbird1
groupdel: cannot remove user's primary group.
```
You can not delete a group which is someone's **initial group**!

So, you can

- modify GID of `vbird1`
- delete account `vbird1`

### `gpasswd`

```
# 关于系统管理员(root)做的动作：
[root@www ~]# gpasswd groupname
[root@www ~]# gpasswd [-A user1,...] [-M user3,...] groupname
[root@www ~]# gpasswd [-rR] groupname
选项与参数：
    ：若没有任何参数时，表示给予 groupname 一个口令(/etc/gshadow)
-A  ：将 groupname 的主控权交由后面的使用者管理(该群组的管理员)
-M  ：将某些账号加入这个群组当中！
-r  ：将 groupname 的口令移除
-R  ：让 groupname 的口令栏失效

# 关于群组管理员(Group administrator)做的动作：
[someone@www ~]$ gpasswd [-ad] user groupname
选项与参数：
-a  ：将某位使用者加入到 groupname 这个群组当中！
-d  ：将某位使用者移除出 groupname 这个群组当中。

范例一：创建一个新群组，名称为 testgroup 且群组交由 vbird1 管理：
[root@www ~]# groupadd testgroup  <==先创建群组
[root@www ~]# gpasswd testgroup   <==给这个群组一个口令吧！
Changing the password for group testgroup
New Password:
Re-enter new password:
# 输入两次口令就对了！
[root@www ~]# gpasswd -A vbird1 testgroup  <==加入群组管理员为 vbird1
[root@www ~]# grep testgroup /etc/group /etc/gshadow
/etc/group:testgroup:x:702:
/etc/gshadow:testgroup:$1$I5ukIY1.$o5fmW.cOsc8.K.FHAFLWg0:vbird1:
# 很有趣吧！此时 vbird1 则拥有 testgroup 的主控权喔！身份有点像板主啦！

范例二：以 vbird1 登陆系统，并且让他加入 vbird1, vbird3 成为 testgroup 成员
[vbird1@www ~]$ id
uid=504(vbird1) gid=505(vbird1) groups=505(vbird1) ....
# 看得出来，vbird1 尚未加入 testgroup 群组喔！

[vbird1@www ~]$ gpasswd -a vbird1 testgroup
[vbird1@www ~]$ gpasswd -a vbird3 testgroup
[vbird1@www ~]$ grep testgroup /etc/group
testgroup:x:702:vbird1,vbird3
```

# Linux Account Management and ACL Configuration

ACL: Access Control List. Configure access control for a specified group or user.
