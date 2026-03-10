# 部署 - MDT

Microsoft 部署工具包（MDT）是微软提供的免费工具，用于自动化 Windows 操作系统和应用程序的部署。

它允许 IT 管理员创建一个包含操作系统映像、驱动程序、更新和应用程序的中央部署共享，然后使用自动化脚本（任务序列）通过网络（轻触式安装）或从介质（USB/DVD）将它们安装到多台计算机上。

## 部署共享

这些文件包含 Microsoft 部署工具包用于将计算机加入域和访问网络资源的凭据。

* **Bootstrap.ini** - 位于 `DeploymentShare\Control\Bootstrap.ini`
* **CustomSettings.ini** - 位于 `DeploymentShare\Control\CustomSettings.ini`

| 名称 | 描述 |
| --- | --- |
| DomainAdmin | 用于将计算机加入域的帐户 |
| DomainAdminPassword | 用于将计算机加入域的密码 |
| UserID | 用于访问网络资源的帐户 |
| UserPassword | 用于访问网络资源的密码 |
| AdminPassword | 计算机上的本地管理员帐户 |
| ADDSUserName | 部署期间提升为 DC 时使用的帐户 |
| ADDSPassword | 部署期间提升为 DC 时使用的密码 |
| Password | 用于将成员服务器提升为域控制器的密码 |
| SafeModeAdminPassword | 部署 DC 时使用，即 AD 恢复模式密码 |
| TPMOwnerPassword | 如果尚未设置，则为 TPM 密码 |
| DBID | 部署期间用于连接 SQL Server 的帐户 |
| DBPwd | 部署期间用于连接 SQL Server 的密码 |
| OSDBitLockerRecoveryPassword | BitLocker 恢复密码 |

其他凭据可以在部署共享中托管的文件中找到：

* `DeploymentShare\Control\TASKSEQUENCENAME\ts.xml`
* `DeploymentShare\Scripts\` 文件夹
* `DeploymentShare\Applications` 文件夹
* `LiteTouchPE_x86|x64.iso`，提取文件并查找 `bootstrap.ini`
* `LiteTouchPE_x86|x64.wim`，提取文件并查找 `bootstrap.ini`

## 参考资料

* [Red Team Gold: Extracting Credentials from MDT Shares - Oddvar Moe - May 20, 2025](https://trustedsec.com/blog/red-team-gold-extracting-credentials-from-mdt-shares)
* [MDT, where are you? - BlackWasp - June 27, 2025](https://hideandsec.sh/books/windows-sNL/page/mdt-where-are-you)
