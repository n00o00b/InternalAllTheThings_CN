# Azure - 要求 (Requirements)

## 渗透测试要求 (Pentest Requirements)

用户和角色：

* Azure AD 中的 **全局读者 (Global Reader)** 和 **安全读者 (Security Reader)** 角色
* 对订阅的 **读者 (Reader)** 权限

订阅 (Subscriptions)：

* [Azure Dev/Test](https://azure.microsoft.com/en-us/pricing/offers/dev-test) 订阅。
* Visual Studio 订阅将决定你每月获得的 Azure 额度：
    * Visual Studio Enterprise: 每月 150 美元
    * MSDN Platforms: 100 美元
    * Visual Studio Professional: 50 美元
    * Visual Studio Test Professional: 50 美元

## Powershell 与原生模块 (Powershell and Native Modules)

* [Microsoft Graph](https://learn.microsoft.com/en-us/powershell/microsoftgraph/installation?view=graph-powershell-1.0): `Install-Module Microsoft.Graph -Scope CurrentUser`
* [Azure AD](https://learn.microsoft.com/fr-fr/powershell/azure/active-directory/install-adv2?view=azureadps-2.0): `Install-Module AzureAD`
* [Azure AD Preview](https://learn.microsoft.com/fr-fr/powershell/azure/active-directory/install-adv2?view=azureadps-2.0): `Install-Module AzureADPreview`
* [Azure CLI](https://learn.microsoft.com/fr-fr/cli/azure/install-azure-cli-windows?tabs=winget): `winget install -e --id Microsoft.AzureCLI`

## 术语 (Terminology)

* **租户 (Tenant)**：Azure AD 的一个实例，代表单个组织。
* **Azure AD 目录 (Azure AD Directory)**：每个租户都有一个专用目录。其用于执行资源的身份和访问管理功能。
* **订阅 (Subscriptions)**：用于支付服务费用。一个目录中可以有多个订阅。
* **核心域名 (Core Domain)**：初始域名 `<tenant>.onmicrosoft.com` 即为核心域名。也可以定义自定义域名。

## 参考资料 (References)

* [Az - Permissions for a Pentest - HackTricks](https://cloud.hacktricks.xyz/pentesting-cloud/azure-security/az-permissions-for-a-pentest)
* [An introduction to penetration testing Azure - HollyGraceful - 06 August 2021](https://akimbocore.com/article/introduction-to-pentesting-azure/)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
