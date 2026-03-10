# Active Directory - 技巧

## Kerberos 时钟同步

在 Kerberos 中，时间用于确保票据有效。为此，域中所有 Kerberos 客户端和服务器的时钟必须在一定容差范围内同步。Kerberos 中的默认时钟偏差容差为 `5 分钟`，这意味着任何两个 Kerberos 实体的时钟时间差不应超过 5 分钟。

* 使用 `nmap` 自动检测时钟偏差

  ```powershell
  $ nmap -sV -sC 10.10.10.10
  clock-skew: mean: -1998d09h03m04s, deviation: 4h00m00s, median: -1998d11h03m05s
  ```

* 自己计算时钟差异

  ```ps1
  nmap -sT 10.10.10.10 -p445 --script smb2-time -vv
  ```

* 修复方法 1：修改你的时钟

  ```ps1
  sudo date -s "14 APR 2015 18:25:16" # Linux
  net time /domain /set # Windows
  ```

* 修复方法 2：伪造你的时钟

  ```ps1
  faketime -f '+8h' date
  ```

## 参考资料

* [BUILDING AND ATTACKING AN ACTIVE DIRECTORY LAB WITH POWERSHELL - @myexploit2600 & @5ub34x](https://1337red.wordpress.com/building-and-attacking-an-active-directory-lab-with-powershell/)
* [Becoming Darth Sidious: Creating a Windows Domain (Active Directory) and hacking it - @chryzsh](https://chryzsh.gitbooks.io/darthsidious/content/building-a-lab/building-a-lab/building-a-small-lab.html)
* [Chump2Trump - AD Privesc talk at WAHCKon 2017 - @l0ss](https://github.com/l0ss/Chump2Trump/blob/master/ChumpToTrump.pdf)
* [How to build a SQL Server Virtual Lab with AutomatedLab in Hyper-V - October 30, 2017 - Craig Porteous](https://www.sqlshack.com/build-sql-server-virtual-lab-automatedlab-hyper-v/)
