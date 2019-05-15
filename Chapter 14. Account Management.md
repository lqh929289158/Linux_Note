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

# 3. Linux Account Management and ACL Configuration

ACL: Access Control List. Configure access control for a specified group or user.

# 4. Switch Account

Why switching account?
- Use ordinary account.
- Start a system service with a lower right.
- Limitations of software.

How to switch?
- `su -` needs password of **root**.
- `sudo [cmd]` needs user's own password.(Need to be configured in advance.)

## `su`

```
[root@www ~]# su [-lm] [-c 命令] [username]
选项与参数：
-   ：单纯使用 - 如『 su - 』代表使用 login-shell 的变量文件读取方式来登陆系统；
      若使用者名称没有加上去，则代表切换为 root 的身份。
-l  ：与 - 类似，但后面需要加欲切换的使用者账号！也是 login-shell 的方式。
-m  ：-m 与 -p 是一样的，表示『使用目前的环境配置，而不读取新使用者的配置文件』
-c  ：仅进行一次命令，所以 -c 后面可以加上命令喔！
```

### `su` non-login-shell
```
范例一：假设你原本是 vbird1 的身份，想要使用 non-login shell 的方式变成 root
[vbird1@www ~]$ su       <==注意提示字符，是 vbird1 的身份喔！
Password:                <==这里输入 root 的口令喔！
[root@www vbird1]# id    <==提示字符的目录是 vbird1 喔！
uid=0(root) gid=0(root) groups=0(root),1(bin),...   <==确实是 root 的身份！
[root@www vbird1]# env | grep 'vbird1'
USER=vbird1
PATH=/usr/local/bin:/bin:/usr/bin:/home/vbird1/bin  <==这个影响最大！
MAIL=/var/spool/mail/vbird1                         <==收到的 mailbox 是 vbird1
PWD=/home/vbird1                                    <==并非 root 的家目录
LOGNAME=vbird1
# 虽然你的 UID 已经是具有 root 的身份，但是看到上面的输出信息吗？
# 还是有一堆变量为原本 vbird1 的身份，所以很多数据还是无法直接利用。
[root@www vbird1]# exit   <==这样可以离开 su 的环境！
```

### `su -` login shell

```
范例二：使用 login shell 的方式切换为 root 的身份并观察变量
[vbird1@www ~]$ su -
Password:   <==这里输入 root 的口令喔！
[root@www ~]# env | grep root
USER=root
MAIL=/var/spool/mail/root
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
PWD=/root
HOME=/root
LOGNAME=root
# 了解差异了吧？下次变换成为 root 时，记得最好使用 su - 喔！
[root@www ~]# exit   <==这样可以离开 su 的环境！
```

### switch to other account

`su - [username]` or `su -l [username]`. That will ensure that **PATH**, **MAIL**, **USER** etc. environment variables will also switch to new environment.

## `sudo`

### `sudo`

```
[root@www ~]# sudo [-b] [-u 新使用者账号]
选项与参数：
-b  ：将后续的命令放到背景中让系统自行运行，而不与目前的 shell 产生影响
-u  ：后面可以接欲切换的使用者，若无此项则代表切换身份为 root 。

范例一：你想要以 sshd 的身份在 /tmp 底下创建一个名为 mysshd 的文件
[root@www ~]# sudo -u sshd touch /tmp/mysshd
[root@www ~]# ll /tmp/mysshd
-rw-r--r-- 1 sshd sshd 0 Feb 28 17:42 /tmp/mysshd
# 特别留意，这个文件的权限是由 sshd 所创建的情况喔！

范例二：你想要以 vbird1 的身份创建 ~vbird1/www 并于其中创建 index.html 文件
[root@www ~]# sudo -u vbird1 sh -c "mkdir ~vbird1/www; cd ~vbird1/www; \
>  echo 'This is index.html file' > index.html"
[root@www ~]# ll -a ~vbird1/www
drwxr-xr-x 2 vbird1 vbird1 4096 Feb 28 17:51 .
drwx------ 5 vbird1 vbird1 4096 Feb 28 17:51 ..
-rw-r--r-- 1 vbird1 vbird1   24 Feb 28 17:51 index.html
# 要注意，创建者的身份是 vbird1 ，且我们使用 sh -c "一串命令" 来运行的！
```

For default, only **root** can use **sudo**.
1. When user execute `sudo`, system will refer `/etc/sudoers`.
2. If the user has rights for `sudo`, then require its password.
3. If password OK, then run commands with ID of root or someone else.
4. If target user ID is the same with current one, no need for password.

