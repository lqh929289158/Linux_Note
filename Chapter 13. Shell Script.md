# Chapter 13. Shell Script.md

## 1. What is Shell script?

- Commands are read from up to down and from left to right.
- \[space\]\[tab\].etc will be ignored.
- Null line will be ignored too.
- Execute the command(a line or a series of) when a \[Enter\] was read.
- Use `\` to escape to next line.(The two lines are still one command.)
- Use `#` to comment.(For a line)
- Use `bash shell.sh` or `sh shell.sh` or `./shell.sh` to execute script.

### Write the 1st script

```
[root@www ~]# mkdir scripts; cd scripts
[root@www scripts]# vi sh01.sh
#!/bin/bash
# Program:
#       This program shows "Hello World!" in your screen.
# History:
# 2005/08/23	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "Hello World! \a \n"
exit 0
```

- Line 1: `#!/bin/bash` declares the shell used by this script.
- Line 2~4: Comments
  - 1. Contents and functions
  - 2. Version
  - 3. Author and contact
  - 4. Date
  - 5. History record
- Line 5~6: Environment variable declaration. Common variable `PATH` and `LANG`. So that you have no need to write absolute path.
- Line 7: `echo` command to print contents on screen.
- Line 8: return value.

> NOTE: Return value will be passed to a variable called `$?`. So if you call `echo $?` after executing the script, then you will see the return value.

Execute the script by
```
[root@www scripts]# sh sh01.sh
Hello World !
```
Or
```
[root@www scripts]# chmod a+x sh01.sh; ./sh01.sh
```

