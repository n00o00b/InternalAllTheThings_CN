# Azure 服务 - Web 应用 (Web Apps)

## 列出 Web 应用 (List Web App)

```ps1
az webapp list
```

## 执行命令 (Execute Commands)

```ps1
$ARMToken = Get-ARMTokenWithRefreshToken `
    -RefreshToken "0.ARwA6WgJJ9X2qk..." `
    -TenantID "contoso.onmicrosoft.com"

Invoke-AzureRMWebAppShellCommand `
    -KuduURI "https://<webapp>.scm.azurewebsites.net/api/command" `
    -Token $ARMToken `
    -Command "whoami"
```

## SSH 连接 (SSH Connection)

首先检查是否启用了基于 HTTP 的 SSH 连接：`(curl https://${appName}?app.scm.azurewebsites.net/webssh/host).statuscode`

```powershell
az webapp create-remote-connection --subscription <订阅 ID> --resource-group <资源组名称> -n <应用服务名称>
```

## Kudu

在 Azure 应用服务 (App Service) 中，Kudu 是先进的管理和部署工具，用于各种操作，如 Web 应用程序的持续集成、故障排除和诊断任务。它提供了一组用于管理应用环境的实用程序和功能，包括访问应用设置、日志流和部署管理。

你可以通过以下 URL 访问此 Kudu 应用：

* 不在隔离层 (Isolated tier) 中的应用：`https://<应用名称>.scm.azurewebsites.net`
* 隔离层中面向互联网的应用 (应用服务环境 / ASE)：`https://<应用名称>.scm.<ase 名称>.p.azurewebsites.net`
* 隔离层中内部应用 (用于内部负载均衡的应用服务环境)：`https://<应用名称>.scm.<ase 名称>.appserviceenvironment.net`

应用服务中 Kudu 的主要功能：

* **基于 Web 的控制台 (Web-Based Console)**：提供命令行界面 (CLI)，直接在应用服务环境中执行命令。
* **文件资源管理器 (File Explorer)**：让你查看和管理应用环境中的文件。
* **环境诊断 (Environment Diagnostics)**：提供对环境变量、应用设置和详细诊断日志的见解。
* **进程资源管理器 (Process Explorer)**：允许你监视和管理在应用环境中运行的进程。
* **日志访问 (Access to Logs)**：轻松查看、下载和流式传输日志，以便进行调试和故障排除。

## 参考资料 (References)

* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
