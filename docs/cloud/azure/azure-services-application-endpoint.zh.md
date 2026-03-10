# Azure 服务 - 应用程序端点 (Application Endpoint)

## 枚举 (Enumerate)

* 枚举显示名称以 PREFIX 开头/结尾的应用程序的可能端点

    ```powershell
    PS C:\Tools> Get-AzureADServicePrincipal -All $true -Filter "startswith(displayName,'PREFIX')" | % {$_.ReplyUrls}
    PS C:\Tools> Get-AzureADApplication -All $true -Filter "endswith(displayName,'PREFIX')" | Select-Object ReplyUrls,WwwHomePage,HomePage
    ```

## 访问 (Access)

```ps1
https://myapps.microsoft.com/signin/<App ID>?tenantId=<TenantID>
```

## 参考资料 (References)

* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
