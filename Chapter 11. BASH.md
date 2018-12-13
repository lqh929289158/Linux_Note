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

