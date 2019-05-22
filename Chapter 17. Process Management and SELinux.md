# Chapter 17. Process Management and SELinux

# 1. What is Process?

In Linux, whenever an event is triggered, it will be defined as a process, and assign a **PID** to it. At the same time, the **authorities** will be bounded to the process according to the its **owner** and related configuration. Then, what the process can do is limited by its **autorities**.

## 1.1 Process and Program

Program: A binary **file** stored in storage device.
Process: A executing program loaded to **main memory** with a label **PID** and related **authorites, data, code** and so on.

The same program triggered by different users will become processes with different authorities.

### Child and Parent process

Child Process: process triggered by another process(Parent process).
Parent Process: A process which triggered another process(Child process).

Use `ps -l` to list all process.
The PID and PPID(Parent PID) will be listed.

### `fork` and `exec` the procedure of process call
We call Parent Process A and Chile Process B.
1. A(PID:x,Name:zzz)  `fork`-> TMP(PID:y, PPID:x, NAME:zzz)
2. A `exec qqq`-> B(PID:y, PPID:x, NAME:qqq)

### System/Network Service: Resident Process

The services always executing in the memory are called resident process, such as **crond**(scanning `/etc/crontab` per minute). They are also called **daemon**.

## 1.2 Multiple-User Multiple-Job Environment

# 2. Job Control

# 3. Process Management

# 4. Special file and process

# 5. SELinux
