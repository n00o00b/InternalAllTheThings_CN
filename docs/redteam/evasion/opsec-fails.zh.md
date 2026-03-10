# 运维安全 (OPSEC)

## 基础设施 (Infrastructure)

* DNS 使用通用名称，避免包含公司名称。
* 签发证书时使用通配符 (*)，以避免泄露内部名称。
* 不要使用 C2 工具中嵌入的默认证书：[elastic/Default Cobalt Strike Team Server Certificate](https://www.elastic.co/docs/reference/security/prebuilt-rules/rules/network/command_and_control_cobalt_strike_default_teamserver_cert), [zeek/zeek_default_cobalt_strike_certificate](https://detection.fyi/sigmahq/sigma/network/zeek/zeek_default_cobalt_strike_certificate/)。

    ```cs
    (event.dataset: network_traffic.tls or event.category: (network or network_traffic))
    and (tls.server.hash.md5:950098276A495286EB2A2556FBAB6D83
    or tls.server.hash.sha1:6ECE5ECE4192683D2D84E25B0BA7E04F9CB7EB7C
    or tls.server.hash.sha256:87F2085C32B6A2CC709B365F55873E207A9CAA10BFFECF2FD16D3CF9D94D390C)
    ```

* 禁用暂存端点 (Staging endpoints) 或限制其访问。
* 不要将你的隐形二进制文件 (Stealthy binaries) 上传到 VirusTotal 或其他在线扫描器。
* 为你的 payload 设置防护栏 (Guardrails)，使其仅针对特定的用户/域名/计算机名触发。
* 使用重定向器 (Redirector)，不要将你的 C2 TLS 栈直接暴露在互联网上。

## 行为 (Behavior)

* 避免调用 `whoami` 等命令：
    * 列出你的 Kerberos 票据。
    * 查找生成你的 Beacon 的进程所有者。
    * 列出你的环境变量：`dir env:` 和 `dir env:USERNAME`。
    * 使用 Beacon Object File (BOF) 来运行自带的 whoami。
* DCSync (Replication) 始终在域控制器之间进行：
    * 来自机器账户的 DCSync 看起来比使用用户账户更合法。
    * 你不需要导出整个数据库，`krbtgt` 账户将授予你所需的所有权限。
* Kerberoasting 必须使用正确的加密方式，攻击性工具通常默认使用 RC4 而不是 AES。

## 入侵指标 (IOC)

**Gophish**:

* 默认 `RID` 参数：[gophish/campaign.go#L130](https://github.com/gophish/gophish/blob/8e79294413932fa302212d8e785b281fb0f8896d/models/campaign.go#L130)
* 默认 `X-Mailer` 请求头中包含 `ServerName`：[gophish/config.go#L46](https://github.com/gophish/gophish/blob/8e79294413932fa302212d8e785b281fb0f8896d/config/config.go#L46)
* 默认 `X-Gophish-Contact`：[gophish/email_request.go#L123](https://github.com/gophish/gophish/blob/8e79294413932fa302212d8e785b281fb0f8896d/models/email_request.go#L123)

**Impacket**:

* `smbexec.py` 使用服务来执行命令。在最早的版本中，它被命名为 `BTOBTO`，但现在是 8 个随机字符。将其更改为 10 个以上的字符以破坏其他关联规则。
* `psexec.py` 基于 2012 年 1 月发布的知名服务：[kavika13/RemComSvc](https://github.com/kavika13/RemCom)
* `wmiexec.py` 的每个命令都会带有 `cmd.exe /Q /c` 前缀：[impacket/wmiexec.py#L127](https://github.com/fortra/impacket/blob/master/examples/wmiexec.py#L127)

**NetExec**:

* NetExec 使用 Impacket 库，因此共享相同的 IOC。
* Kerberoasting 搜索过滤器查询所有账户：[NetExec/ldap.py#L931](https://github.com/Pennyw0rth/NetExec/blob/5f29e661b7e2f367faf2af7688f777d8b2d1bf6d/nxc/protocols/ldap.py#L931)

    ```py
    (&(servicePrincipalName=*)(UserAccountControl:1.2.840.113556.1.4.803:=512)(!(UserAccountControl:1.2.840.113556.1.4.803:=2))(!(objectCategory=computer)))
    ```

**AWS**:

* AWS CLI 使用 Boto3 库，它在每个请求中都发送包含操作系统版本的 User-Agent：
    * Kali Linux 操作系统会触发警报：[PenTest:IAMUser/KaliLinux](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-iam.html#pentest-iam-kalilinux)

## 参考资料 (References)

* [DLS 2024 - RedTeam Fails - "Oops my bad I ruined the operation" - Swissky - January 15, 2024](https://swisskyrepo.github.io/Drink-Love-Share-Rump/)
* [Five Ways I got Caught before Lunch - Mystikcon 2021 - cyberv1s3r1on3 - November 24, 2021](https://youtu.be/qIbrozlf2wM)