### visudo and `/etc/sudoers`

You have to configure `/etc/sudoers` as **root** if you want someone to have the access to `sudo`.

Why **visudo** ? Because `/etc/sudoers` has its own grammar, **visudo** will check grammar after you modify it.

1. An account can execute **all** commands as **root**.
```
[username] [host name where you from]=([account that can switch to])  [commands allowed]
```

```
[root@www ~]# visudo
....(前面省略)....
root    ALL=(ALL)       ALL  <==找到这一行，大约在 76 行左右
vbird1  ALL=(ALL)       ALL  <==这一行是你要新增的！
....(前面省略)....
```

2. Configure group and no-password

```
[root@www ~]# visudo  <==同样的，请使用 root 先配置
....(前面省略)....
%wheel     ALL=(ALL)    ALL <==大约在 84 行左右，请将这行的 # 拿掉！
# 在最左边加上 % ，代表后面接的是一个『群组』之意！改完请储存后离开

[root@www ~]# usermod -a -G wheel pro1 <==将 pro1 加入 wheel 的支持
```
So that everyone in group `wheel` can access `sudo`.

```
[root@www ~]# visudo  <==同样的，请使用 root 先配置
....(前面省略)....
%wheel     ALL=(ALL)   NOPASSWD: ALL <==大约在 87 行左右，请将 # 拿掉！
# 在最左边加上 % ，代表后面接的是一个『群组』之意！改完请储存后离开
```
So that everyone in group `wheel` can access `sudo` without entering their own password.

3. Limited commands

```
[root@www ~]# visudo  <==注意是 root 身份
myuser1	ALL=(root)  /usr/bin/passwd  <==最后命令务必用绝对路径
```
> NOTE: Must use absolute path!
> WARNING: The configuration above is problemistic because `myuser1` can modify the password of `root`!

```
[myuser1@www ~]$ sudo passwd myuser3  <==注意，身份是 myuser1
Password:  <==输入 myuser1 的口令
Changing password for user myuser3. <==底下改的是 myuser3 的口令喔！这样是正确的
New UNIX password:
Retype new UNIX password:
passwd: all authentication tokens updated successfully.

[myuser1@www ~]$ sudo passwd
Changing password for user root.  <==见鬼！怎么会去改 root 的口令？
```

```
[root@www ~]# visudo  <==注意是 root 身份
myuser1	ALL=(root)  !/usr/bin/passwd, /usr/bin/passwd [A-Za-z]*, \
                    !/usr/bin/passwd root
```

- Ban `sudo passwd`.
- Ban `sudo passwd root`.
- Allow the others.

4. Configure **visudo** as alias

- `User_Alias`
- `Cmnd_Alias`

```
[root@www ~]# visudo  <==注意是 root 身份
User_Alias ADMPW = pro1, pro2, pro3, myuser1, myuser2
Cmnd_Alias ADMPWCOM = !/usr/bin/passwd, /usr/bin/passwd [A-Za-z]*, \
                      !/usr/bin/passwd root
ADMPW   ALL=(root)  ADMPWCOM
```
5. Interval

Probably, you have to enter your password again after 5 minutes withou action.

6. `sudo`+`su`

```
[root@www ~]# visudo
User_Alias  ADMINS = pro1, pro2, pro3, myuser1
ADMINS ALL=(root)  /bin/su -
```
So that, the users can switch to root by `sudo su -` with their own passwords. Thus, the password of **root** will be secret.

> NOTE: The users who can switch to **root** should be the ones you can believe in. 

# 5. Special Shell and PAM module.

## Special Shell `/sbin/nologin`

The accounts(including system accounts) configured to `/sbin/nologin` shell have no access to **bash** or other shells. But they can access parts of the resources.

For example, the Linux host providing mail service, almost all accounts on the host, just receive and send mails.

### `/etc/nologin.txt`

The information shown when we can not log in to an account.

```
[root@www ~]# su - myuser3
This account is system account or mail account.
Please DO NOT use this account to login my Linux server.
```

## PAM(Pluggable Authentication Modules) Module

The module you can call to authentication whatever program you are using.

The procedure after you execute `passwd` which calls PAM:
1. User runs `/user/bin/passwd` and enter password.
2. `passwd` call PAM for authentication.
3. PAM search a configuration file called `passwd` in path `/etc/pam.d/`.
4. According to configuration, import related PAM module and analyize.
5. Return authentication results to `passwd`.
6. `passwd` acts depending on the return values.

### Configuration Grammar

