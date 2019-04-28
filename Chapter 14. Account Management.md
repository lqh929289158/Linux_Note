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
