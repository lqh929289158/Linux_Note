# Chapter 16. Routine Process

# 1. What is Routine Process(crontab)?

## 1.1 Types of routine: `at`,`cron`
- Repeated: Repeat a work periodically. `crontab`
- Once: Only do once. `at`

## 1.2 Usually-used Rountine Process
- Log rotate: Store new login info and old ones separately.
- `logwatch` analyze log file: Maybe **root** often receives mail from `logwatch`.
- Create/Update database of `locate`: `updatedb` for the file in `/var/lib/mlocate/`
- Create/Update database of `whatis`: 
- Track/Update database of **RPM**: 
- Remove cached files: `tmpwatch`
- Analyze network service: 
# 2. Routine Process only once(`at`)

## 2.1 Start **atd** and `at`

Activate **atd** service first.
```
[root@www ~]# /etc/init.d/atd restart
正在停止 atd:                          [  确定  ]
正在启动 atd:                          [  确定  ]

# 再配置一下启动时就启动这个服务，免得每次重新启动都得再来一次！
[root@www ~]# chkconfig atd on
```
### Principle of `at`

Use `at` to generate a work and add it to `/var/spool/at/` as a text file. So that the work can be scheduled and executed by the service **atd**.

For security, the white list and black list are in `/etc/at.allow` `/etc/deny`.
1. If `/etc/at.allow` exists, the users not listed in `/etc/at.allow` **can't** use `at`.
2. Else if `/etc/at.deny` exists, the users not listed in `/etc/at.deny` **can** use `at`.
3. If neither, **only root** can use `at`.

## 2.2 Execute one-time routine.

```
[root@www ~]# at [-mldv] TIME
[root@www ~]# at -c 工作号码
选项与参数：
-m  ：当 at 的工作完成后，即使没有输出信息，亦以 email 通知使用者该工作已完成。
-l  ：at -l 相当於 atq，列出目前系统上面的所有该使用者的 at 排程；
-d  ：at -d 相当於 atrm ，可以取消一个在 at 排程中的工作；
-v  ：可以使用较明显的时间格式列出 at 排程中的工作列表；
-c  ：可以列出后面接的该项工作的实际命令内容。

TIME：时间格式，这里可以定义出『什么时候要进行 at 这项工作』的时间，格式有：
  HH:MM				ex> 04:00
	在今日的 HH:MM 时刻进行，若该时刻已超过，则明天的 HH:MM 进行此工作。
  HH:MM YYYY-MM-DD		ex> 04:00 2009-03-17
	强制规定在某年某月的某一天的特殊时刻进行该工作！
  HH:MM[am|pm] [Month] [Date]	ex> 04pm March 17
	也是一样，强制在某年某月某日的某时刻进行！
  HH:MM[am|pm] + number [minutes|hours|days|weeks]
	ex> now + 5 minutes	ex> 04pm + 3 days
	就是说，在某个时间点『再加几个时间后』才进行。
```

Example:
```
范例一：再过五分钟后，将 /root/.bashrc 寄给 root 自己
[root@www ~]# at now + 5 minutes  <==记得单位要加 s 喔！
at> /bin/mail root -s "testing at job" < /root/.bashrc
at> <EOT>   <==这里输入 [ctrl] + d 就会出现 <EOF> 的字样！代表结束！
job 4 at 2009-03-14 15:38
# 上面这行资讯在说明，第 4 个 at 工作将在 2009/03/14 的 15:38 进行！
# 而运行 at 会进入所谓的 at shell 环境，让你下达多重命令等待运行！

范例二：将上述的第 4 项工作内容列出来查阅
[root@www ~]# at -c 4
#!/bin/sh               <==就是透过 bash shell 的啦！
# atrun uid=0 gid=0
# mail     root 0
umask 22
....(中间省略许多的环境变量项目)....
cd /root || {           <==可以看出，会到下达 at 时的工作目录去运行命令
         echo 'Execution directory inaccessible' >&2
         exit 1
}

/bin/mail root -s "testing at job" < /root/.bashrc
# 你可以看到命令运行的目录 (/root)，还有多个环境变量与实际的命令内容啦！

范例三：由於机房预计於 2009/03/18 停电，我想要在 2009/03/17 23:00 关机？
[root@www ~]# at 23:00 2009-03-17
at> /bin/sync
at> /bin/sync
at> /sbin/shutdown -h now
at> <EOT>
job 5 at 2009-03-17 23:00
# 您瞧瞧！ at 还可以在一个工作内输入多个命令呢！不错吧！
```

The commands will be executed in a **at shell**, so it is recommended to use **absolute** path.
> NOTE: All printed information by at shell **WILL NOT** be printed in your terminal because all results are sent to the mailbox of the executor(only when `-m` parameter is set). If you want to display them on a terminal, use `echo "Hello" > /dev/tty1`(tty1 is your terminal)

Usually used for background execution.

