# Active Directory - 枚举

## 使用 BloodHound

使用适当的数据收集器收集 **BloodHound** 或 **BloodHound 社区版（CE）** 在各平台上的信息。

* [BloodHoundAD/AzureHound](https://github.com/BloodHoundAD/AzureHound) 用于 Azure Active Directory
* [BloodHoundAD/SharpHound](https://github.com/BloodHoundAD/SharpHound) 用于本地 Active Directory（C# 收集器）
* [FalconForceTeam/SOAPHound](https://github.com/FalconForceTeam/SOAPHound) 用于本地 Active Directory（使用 ADWS 的 C# 收集器）
* [g0h4n/RustHound-CE](https://github.com/g0h4n/RustHound-CE) 用于本地 Active Directory（Rust 收集器）
* [NH-RED-TEAM/RustHound](https://github.com/NH-RED-TEAM/RustHound) 用于本地 Active Directory（Rust 收集器）
* [fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py) 用于本地 Active Directory（Python 收集器）
* [coffeegist/bofhound](https://github.com/coffeegist/bofhound) 用于本地 Active Directory（从 ldapsearch BOF、pyldapsearch 和 Brute Ratel 的 LDAP Sentinel 写入的日志生成 BloodHound 兼容 JSON）
* [c3c/ADExplorerSnapshot.py](https://github.com/c3c/ADExplorerSnapshot.py) 用于本地 Active Directory（从 AD Explorer 快照生成 BloodHound 兼容 JSON）
* [CrowdStrike/sccmhound](https://github.com/CrowdStrike/sccmhound) 用于本地 Active Directory（使用 Microsoft Configuration Manager 的 C# 收集器）
* [SpecterOps/MSSQLHound](https://github.com/SpecterOps/MSSQLHound) 用于 MSSQL 攻击路径（BloodHound OpenGraph PowerShell 收集器）
* [SpecterOps/SnowHound](https://github.com/SpecterOps/SnowHound) 用于 Snowflake 攻击路径（BloodHound OpenGraph PowerShell 收集器）
* [SpecterOps/GitHound](https://github.com/SpecterOps/GitHound) 用于 GitHub 攻击路径（BloodHound OpenGraph PowerShell 收集器）
* [SpecterOps/1PassHound](https://github.com/SpecterOps/1PassHound) 用于 1Password 攻击路径（BloodHound OpenGraph PowerShell 收集器）
* [TheSleekBoyCompany/AnsibleHound](https://github.com/TheSleekBoyCompany/AnsibleHound) 用于 Ansible WorX 和 Ansible Tower 攻击路径（BloodHound OpenGraph Go 收集器）
* [p0dalirius/sharehound](https://github.com/p0dalirius/sharehound) - 用于网络共享攻击路径（BloodHound OpenGraph Python 收集器）
* [C0KERNEL/SecretHound](https://github.com/C0KERNEL/SecretHound) - 用于机密信息（BloodHound OpenGraph Python 收集器）
* [F41zK4r1m/GCP-Hound](https://github.com/F41zK4r1m/GCP-Hound) - 用于 GCP 攻击路径（BloodHound OpenGraph Python 收集器）
* [SpecterOps/ConfigManBearPig](https://github.com/SpecterOps/ConfigManBearPig) - 用于 SCCM 攻击路径（BloodHound OpenGraph PowerShell 收集器）

**示例**：

* 使用 [BloodHoundAD/AzureHound](https://github.com/BloodHoundAD/AzureHound)（更多信息：[Cloud - Azure Pentest](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cloud%20-%20Azure%20Pentest.md#azure-recon-tools)）

* 使用 [BloodHoundAD/SharpHound.exe](https://github.com/BloodHoundAD/BloodHound) - 在机器上使用 SharpHound.exe 运行收集器

  ```powershell
  .\SharpHound.exe -c all -d active.htb --searchforest
  .\SharpHound.exe -c all,GPOLocalGroup # all 收集默认不包含 GPOLocalGroup
  .\SharpHound.exe --CollectionMethod DCOnly # 仅从 DC 收集，不查询计算机（更隐蔽）

  .\SharpHound.exe -c all --LdapUsername <UserName> --LdapPassword <Password> --JSONFolder <PathToFile>
  .\SharpHound.exe -c all --LdapUsername <UserName> --LdapPassword <Password> --domaincontroller 10.10.10.100 -d active.htb

  .\SharpHound.exe -c All,GPOLocalGroup --outputdirectory C:\Windows\Temp --prettyprint --randomfilenames --collectallproperties --throttle 10000 --jitter 23  --outputprefix internalallthething
  ```

* 使用 [BloodHoundAD/SharpHound.ps1](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.ps1) - 在机器上使用 PowerShell 运行收集器

  ```powershell
  Invoke-BloodHound -SearchForest -CSVFolder C:\Users\Public
  Invoke-BloodHound -CollectionMethod All  -LDAPUser <UserName> -LDAPPass <Password> -OutputDirectory <PathToFile>
  ```

* 使用 [ly4k/Certipy](https://github.com/ly4k/Certipy) 收集证书数据

  ```ps1
  certipy find 'corp.local/john:Passw0rd@dc.corp.local' -bloodhound
  certipy find 'corp.local/john:Passw0rd@dc.corp.local' -old-bloodhound
  certipy find 'corp.local/john:Passw0rd@dc.corp.local' -vulnerable -hide-admins -username user@domain -password Password123
  ```

* 使用 [NH-RED-TEAM/RustHound](https://github.com/OPENCYBER-FR/RustHound)

  ```ps1
  # Windows 使用 GSSAPI 会话
  rusthound.exe -d domain.local --ldapfqdn domain
  # Windows/Linux 使用用户名:密码进行简单绑定连接
  rusthound.exe -d domain.local -u user@domain.local -p Password123 -o output -z
  # Linux 使用用户名:密码和 ADCS 模块（@ly4k BloodHound 版本）
  rusthound -d domain.local -u 'user@domain.local' -p 'Password123' -o /tmp/adcs --adcs -z
  ```

* 使用 [FalconForceTeam/SOAPHound](https://github.com/FalconForceTeam/SOAPHound)

  ```ps1
  --buildcache: 仅构建缓存，不执行进一步操作
  --bhdump: 导出 BloodHound 数据
  --certdump: 导出 AD 证书服务（ADCS）数据
  --dnsdump: 导出 AD 集成 DNS 数据

  SOAPHound.exe --buildcache -c c:\temp\cache.txt
  SOAPHound.exe -c c:\temp\cache.txt --bhdump -o c:\temp\bloodhound-output
  SOAPHound.exe -c c:\temp\cache.txt --bhdump -o c:\temp\bloodhound-output --autosplit --threshold 1000
  SOAPHound.exe -c c:\temp\cache.txt --certdump -o c:\temp\bloodhound-output
  SOAPHound.exe --dnsdump -o c:\temp\dns-output
  ```

* 使用 [fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py)

  ```ps1
  pip install bloodhound
  bloodhound-python -d domain.local -u username -p password -gc LAB2008DC01.domain.local -c all
  ```

* 使用 [c3c/ADExplorerSnapshot.py](https://github.com/c3c/ADExplorerSnapshot.py) 从 SysInternals/ADExplorer 快照中查询数据（ADExplorer 仍然是微软签名的合法二进制文件，可避免安全解决方案的检测）。

  ```py
  ADExplorerSnapshot.py <snapshot path> -o <*.json output folder path>
  ```

然后将 zip/json 文件导入 Neo4J 数据库并查询。

```powershell
root@payload$ apt install bloodhound 

# 启动 BloodHound 和数据库
root@payload$ neo4j console
# 或使用 docker
root@payload$ docker run -itd -p 7687:7687 -p 7474:7474 --env NEO4J_AUTH=neo4j/bloodhound -v $(pwd)/neo4j:/data neo4j:4.4-community

root@payload$ ./bloodhound --no-sandbox
访问 http://127.0.0.1:7474，使用 db:bolt://localhost:7687，user:neo4J，pass:neo4j
```

注意：目前 BloodHound 社区版仍在开发中，强烈建议继续使用原版 [BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound/)。

```ps1
git clone https://github.com/SpecterOps/BloodHound
cd examples/docker-compose/
cat docker-compose.yml | docker compose -f - up
# UI: http://localhost:8080/ui/login
# 用户名: admin
# 密码: 查看 Docker 日志
```

你可以添加一些自定义查询：

* [BloodHound Queries For All - SpecterOps](https://queries.specterops.io/)
* [Bloodhound-Custom-Queries from @hausec](https://github.com/hausec/Bloodhound-Custom-Queries/blob/master/customqueries.json)
* [BloodHoundQueries from CompassSecurity](https://github.com/CompassSecurity/BloodHoundQueries/blob/master/customqueries.json)
* [BloodHound Custom Queries from Exegol - @ShutdownRepo](https://raw.githubusercontent.com/ThePorgs/Exegol-images/main/sources/assets/bloodhound/customqueries.json)
* [Certipy BloodHound Custom Queries from ly4k](https://github.com/ly4k/Certipy/blob/main/customqueries.json)

替换位于 `/home/username/.config/bloodhound/customqueries.json` 或 `C:\Users\USERNAME\AppData\Roaming\BloodHound\customqueries.json` 的 customqueries.json 文件。

## 使用 PowerView
  
* **获取当前域：** `Get-NetDomain`
* **枚举其他域：** `Get-NetDomain -Domain <DomainName>`
* **获取域 SID：** `Get-DomainSID`
* **获取域策略：**

  ```powershell
  Get-DomainPolicy

  # 显示关于系统访问或 Kerberos 的域策略配置
  (Get-DomainPolicy)."system access"
  (Get-DomainPolicy)."kerberos policy"
  ```

* **获取域控制器：**

  ```powershell
  Get-NetDomainController
  Get-NetDomainController -Domain <DomainName>
  ```

* **枚举域用户：**

  ```powershell
  Get-NetUser
  Get-NetUser -SamAccountName <user> 
  Get-NetUser | select cn
  Get-UserProperty

  # 检查最后一次密码更改时间
  Get-UserProperty -Properties pwdlastset

  # 在用户属性中搜索特定字符串
  Find-UserField -SearchField Description -SearchTerm "wtver"
  
  # 枚举登录到某台机器上的用户
  Get-NetLoggedon -ComputerName <ComputerName>
  
  # 枚举机器的会话信息
  Get-NetSession -ComputerName <ComputerName>
  
  # 枚举当前/指定域中特定用户登录的域计算机
  Find-DomainUserLocation -Domain <DomainName> | Select-Object UserName, SessionFromName
  ```

* **枚举域计算机：**

  ```powershell
  Get-NetComputer -FullData
  Get-DomainGroup

  # 枚举在线机器
  Get-NetComputer -Ping
  ```

* **枚举组和组成员：**

  ```powershell
  Get-NetGroupMember -GroupName "<GroupName>" -Domain <DomainName>
  
  # 枚举域中指定组的成员
  Get-DomainGroup -Identity <GroupName> | Select-Object -ExpandProperty Member
  
  # 返回域中通过受限组或组策略首选项修改本地组成员资格的所有 GPO
  Get-DomainGPOLocalGroup | Select-Object GPODisplayName, GroupName
  ```

* **枚举共享**

  ```powershell
  # 枚举域共享
  Find-DomainShare
  
  # 枚举当前用户有权访问的域共享
  Find-DomainShare -CheckShareAccess
  ```

* **枚举组策略：**

  ```powershell
  Get-NetGPO

  # 显示指定机器上的活动策略
  Get-NetGPO -ComputerName <Name of the PC>
  Get-NetGPOGroup

  # 获取属于某台机器本地管理员组的用户
  Find-GPOComputerAdmin -ComputerName <ComputerName>
  ```

* **枚举 OU：**

  ```powershell
  Get-NetOU -FullData 
  Get-NetGPO -GPOname <The GUID of the GPO>
  ```

* **枚举 ACL：**

  ```powershell
  # 返回与指定帐户关联的 ACL
  Get-ObjectAcl -SamAccountName <AccountName> -ResolveGUIDs
  Get-ObjectAcl -ADSprefix 'CN=Administrator, CN=Users' -Verbose

  # 搜索有趣的 ACE
  Invoke-ACLScanner -ResolveGUIDs

  # 检查与指定路径（如 SMB 共享）关联的 ACL
  Get-PathAcl -Path "\\Path\Of\A\Share"
  ```

* **枚举域信任：**

  ```powershell
  Get-NetDomainTrust
  Get-NetDomainTrust -Domain <DomainName>
  ```

* **枚举森林信任：**

  ```powershell
  Get-NetForestDomain
  Get-NetForestDomain Forest <ForestName>

  # 森林域枚举
  Get-NetForestDomain
  Get-NetForestDomain Forest <ForestName>

  # 映射森林的信任关系
  Get-NetForestTrust
  Get-NetDomainTrust -Forest <ForestName>
  ```

* **用户搜索：**

  ```powershell
  # 查找当前域中当前用户拥有本地管理员权限的所有机器
  Find-LocalAdminAccess -Verbose

  # 查找域中所有机器上的本地管理员：
  Invoke-EnumerateLocalAdmin -Verbose

  # 查找域管理员或指定用户有会话的计算机
  Invoke-UserHunter
  Invoke-UserHunter -GroupName "RDPUsers"
  Invoke-UserHunter -Stealth

  # 确认管理员权限：
  Invoke-UserHunter -CheckAccess
  ```

## 使用 AD 模块

* **获取当前域：** `Get-ADDomain`
* **枚举其他域：** `Get-ADDomain -Identity <Domain>`
* **获取域 SID：** `Get-DomainSID`
* **获取域控制器：**

  ```powershell
  Get-ADDomainController
  Get-ADDomainController -Identity <DomainName>
  ```
  
* **枚举域用户：**

  ```powershell
  Get-ADUser -Filter * -Identity <user> -Properties *

  # 在用户属性中搜索特定字符串
  Get-ADUser -Filter 'Description -like "*wtver*"' -Properties Description | select Name, Description
  ```

* **枚举域计算机：**

  ```powershell
  Get-ADComputer -Filter * -Properties *
  Get-ADGroup -Filter * 
  ```

* **枚举域信任：**

  ```powershell
  Get-ADTrust -Filter *
  Get-ADTrust -Identity <DomainName>
  ```

* **枚举森林信任：**

  ```powershell
  Get-ADForest
  Get-ADForest -Identity <ForestName>

  # 森林域枚举
  (Get-ADForest).Domains
  ```

* **枚举本地 AppLocker 有效策略：**

 ```powershell
 Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
 ```

## 用户搜索

有时你需要找到某个特定用户登录的机器。你可以远程查询网络中的每台机器以获取用户会话列表。

* netexec

  ```ps1
  nxc smb 10.10.10.0/24 -u Administrator -p 'P@ssw0rd' --sessions
  SMB         10.10.10.10    445    WIN-8OJFTLMU1IG  [+] Enumerated sessions
  SMB         10.10.10.10    445    WIN-8OJFTLMU1IG  \\10.10.10.10            User:Administrator
  ```

* Impacket Smbclient

  ```ps1
  $ impacket-smbclient Administrator@10.10.10.10
  # who
  host:  \\10.10.10.10, user: Administrator, active:     1, idle:     0
  ```

* PowerView Invoke-UserHunter

  ```ps1
  # 查找域管理员或指定用户有会话的计算机
  Invoke-UserHunter
  Invoke-UserHunter -GroupName "RDPUsers"
  Invoke-UserHunter -Stealth
  ```

## RID 循环枚举

在 Windows 中，每个安全主体（用户、组等）都有一个安全标识符（SID）。SID 是用于访问控制的唯一标识符。

```ps1
S-1-5-21-<domain>-<RID>
```

* `S-1-5-21-<domain>` = 基域 SID
* `<RID>` = 分配给用户/组的唯一 ID

RID 循环枚举涉及暴力破解一系列 RID（如 500-1500），将其附加到已知的域 SID 上，并尝试将每个 SID 解析为用户名。

* 使用 [Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)

  ```ps1
  netexec smb 10.10.11.231 -u guest -p '' --rid-brute 10000 --log rid-brute.txt
  SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
  SMB         10.10.11.231    445    DC01             [+] rebound.htb\guest: 
  SMB         10.10.11.231    445    DC01             498: rebound\Enterprise Read-only Domain Controllers (SidTypeGroup)
  SMB         10.10.11.231    445    DC01             500: rebound\Administrator (SidTypeUser)
  SMB         10.10.11.231    445    DC01             501: rebound\Guest (SidTypeUser)
  SMB         10.10.11.231    445    DC01             502: rebound\krbtgt (SidTypeUser)
  ```

* 使用 Impacket 脚本 [impacket/lookupsid.py](https://github.com/fortra/impacket/blob/master/examples/lookupsid.py)

  ```ps1
  lookupsid.py -no-pass 'guest@rebound.htb' 20000
  ```

## 其他有用命令

* **查找域控制器**

  ```ps1
  nslookup domain.com
  nslookup -type=srv _ldap._tcp.dc._msdcs.<domain>.com
  nltest /dclist:domain.com
  Get-ADDomainController -filter * | Select-Object name
  gpresult /r
  $Env:LOGONSERVER 
  echo %LOGONSERVER%
  ```

## 参考资料

* [Explain like I'm 5: Kerberos - Apr 2, 2013 - @roguelynn](https://www.roguelynn.com/words/explain-like-im-5-kerberos/)
* [Pen Testing Active Directory Environments - Part I: Introduction to netexec (and PowerView)](https://blog.varonis.com/pen-testing-active-directory-environments-part-introduction-netexec-powerview/)
* [Pen Testing Active Directory Environments - Part II: Getting Stuff Done With PowerView](https://blog.varonis.com/pen-testing-active-directory-environments-part-ii-getting-stuff-done-with-powerview/)
* [Pen Testing Active Directory Environments - Part III:  Chasing Power Users](https://blog.varonis.com/pen-testing-active-directory-environments-part-iii-chasing-power-users/)
* [Pen Testing Active Directory Environments - Part IV: Graph Fun](https://blog.varonis.com/pen-testing-active-directory-environments-part-iv-graph-fun/)
* [Pen Testing Active Directory Environments - Part V: Admins and Graphs](https://blog.varonis.com/pen-testing-active-directory-v-admins-graphs/)
* [Pen Testing Active Directory Environments - Part VI: The Final Case](https://blog.varonis.com/pen-testing-active-directory-part-vi-final-case/)
* [Attacking Active Directory: 0 to 0.9 - Eloy Pérez González - 2021/05/29](https://zer1t0.gitlab.io/posts/attacking_ad/)
* [Fun with LDAP, Kerberos (and MSRPC) in AD Environments](https://speakerdeck.com/ropnop/fun-with-ldap-kerberos-and-msrpc-in-ad-environments)
* [Penetration Testing Active Directory, Part I - March 5, 2019 - Hausec](https://hausec.com/2019/03/05/penetration-testing-active-directory-part-i/)
* [Penetration Testing Active Directory, Part II - March 12, 2019 - Hausec](https://hausec.com/2019/03/12/penetration-testing-active-directory-part-ii/)
* [Using bloodhound to map the user network - Hausec](https://hausec.com/2017/10/26/using-bloodhound-to-map-the-user-network/)
* [PowerView 3.0 Tricks - HarmJ0y](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)
* [SOAPHound - tool to collect Active Directory data via ADWS - Nikos Karouzos - 01/26/204](https://medium.com/falconforce/soaphound-tool-to-collect-active-directory-data-via-adws-165aca78288c)
* [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
