# Chapter 18. System service and daemon

# 1. What is `daemon` and service
## 1.1 `daemon` types

### stand_alone: start a service by itself

Daemon can start a service without other mechanism. The loaded daemon will stay in the memory all the time but the response is **fast** too.

### super_daemon: one daemon manages all services

When there is no request, `xinetd`(super_daemon) will deallocate all resources. It will call service when requests come from clients. `xinetd` has **security** mechanism. No busy system resources. But it will be **slower** to respond request!

# 2. Analyze configuration of **super daemon**

# 3. Firewall of service `xinetd`, `TCP Wrappers`

# 4. Services started by system