```
[root@www ~]# cat /etc/pam.d/passwd
#%PAM-1.0  <==PAM版本的说明而已！
auth       include      system-auth <==每一行都是一个验证的过程
account    include      system-auth
password   include      system-auth
验证类别   控制标准     PAM 模块与该模块的参数
```
- Type(Typically in sequence):
  - **auth**: authenticate the identification of user.
  - **account**: Authorization. Wheter have rights for something.
  - **session**: ...
  - **passowrd**: Modification of password.
- Control Flag:
  - **required**: return success value if authentication succeed, and vice versa.
  - **requisite**: return failure value immediately if authentication fails, and skip the authentication followed.
  - **sufficient**: return success value immediately if authentication succeed, and skip the authentication followed.
  - **optional**: Just for showing info, not for authentication.
  - **inlcude**: refer to the module file(the 3rd parameter).
- Usually-used module:
```
[root@www ~]# cat /etc/pam.d/login
#%PAM-1.0
auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_securetty.so
auth       include      system-auth
account    required     pam_nologin.so
account    include      system-auth
password   include      system-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    include      system-auth
session    required     pam_loginuid.so
session    optional     pam_console.so
# pam_selinux.so open should only be followed by sessions...
session    required     pam_selinux.so open
session    optional     pam_keyinit.so force revoke
# 我们可以看到，其实 login 也呼叫多次的 system-auth ，所以底下列出该配置文件

[root@www ~]# cat /etc/pam.d/system-auth
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth     required     pam_env.so
auth     sufficient   pam_unix.so nullok try_first_pass
auth     requisite    pam_succeed_if.so uid >= 500 quiet
auth     required     pam_deny.so

account  required     pam_unix.so
account  sufficient   pam_succeed_if.so uid < 500 quiet
account  required     pam_permit.so

password requisite    pam_cracklib.so try_first_pass retry=3
password sufficient   pam_unix.so md5 shadow nullok try_first_pass use_authtok
password required     pam_deny.so

session  optional     pam_keyinit.so revoke
session  required     pam_limits.so
session  [success=1 default=ignore] pam_succeed_if.so service in crond quiet \
                      use_uid
session  required     pam_unix.so
```
  - `/etc/pam.d/*`: PAM configuration for each program.
  - `/lib/security/*`: The path where PAM configuration files are stored.
  - `/etc/security/*`: Other PAM configuration files.
  - `/usr/share/doc/pam-*/`: Detailed PAM info file.

- `pam_securetty.so`: Limit root to login only from secure terminal.
- `pam_nologin.so`: Limit ordinary user not to login host.
- `pam_selinux.so`: 
- `pam_console.so`:
- `pam_loginuid.so`:
- `pam_env.so`: Module for configure environment variables
- `pam_unix.so`: Complex but frequently used.
- `pam_cracklib.so`: Check the security level of password. Including the max number of try, password in dictionary or not.
- `pam_limits.so`: 

The procedure of login.
1. Authentication: 
  - According to `pam_securetty.so`, if user is root, refer to `/etc/securetty`.
  - Configure extra environment variables by `pam_env.so`. 
  - Check password by `pam_unix.so`. If OK, return to 'login' program, else
  - Check UID by `pam_succeed_if.so`. If UID < 500,  return failure, else
  - Reject connection by `pam_deny.so`.
2. Account:
  - Judge `/etc/nologin` exist or not by `pam_nologin.so`. If exists, reject login of ordinary users.
  - Manage account by `pam_unix`.
  - Check UID by `pam_succeed_if.so`. If UID < 500, not log info.
  - Permit login by `pam_permit.so`
3. Password:
  - Configure max try to 3 by `pam_cracklib.so`.
  - Use **md5**,**shadow** etc. functions to check password by `pam_unix.so`. If OK, return value to login, else
  - Reject login by `pam_deny.so`.
4. Session:
  - Turn off **SELinux** temporarily by `pam_selinux.so`.
  - Configure system resources available for the user by `pam_limits.so`
  - Log related info into login files.
  - Configure access control according to UID by `pam_loginuid.so`.
  - Turn on **SELinux**.
> NOTE: WHY can't login system as root by `telnet` but OK by `ssh`? `telnet` uses dynamic terminal `pts/n` which is not written into `/etc/securetyy`.But `telnet` login is limited by `/etc/securetty`. So **root** can not login remote sysytem by `telnet`. `ssh` uses the module `/etc/pam.d/sshd` which is not written to `/etc/securetty` too. But `ssh` login is not limited by `/etc/securetyy`!

