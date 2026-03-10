# Active Directory - 机器帐户配额

在 Active Directory（AD）中，`MachineAccountQuota` 是对特定用户或组可以在域中创建的计算机帐户数量的限制。

当用户尝试创建新的计算机帐户时，AD 会将该用户已创建的计算机帐户当前数量与为该用户或组定义的配额进行对比检查。

然而，Active Directory 不会直接在用户属性中存储已创建的机器帐户当前计数。相反，你需要执行查询来计算由特定用户创建的机器帐户数量。

## 机器帐户配额流程

1. **配额定义**：`MachineAccountQuota` 在域级别定义，可以为单个用户或组设置。默认情况下，对"域管理员"组设置为 **10**，对标准用户设置为 0，限制他们创建计算机帐户的能力。

    ```powershell
    nxc ldap <ip> -u user -p pass -M maq
    ```

2. **创建过程**：当用户尝试创建新的计算机帐户（例如通过 Active Directory 用户和计算机中的"添加计算机"选项或通过 PowerShell）时，帐户创建请求会发送到域控制器（DC）。

    ```powershell
    impacket@linux> addcomputer.py -computer-name 'ControlledComputer$' -computer-pass 'ComputerPassword' -dc-host DC01 -domain-netbios domain 'domain.local/user1:complexpassword'
    ```

3. **配额评估**：在创建帐户之前，Active Directory 会检查该用户创建的计算机帐户当前计数。这是通过查询 `msDS-CreatorSID` 属性来完成的，该属性保存创建该对象的用户的 SID。
系统将此计数与为该用户设置的 `MachineAccountQuota` 值进行比较。如果计数小于配额，则继续创建；如果等于或超过配额，则拒绝创建并返回错误。

    ```powershell
    # 将 DOMAIN\username 替换为实际的域和用户名
    $user = "DOMAIN\username"

    # 获取用户的 SID
    $userSID = (Get-ADUser -Identity $user).SID

    # 计算由此用户创建的计算机帐户数量
    $computerCount = (Get-ADComputer -Filter { msDS-CreatorSID -eq $userSID }).Count

    # 显示计数
    $computerCount
    ```

4. **失败处理**：如果超过配额，尝试创建帐户的用户将收到错误消息，指示他们因达到配额限制而无法创建新的计算机帐户。

## 参考资料

* [MachineAccountQuota - The Hacker Recipes - 24/10/2024](https://www.thehacker.recipes/ad/movement/builtins/machineaccountquota)
* [MachineAccountQuota is USEFUL Sometimes: Exploiting One of Active Directory's Oddest Settings - Kevin Robertson - March 6, 2019](https://www.netspi.com/blog/technical-blog/network-penetration-testing/machineaccountquota-is-useful-sometimes/)
* [Machine Account Quota - NetExec - 13/09/2023](https://www.netexec.wiki/ldap-protocol/machine-account-quota)
