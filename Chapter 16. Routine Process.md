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
# 2. Routine Process only once

# 3. Routine Process loop

# 4. Routine Process during power off