### `at` job management(`atrm` remove)
```
[root@www ~]# atq
[root@www ~]# atrm (jobnumber)

范例一：查询目前主机上面有多少的 at 工作排程？
[root@www ~]# atq
5       2009-03-17 23:00 a root
# 上面说的是：『在 2009/03/17 的 23:00 有一项工作，该项工作命令下达者为 
# root』而且，该项工作的工作号码 (jobnumber) 为 5 号喔！

范例二：将上述的第 5 个工作移除！
[root@www ~]# atrm 5
[root@www ~]# atq
# 没有任何资讯，表示该工作被移除了！
```
### `batch` continue job when PC is not busy

```
范例一：同样是机房停电在 2009/3/17 23:00 关机，但若当时系统负载太高，则暂缓运行
[root@www ~]# batch 23:00 2009-3-17
at> sync
at> sync
at> shutdown -h now
at> <EOT>
job 6 at 2009-03-17 23:00

[root@www ~]# atq
6       2009-03-17 23:00 b root
[root@www ~]# atrm 6
```

# 3. Routine Process loop(`crontab`)

## 3.1 Configuration of users and usage

Similar to `at`. 
- 1. If `/etc/cron.allow` exists, only users listed in the file can use `crontab`.
- 2. Else if `/etc/cron.deny` exists, the users not listed in the file can use `crontab`.
- 3. Else only **root** can use it.

- The tasks created will be put to `/var/spool/cron/[username]/`.
- All jobs executed by `cron` will be logged into `/var/log/cron`.(Check whether there are virus or troyean?)

```
[root@www ~]# crontab [-u username] [-l|-e|-r]
选项与参数：
-u  ：只有 root 才能进行这个任务，亦即帮其他使用者创建/移除 crontab 工作排程；
-e  ：编辑 crontab 的工作内容
-l  ：查阅 crontab 的工作内容
-r  ：移除所有的 crontab 的工作内容，若仅要移除一项，请用 -e 去编辑。

范例一：用 dmtsai 的身份在每天的 12:00 发信给自己
[dmtsai@www ~]$ crontab -e
# 此时会进入 vi 的编辑画面让您编辑工作！注意到，每项工作都是一行。
0   12  *  *  * mail dmtsai -s "at 12:00" < /home/dmtsai/.bashrc
#分 时 日 月 周 |<==============命令串========================>|
```
- minute: 0~59
- hour: 0~23
- date: 1~31
- month: 1-12
- week: 0-7 (both 0 and 7 are **Sunday**)
- command

- Special Symbol:
 - `*`: Any
 - `,`:  And `0 3,6 * * * cmd` 3:00 and 6:00.
 - `-`: Range `0 3-6 * * * cmd` from 3:00 to 6:00.
 - `/n`: per `*/5 * * * * cmd` per 5 minutes execute once.(Or `0-59/5 * * * * cmd`)
## 3.2 Configuration File `/etc/crontab`

The command `crontab -e` is designed for **users' cron**.
You can edit `/etc/crontab` to manage **system routine**. It is just a **text** file! It will work in 1 minutes after edition.

```
[root@www ~]# cat /etc/crontab
SHELL=/bin/bash                     <==使用哪种 shell 介面
PATH=/sbin:/bin:/usr/sbin:/usr/bin  <==运行档搜寻路径
MAILTO=root                         <==若有额外STDOUT，以 email将数据送给谁
HOME=/                              <==默认此 shell 的家目录所在

# run-parts
01  *  *  *  *   root      run-parts /etc/cron.hourly   <==每小时
02  4  *  *  *   root      run-parts /etc/cron.daily    <==每天
22  4  *  *  0   root      run-parts /etc/cron.weekly   <==每周日
42  4  1  *  *   root      run-parts /etc/cron.monthly  <==每个月 1 号
分 时 日 月 周 运行者身份  命令串
```

- `MAILTO=root`: The error information of **cron** will be sent to which mailbox.
- `PATH`: The execution path.
- The fifth column: Identification to run the routine. Default: **root**.
- (**IMPORTANT**)`run-parts /etc/cron.hourly/`: Run all scripts in path `/etc/cron.hourly` every hour.

## 3.3 NOTES
### Balance of resource allocation

If all **cron** start from the same time and repeat every five minutes, the load of CPU will be heavy for every five minutes. So, we can do this:
```
[root@www ~]# vi /etc/crontab
1,6,11,16,21,26,31,36,41,46,51,56 * * * * root  CMD1
2,7,12,17,22,27,32,37,42,47,52,57 * * * * root  CMD2
3,8,13,18,23,28,33,38,43,48,53,58 * * * * root  CMD3
4,9,14,19,24,29,34,39,44,49,54,59 * * * * root  CMD4
```
Let the routines start from different times.
### Cancel unnecessary output
Redirect the output to `/dev/null` by ` > /dev/null`

### Check security
Whether there are unexpected commands in `/var/log/cron`?

### week can't be together with (date and month)

# 4. Routine Process during power off
