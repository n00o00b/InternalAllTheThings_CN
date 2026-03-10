# 信任 - 信任关系 (Trust-Relationship)

- 单向 (One-way)
    - 域 B 信任 域 A (Domain B trusts A)
    - 域 A 中的用户可以访问域 B 中的资源 (Users in Domain A can access resources in Domain B)
    - 域 B 中的用户无法访问域 A 中的资源 (Users in Domain B cannot access resources in Domain A)
- 双向 (Two-way)
    - 域 A 信任 域 B (Domain A trusts Domain B)
    - 域 B 信任 域 A (Domain B trusts Domain A)
    - 身份验证请求可以在两个域之间双向传递 (Authentication requests can be passed between the two domains in both directions)

## 枚举域之间的信任关系 (Enumerate trusts between domains)

- 原生 `nltest` 命令

  ```powershell
  nltest /trusted_domains
  ```

- PowerShell 命令 `GetAllTrustRelationships`

  ```powershell
  ([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()

  SourceName          TargetName                    TrustType      TrustDirection
  ----------          ----------                    ---------      --------------
  domainA.local      domainB.local                  TreeRoot       Bidirectional
  ```

- netexec 模块 `enum_trusts`

  ```powershell
  nxc ldap <ip> -u <user> -p <pass> -M enum_trusts 
  ```

## 利用域之间的信任关系 (Exploit trusts between domains)

:warning: 需要具有对当前域的 Domain-Admin (域管理员) 级别访问权限。

| 来源 (Source) | 目标 (Target) | 使用的技术 (Technique to use) | 信任关系 (Trust relationship) |
|---|---|---|---|
| 林根域 (Root) | 子域 (Child)  | 黄金票据 (Golden Ticket) + 企业管理员组 (Enterprise Admin group) (Mimikatz /groups) | 域内信任 (Inter Realm) (双向 2-way)  |
| 子域 (Child)  | 子域 (Child)  | SID 历史记录利用 (SID History exploitation) (Mimikatz /sids)                 | 父子域间信任 (Inter Realm Parent-Child) (双向 2-way)  |
| 子域 (Child)  | 林根域 (Root) | SID 历史记录利用 (SID History exploitation) (Mimikatz /sids)                 | 树根域间信任 (Inter Realm Tree-Root) (双向 2-way)  |
| 林 A (Forest A)| 林 B (Forest B)| PrinterBug + 非约束委派 (Unconstrained delegation) ? | 林间信任或外部信任 (Inter Realm Forest or External) (双向 2-way) |

## 参考资料 (References)

- [External Trusts Are Evil - 14 March 2023 - Charlie Clark (@exploitph)](https://exploit.ph/external-trusts-are-evil.html)
- [Carlos Garcia - Rooted2019 - Pentesting Active Directory Forests public.pdf](https://www.dropbox.com/s/ilzjtlo0vbyu1u0/Carlos%20Garcia%20-%20Rooted2019%20-%20Pentesting%20Active%20Directory%20Forests%20public.pdf?dl=0)
- [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
