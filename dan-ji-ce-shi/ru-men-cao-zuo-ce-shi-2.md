---
description: 能力机制（Capability）是 Linux 内核一个强大的特性，可以提供细粒度的权限访问控制。
---

# 入门操作测试2

## 测试2：

以`entrypoint`指定命令运行busybox容器：

```text
sudo docker run --name bbox2 --rm -it --entrypoint /bin/sh busybox
```

这里添加的`--rm` 参数，是实现退出这个容器后马上把它删除

默认情况下，容器被严格限制只允许使用内核的一部分能力，根据实际情况，可以使用参数`--cap-add`和 `--cap-drop`来控制容器的能力。例如：

```text
sudo docker run --name bbox2 --rm -it --cap-add NET_ADMIN --cap-drop CHOWN --entrypoint /bin/sh busybox
```

更多用于docker的capability key如下表：（来自[https://docs.docker.com/engine/reference/run/](https://docs.docker.com/engine/reference/run/)）

| Capability Key | Capability Description |
| :--- | :--- |
| SETPCAP | Modify process capabilities. |
| MKNOD | Create special files using mknod\(2\). |
| AUDIT\_WRITE | Write records to kernel auditing log. |
| CHOWN | Make arbitrary changes to file UIDs and GIDs \(see chown\(2\)\). |
| NET\_RAW | Use RAW and PACKET sockets. |
| DAC\_OVERRIDE | Bypass file read, write, and execute permission checks. |
| FOWNER | Bypass permission checks on operations that normally require the file system UID of the process to match the UID of the file. |
| FSETID | Don’t clear set-user-ID and set-group-ID permission bits when a file is modified. |
| KILL | Bypass permission checks for sending signals. |
| SETGID | Make arbitrary manipulations of process GIDs and supplementary GID list. |
| SETUID | Make arbitrary manipulations of process UIDs. |
| NET\_BIND\_SERVICE | Bind a socket to internet domain privileged ports \(port numbers less than 1024\). |
| SYS\_CHROOT | Use chroot\(2\), change root directory. |
| SETFCAP | Set file capabilities. |

| Capability Key | Capability Description |
| :--- | :--- |
| SYS\_MODULE | Load and unload kernel modules. |
| SYS\_RAWIO | Perform I/O port operations \(iopl\(2\) and ioperm\(2\)\). |
| SYS\_PACCT | Use acct\(2\), switch process accounting on or off. |
| SYS\_ADMIN | Perform a range of system administration operations. |
| SYS\_NICE | Raise process nice value \(nice\(2\), setpriority\(2\)\) and change the nice value for arbitrary processes. |
| SYS\_RESOURCE | Override resource Limits. |
| SYS\_TIME | Set system clock \(settimeofday\(2\), stime\(2\), adjtimex\(2\)\); set real-time \(hardware\) clock. |
| SYS\_TTY\_CONFIG | Use vhangup\(2\); employ various privileged ioctl\(2\) operations on virtual terminals. |
| AUDIT\_CONTROL | Enable and disable kernel auditing; change auditing filter rules; retrieve auditing status and filtering rules. |
| MAC\_ADMIN | Allow MAC configuration or state changes. Implemented for the Smack LSM. |
| MAC\_OVERRIDE | Override Mandatory Access Control \(MAC\). Implemented for the Smack Linux Security Module \(LSM\). |
| NET\_ADMIN | Perform various network-related operations. |
| SYSLOG | Perform privileged syslog\(2\) operations. |
| DAC\_READ\_SEARCH | Bypass file read permission checks and directory read and execute permission checks. |
| LINUX\_IMMUTABLE | Set the FS\_APPEND\_FL and FS\_IMMUTABLE\_FL i-node flags. |
| NET\_BROADCAST | Make socket broadcasts, and listen to multicasts. |
| IPC\_LOCK | Lock memory \(mlock\(2\), mlockall\(2\), mmap\(2\), shmctl\(2\)\). |
| IPC\_OWNER | Bypass permission checks for operations on System V IPC objects. |
| SYS\_PTRACE | Trace arbitrary processes using ptrace\(2\). |
| SYS\_BOOT | Use reboot\(2\) and kexec\_load\(2\), reboot and load a new kernel for later execution. |
| LEASE | Establish leases on arbitrary files \(see fcntl\(2\)\). |
| WAKE\_ALARM | Trigger something that will wake up the system. |
| BLOCK\_SUSPEND | Employ features that can block system suspend. |

更多关于Linux Capability的知识点，可以访问这个网页：

[http://man7.org/linux/man-pages/man7/capabilities.7.html](http://man7.org/linux/man-pages/man7/capabilities.7.html)





\`\`

\`\`

\`\`

