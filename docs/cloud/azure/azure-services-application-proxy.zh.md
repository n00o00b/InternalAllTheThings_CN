# Azure 服务 - 应用程序代理 (Application Proxy)

## 枚举 (Enumerate)

* 枚举配置了代理 (Proxy) 的应用程序

    ```powershell
    PS C:\Tools> Get-AzureADApplication -All $true | %{try{GetAzureADApplicationProxyApplication -ObjectId $_.ObjectID;$_.DisplayName;$_.ObjectID}catch{}}
    PS C:\Tools> Get-AzureADServicePrincipal -All $true | ?{$_.DisplayName -eq "Finance Management System"}

    PS C:\Tools> . C:\Tools\GetApplicationProxyAssignedUsersAndGroups.ps1
    PS C:\Tools> Get-ApplicationProxyAssignedUsersAndGroups -ObjectId <对象 ID>
    ```

## 参考资料 (References)

* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
