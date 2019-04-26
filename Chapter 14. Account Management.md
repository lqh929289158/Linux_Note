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