### `limits.conf`

```
范例一：vbird1 这个用户只能创建 100MB 的文件，且大于 90MB 会警告
[root@www ~]# vi /etc/security/limits.conf
vbird1	soft		fsize		 90000
vbird1	hard		fsize		100000
#账号   限制依据	限制项目 	限制值
# 第一字段为账号，或者是群组！若为群组则前面需要加上 @ ，例如 @projecta
# 第二字段为限制的依据，是严格(hard)，还是仅为警告(soft)；
# 第三字段为相关限制，此例中限制文件容量，
# 第四字段为限制的值，在此例中单位为 KB。
# 若以 vbird1 登陆后，进行如下的操作则会有相关的限制出现！

[vbird1@www ~]$ ulimit -a
....(前面省略)....
file size               (blocks, -f) 90000
....(后面省略)....

[vbird1@www ~]$ dd if=/dev/zero of=test bs=1M count=110
File size limit exceeded
[vbird1@www ~]$ ll -k test
-rw-rw-r-- 1 vbird1 vbird1 90000 Mar  4 11:30 test
# 果然有限制到了

范例二：限制 pro1 这个群组，每次仅能有一个用户登陆系统 (maxlogins)
[root@www ~]# vi /etc/security/limits.conf
@pro1   hard   maxlogins   1
# 如果要使用群组功能的话，这个功能似乎对初始群组才有效喔！
# 而如果你尝试多个 pro1 的登陆时，第二个以后就无法登陆了。
# 而且在 /var/log/secure 文件中还会出现如下的信息：
# pam_limits(login:session): Too many logins (max 1) for pro1
```
The file is called dynamically, so you do not have to restart any service after modifying it.

### `/var/log/secure` and `/var/log/messages`

Login failure and unexpected errors will be logged in these files.

# 6. Message between users.

## `w`,`who`,`last`,`lastlog` inquire users

Review `id` and `finger` to list infos about a user.

- `w`: list all login users.
- `who`: list all login users only with TTY and login time.
- `lastlog`: list all account recently logged in and login time.

## Talk between users `write`,`mesg`,`wall`

### `write`

```
[root@www ~]# write 使用者账号 [用户所在终端接口]

[root@www ~]# who
root     pts/1    2009-03-04 11:04 (192.168.1.100)
vbird1   pts/2    2009-03-04 13:15 (192.168.1.100)  <==有看到 vbird1 在在线

[root@www ~]# write vbird1 pts/2
Hello, there:
Please don't do anything wrong...  <==这两行是 root 写的信息！
# 结束时，请按下 [crtl]-d 来结束输入。此时在 vbird1 的画面中，会出现：

Message from root@www.vbird.tsai on pts/1 at 13:23 ...
Hello, there:
Please don't do anything wrong...
EOF
```

- User can refuse message by `mesg n`.
- User can check the state of itself by `mesg`.
- But user can not refuse message from **root**.
- **root** can refuse message from anyone.
- Use `mesg y` to allow message.

### `wall` to broadcast message.
```
[root@www ~]# wall "I will shutdown my linux server..."
```

## Mailbox `mail` to send message offline.
Problem: `wall`,`write` only used for **online** users!

So, use `mail` to send message to offline users.

Usage: `mail username@host -s "[Title]"`

```
[root@www ~]# mail vbird1 -s "nice to meet you"
Hello, D.M. Tsai
Nice to meet you in the network.
You are so nice.  byebye!
.    <==这里很重要喔，结束时，最后一行输入小数点 . 即可！
Cc:  <==这里是所谓的『副本』，不需要寄给其他人，所以直接 [Enter]
[root@www ~]#  <==出现提示字符，表示输入完毕了！
```

- When `host` is `localhost`, it can be omitted.
- If you want to deliver a message from a file. Use redirection `<`.
```
mail username@host -s "[Title]" < [File name]
```

How to read mail from mailbox?

The meaning of commands after entering `mail`:
- `h [n]`: list first `[n]` titles.
- `d[n1-n2]`: delete `[n1]` to `[n2]` mails in mailbox.
- `s [n] [path]`: save the `[n]`th mail as a file in `[path]`.
- `x` or `exit`: leave mailbox without saving.
- `q`: 1. Remove the mails deleted just now. 2. Move the read mails to `~/mbox` and remove them from mailbox. 3.Leave

> NOTE: `/var/spool/mail/username` is the mailbox to store new messages. `/home/[username]/mbox` is the mail box to store having been read messages. Use `mail -f /home/[username]/mbox` to check mails.








