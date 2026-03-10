# 杂项与技巧 (Miscellaneous & Tricks)

这里包含一些未能归入其他分类的技巧。

## 向其他用户发送消息 (Send Messages to Other Users)

* Windows

```powershell
PS C:\> msg Swissky /SERVER:CRASHLAB "Stop rebooting the XXXX service !"
PS C:\> msg * /V /W /SERVER:CRASHLAB "Hello all !"
```

* Linux

```powershell
wall "Stop messing with the XXX service !"
wall -n "System will go down for 2 hours maintenance at 13:00 PM"  # "-n" 仅限 root 用户
who
write root pts/2 # 输入消息后按 Ctrl+D 发送。
```

## NetExec 凭据数据库 (NetExec Credential Database)

```ps1
nxcdb (default) > workspace create test
nxcdb (test) > workspace default
nxcdb (test) > proto smb
nxcdb (test)(smb) > creds
nxcdb (test)(smb) > export creds csv /tmp/creds
```

NetExec 工作区 (Workspaces)

```ps1
# 获取当前工作区
poetry run nxcdb -gw 

# 创建工作区
poetry run nxcdb -cw testing

# 设置工作区
poetry run nxcdb -sw testing 
```
