# MSSQL - 数据库枚举 (Database Enumeration)

## 目录 (Summary)

- [工具 (Tools)](#tools)
- [识别实例与数据库 (Identify Instances and Databases)](#identify-instances-and-databases)
    - [发现本地 SQL Server 实例](#discover-local-sql-server-instances)
    - [发现域内 SQL Server 实例](#discover-domain-sql-server-instances)
    - [发现远程 SQL Server 实例](#discover-remote-sql-server-instances)
    - [识别加密数据库](#identify-encrypted-databases)
    - [版本查询](#version-query)
- [识别用户与角色 (Identify Users and Roles)](#identify-users-and-roles)
- [识别敏感信息 (Identify Sensitive Information)](#identify-sensitive-information)
    - [从特定数据库获取表名](#get-tables-from-a-specific-database)
    - [从每个列中收集 5 条条目](#gather-5-entries-from-each-column)
    - [从特定表中收集 5 条条目](#gather-5-entries-from-a-specific-table)
    - [将服务器常用信息转储到文件](#dump-common-information-from-server-to-files)

## 工具 (Tools)

- [NetSPI/PowerUpSQL](https://github.com/NetSPI/PowerUpSQL) - 一个用于攻击 SQL Server 的 PowerShell 工具包。
- [skahwah/SQLRecon](https://github.com/skahwah/SQLRecon/) - 一个专为攻击性侦察和后期利用设计的 C# MS SQL 工具包。

## 识别实例与数据库 (Identify Instances and Databases)

### 发现本地 SQL Server 实例

```ps1
Get-SQLInstanceLocal
```

### 发现域内 SQL Server 实例

```ps1
Get-SQLInstanceDomain -Verbose
# 获取已发现实例的服务器信息
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
# 获取数据库名称
Get-SQLInstanceDomain | Get-SQLDatabase -NoDefaults
```

### 发现远程 SQL Server 实例

```ps1
Get-SQLInstanceBroadcast -Verbose
Get-SQLInstanceScanUDPThreaded -Verbose -ComputerName SQLServer1
```

### 识别加密数据库

注意：这些数据库会自动为管理员解密。

```ps1
Get-SQLDatabase -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Verbose | Where-Object {$_.is_encrypted -eq "True"}
```

### 版本查询

```ps1
Get-SQLInstanceDomain | Get-Query "select @@version"
```

## 识别用户与角色 (Identify Users and Roles)

- 查询当前用户并确定其是否为 sysadmin (系统管理员)

    ```sql
    select suser_sname()
    Select system_user
    select is_srvrolemember('sysadmin')
    ```

- 当前角色

    ```sql
    select user
    ```

- 服务器上的所有登录名

    ```sql
    Select * from sys.server_principals where type_desc != 'SERVER_ROLE'
    ```

- 数据库的所有数据库用户

    ```sql
    Select * from sys.database_principals where type_desc != 'database_role';
    ```

- 列出所有 Sysadmins (系统管理员)

    ```sql
    SELECT name,type_desc,is_disabled FROM sys.server_principals WHERE IS_SRVROLEMEMBER ('sysadmin',name) = 1
    ```

- 列出所有数据库角色 (Database Roles)

    ```sql
    SELECT DB1.name AS DatabaseRoleName,
    isnull (DB2.name, 'No members') AS DatabaseUserName
    FROM sys.database_role_members AS DRM
    RIGHT OUTER JOIN sys.database_principals AS DB1
    ON DRM.role_principal_id = DB1.principal_id
    LEFT OUTER JOIN sys.database_principals AS DB2
    ON DRM.member_principal_id = DB2.principal_id
    WHERE DB1.type = 'R'
    ORDER BY DB1.name;
    ```

## 识别敏感信息 (Identify Sensitive Information)

### 从特定数据库获取表名

```ps1
Get-SQLInstanceDomain | Get-SQLTable -DatabaseName <来自Get-SQLDatabase命令的数据库名> -NoDefaults
# 从表中获取列详细信息
Get-SQLInstanceDomain | Get-SQLColumn -DatabaseName <数据库名> -TableName <表名>
```

- 当前数据库

    ```sql
    select db_name()
    ```

- 列出所有表

    ```sql
    select table_name from information_schema.tables
    ```

- 列出所有数据库

    ```sql
    select name from master..sysdatabases
    ```

- 列出服务器信息

    ```sql
    SELECT * FROM sys.configurations
    ```

### 从每个列中收集 5 条条目

```ps1
Get-SQLInstanceDomain | Get-SQLColumnSampleData -Keywords "<列名1,列名2,列名3,列名4,列名5>" -Verbose -SampleSize 5
```

### 从特定表中收集 5 条条目

```ps1
Get-SQLQuery -Instance "<DBSERVERNAME\DBInstance>" -Query 'select TOP 5 * from <数据库名>.dbo.<表名>'
```

### 将服务器常用信息转储到文件

```ps1
Invoke-SQLDumpInfo -Verbose -Instance SQLSERVER1\Instance1 -csv
```

## 参考资料 (References)

- [PowerUpSQL Cheat Sheet & SQL Server Queries - Leo Pitt](https://medium.com/@D00MFist/powerupsql-cheat-sheet-sql-server-queries-40e1c418edc3)
- [PowerUpSQL Cheat Sheet - Scott Sutherland](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet)
