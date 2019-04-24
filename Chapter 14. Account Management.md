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

