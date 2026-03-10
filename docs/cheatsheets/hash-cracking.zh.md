# 哈希破解 (Hash Cracking)

## 目录 (Summary)

* [Hashcat](https://hashcat.net/hashcat/)
    * [Hashcat 示例哈希](https://hashcat.net/wiki/doku.php?id=example_hashes)
    * [Hashcat 安装](#hashcat-%e5%ae%89%e8%a3%85-hashcat-install)
    * [掩码攻击 (Mask attack)](#%e6%8e%a9%e7%a0%81%e6%94%bb%e5%87%bb-mask-attack)
    * [字典攻击 (Dictionary)](#%e5%ad%97%e5%85%b8%e6%94%bb%e5%87%bb-dictionary)
* [John the Ripper (John)](https://github.com/openwall/john)
    * [常用用法](#john-%e5%b8%b8%e7%94%a8%e7%94%a8%e6%b3%95-john-usage)
* [彩虹表 (Rainbow tables)](#%e5%bd%a9%e8%99%b9%e8%a1%a8-rainbow-tables)
* [技巧与心得 (Tips and Tricks)](#%e6%8a%80%e5%b7%a7%e4%b8%8e%e5%bf%83%e5%be%97-tips-and-tricks)
* [在线破解资源 (Online Cracking Resources)](#%e5%9c%a8%e7%ba%bf%e7%a0%b4%e8%a7%a3%e8%b5%84%e6%ba%90-online-cracking-resources)
* [参考资料 (References)](#%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99-references)

## Hashcat

### Hashcat 安装 (Hashcat Install)

```powershell
apt install cmake build-essential -y
apt install checkinstall git -y
git clone https://github.com/hashcat/hashcat.git && cd hashcat && make -j 8 && make install
```

1. 提取哈希。
2. 确定哈希格式：[hashcat.net/example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)
3. 根据哈希格式建立破解策略（如：字典 -> 字典+规则 -> 掩码 -> 组合模式 -> Prince 攻击 -> ...）。
4. 坐等明文。
5. 审查策略。
6. 必要时重新开始。

### 字典攻击 (Dictionary)

> 将给定列表（即字典）中的每个单词进行哈希计算，并与目标哈希进行比较。

```powershell
hashcat --attack-mode 0 --hash-type $number $hashes_file $wordlist_file -r $my_rules
```

* 字典 (Wordlists)
    * [packetstorm](https://packetstormsecurity.com/Crackers/wordlists/)
    * [weakpass_3a](https://download.weakpass.com/wordlists/1948/weakpass_3a.7z)
    * [weakpass_3](https://download.weakpass.com/wordlists/1947/weakpass_3.7z)
    * [Hashes.org](https://download.weakpass.com/wordlists/1931/Hashes.org.7z)
    * [kerberoast_pws](https://gist.github.com/edermi/f8b143b11dc020b854178d3809cf91b5/raw/b7d83af6a8bbb43013e04f78328687d19d0cf9a7/kerberoast_pws.xz)
    * [hashmob.net](https://hashmob.net/research/wordlists)
    * [clem9669/wordlists](https://github.com/clem9669/wordlists)

* 规则 (Rules)
    * [One Rule to Rule Them All](https://notsosecure.com/one-rule-to-rule-them-all/)
    * [nsa-rules](https://github.com/NSAKEY/nsa-rules)
    * [hob064](https://raw.githubusercontent.com/praetorian-inc/Hob0Rules/master/hob064.rule)
    * [d3adhob0](https://raw.githubusercontent.com/praetorian-inc/Hob0Rules/master/d3adhob0.rule)
    * [clem9669/hashcat-rule](https://github.com/clem9669/hashcat-rule)

### 掩码攻击 (Mask attack)

掩码攻击是一种旨在优化暴力破解的攻击模式。

> 将给定字符集和长度下的每一种可能性（如 aaa, aab, aac, ...）进行哈希计算，并与目标哈希进行比较。

```powershell
# 掩码：1位大写 + 5位小写 + 2位数字 以及 1位大写 + 6位小写 + 2位数字
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 ?u?l?l?l?l?l?d?d
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 ?u?l?l?l?l?l?l?d?d 
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 -1 "*+!??" ?u?l?l?l?l?l?d?d?1
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 -1 "*+!??" ?u?l?l?l?l?l?l?d?d?1 

# 掩码：1位大写 + 3位小写 + 4位数字 以及 1位大写 + 4位小写 + 4位数字
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 ?u?l?l?l?d?d?d?d
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 ?u?l?l?l?l?d?d?d?d
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 ?u?l?l?l?l?l?d?d?d?d
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 -1 "*+!??" ?u?l?l?l?d?d?d?d?1
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 -1 "*+!??" ?u?l?l?l?l?d?d?d?d?1

# 掩码：6位小写 + 2位数字 + 特殊符号(+!?*)
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 -1 "*+!??" ?l?l?l?l?l?l?d?d?1
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 -1 "*+!??" ?l?l?l?l?l?l?d?d?1?1

# 掩码：6位小写 + 2位数字
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 /content/hashcat/masks/8char-1l-1u-1d-1s-compliant.hcmask
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 -1 ?l?d?u ?1?1?1?1?1?1?1?1

# 其他示例
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 ?a?a?a?a?a?a?a?a?a
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 ?a?a?a?a?a?a?a?a 
hashcat -m 1000 --status --status-timer 300 -w 4 -O /content/*.ntds -a 3 ?u?l?l?l?l?l?l?d?d?d?d
hashcat --attack-mode 3 --increment --increment-min 4 --increment-max 8 --hash-type $number $hashes_file "?a?a?a?a?a?a?a?a?a?a?a?a"
hashcat --attack-mode 3 --hash-type $number $hashes_file "?u?l?l?l?d?d?d?d?s"
hashcat --attack-mode 3 --hash-type $number $hashes_file "?a?a?a?a?a?a?a?a"
hashcat --attack-mode 3 --custom-charset1 "?u" --custom-charset2 "?l?u?d" --custom-charset3 "?d" --hash-type $number $hashes_file "?1?2?2?2?3"
```

| 快捷键 | 对应字符  |
|----|----------------------------|
| ?l | 小写字母 (abcdefghijklmnopqrstuvwxyz) |
| ?u | 大写字母 (ABCDEFGHIJKLMNOPQRSTUVWXYZ) |
| ?d | 数字 (0123456789) |
| ?s | 特殊符号 (!"#$%&'()*+,-./:;<=>?@[\]^_`{}~) |
| ?a | 包含以上所有 (?l?u?d?s) |
| ?b | 二进制数据 (0x00 - 0xff) |

## John the Ripper (John)

### John 常用用法 (John Usage)

```bash
# 对包含待破解哈希的密码文件运行
john passwd

# 使用特定字典
john --wordlist=<字典文件路径> passwd

# 使用特定字典并配合规则
john --wordlist=<字典文件路径> passwd --rules=Jumbo

# 显示已破解的密码
john --show passwd

# 恢复中断的会话
john --restore
```

## 彩虹表 (Rainbow tables)

> 在预先计算好的哈希表中查找目标哈希。这是一种通过空间换取时间的方法，可以更快地破解哈希，但比传统的字典或暴力攻击消耗更多的内存。如果哈希值使用了“加盐 (Salted)”处理（即可变随机值作为前缀/后缀），则此攻击无效，因为预计算表会因此失效。

## 技巧与心得 (Tips and Tricks)

* 云端 GPU
    * [penglab - 滥用 Google Colab 进行哈希破解。 🐧](https://github.com/mxrch/penglab)
    * [google-colab-hashcat - 在 Google Colab 上运行 Hashcat](https://github.com/ShutdownRepo/google-colab-hashcat)
    * [Cloudtopolis - 无基础设施的密码破解](https://github.com/JoelGMSec/Cloudtopolis)
    * [Nephelees - 同样也是一款滥用 Google Colab 破解 NTDS 的工具](https://github.com/swisskyrepo/Nephelees)
* 搭建物理破解平台 (Rig)
    * [渗透测试者的便携式破解平台 - 1000 美金级](https://www.netmux.com/blog/portable-cracking-rig)
    * [如何搭建密码破解平台 - 5000 美金级](https://www.netmux.com/blog/how-to-build-a-password-cracking-rig)
* 在线破解
    * [Hashes.com](https://hashes.com/en/decrypt/hash)
    * [hashmob.net](https://hashmob.net/)：非常棒的社区，拥有活跃的 Discord 频道。
* 将 `loopback` 模式与规则及字典结合使用，可以持续破解直到不再发现新密码：`hashcat --loopback --attack-mode 0 --rules-file $rules_file --hash-type $number $hashes_file $wordlist_file`
* PACK (密码分析与破解工具包)
    * [iphelix/pack](https://github.com/iphelix/pack/blob/master/README)
    * 能够基于输入数据集的统计数据和规则生成自定义的 .hcmask 文件，供 Hashcat 使用。
* 使用深度学习 (Deep Learning)
    * [brannondorsey/PassGAN](https://github.com/brannondorsey/PassGAN)

## 在线破解资源 (Online Cracking Resources)

* [hashes.com](https://hashes.com)
* [crackstation.net](https://crackstation.net)
* [hashmob.net](https://hashmob.net/)

## 参考资料 (References)

* [Cracking - The Hacker Recipes](https://www.thehacker.recipes/ad-ds/movement/credentials/cracking)
* [Using Hashcat to Crack Hashes on Azure](https://durdle.com/2017/04/23/using-hashcat-to-crack-hashes-on-azure/)
* [miloserdov.org hashcat 教程](https://miloserdov.org/?p=5426&PageSpeed=noscript)
* [miloserdov.org john 教程](https://miloserdov.org/?p=4961&PageSpeed=noscript)
* [DeepPass — 使用深度学习寻找密码 - Will Schroeder - Jun 1](https://posts.specterops.io/deeppass-finding-passwords-with-deep-learning-4d31c534cd00)
