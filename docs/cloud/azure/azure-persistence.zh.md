# Azure AD - 持久化 (Persistence)

## 向应用程序添加机密 (Add Secrets to Application)

* 使用 [lutzenfried/OffensiveCloud/Add-AzADAppSecret.ps1](https://github.com/lutzenfried/OffensiveCloud/blob/main/Azure/Tools/Add-AzADAppSecret.ps1) 添加机密 (secrets)

    ```powershell
    PS > . C:\Tools\Add-AzADAppSecret.ps1
    PS > Add-AzADAppSecret -GraphToken $graphtoken -Verbose
    ```

* 使用机密以服务主体 (Service Principal) 身份进行身份验证

    ```ps1
    PS > $password = ConvertTo-SecureString '<机密/密码>' -AsPlainText -Force
    PS > $creds = New-Object System.Management.Automation.PSCredential('<AppID>', $password)
    PS > Connect-AzAccount -ServicePrincipal -Credential $creds -Tenant '<TenantID>'
    ```

## 添加服务主体 (Add Service Principal)

* 生成新的服务主体密码/机密

    ```ps1
    Import-Module Microsoft.Graph.Applications
    Connect-MgGraph 
    $servicePrincipalId = "<服务主体 ID>"

    $params = @{
        passwordCredential = @{
            displayName = "NewCreds"
        }
    }
    Add-MgServicePrincipalPassword -ServicePrincipalId $servicePrincipalId -BodyParameter $params
    ```

## 将用户添加到组 (Add User to Group)

```ps1
Add-AzureADGroupMember -ObjectId <组 ID> -RefObjectId <用户 ID> -Verbose
```

## 使用 KFM 的 PowerShell 配置文件后门 (PowerShell Profile Backdoor Using KFM)

OneDrive for Business 的“已知文件夹移动 (Known Folder Move / KFM)”是 Microsoft OneDrive for Business 中的一项功能，它允许用户和组织自动将关键 Windows 用户文件夹（桌面、文档和图片）的内容从本地电脑重定向到 OneDrive。

PowerShell 配置文件 (Profile) 是一个脚本文件，每当你启动新的 PowerShell 会话（例如打开 PowerShell 或 Windows Terminal）时都会加载该文件。用户和管理员经常自定义其配置文件以设置别名、环境变量、函数或预加载模块。

**要求**：

* `Files.ReadWrite.All` 权限

**方法论**：

已知文件夹移动 (Known Folder Move) 会将用户的“文档 (Documents)”（和/或“桌面”、“图片”）文件夹移动到 OneDrive for Business，通常同步路径为：

```ps1
C:\Users\<用户名>\Documents → C:\Users\<用户名>\OneDrive - <租户名称>\Documents
```

这意味着 PowerShell 配置文件 (`Documents\PowerShell\Microsoft.PowerShell_profile.ps1`) 现在将被同步到 OneDrive。

将恶意 PowerShell 配置文件推送至 `$HOME\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`。

## 参考资料 (References)

* [High-Profile Cloud Privesc - Leonidas Tsaousis - July 15, 2025](https://labs.reversec.com/posts/2025/07/high-profile-cloud-privesc)
* [Maintaining Azure Persistence via automation accounts - Karl Fosaaen - September 12, 2019](https://blog.netspi.com/maintaining-azure-persistence-via-automation-accounts/)
* [Microsoft Graph - servicePrincipal: addPassword](https://learn.microsoft.com/en-us/graph/api/serviceprincipal-addpassword?view=graph-rest-1.0&tabs=powershell)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
