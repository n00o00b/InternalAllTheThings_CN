# 网络探测 (Network Discovery)

## MAC 地址

* [mac2vendor.com](https://mac2vendor.com/) —— OUI 数据库查询
* [oui.is](https://oui.is/) —— MAC 地址厂商查询

| MAC 前缀 | 描述 |
| ---------- | --------------------- |
| FC:D4:F2   | Coca Cola Company     |
| 00:9E:C8   | Xiaomi Communications |
| 08:9E:08   | Google                |

```ps1
sudo ifconfig <网卡名称> down
sudo ifconfig <网卡名称> hw ether <新MAC地址> 
sudo ifconfig <网卡名称> up
```

## DHCP

DHCP (动态主机配置协议) 是一种网络协议，用于自动为网络设备分配 IP 地址和其他网络配置参数。DHCP 允许设备从 DHCP 服务器获取必要的网络配置信息，而无需手动配置。

```ps1
sudo nmap --script broadcast-dhcp-discover
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-04 11:15 CET
Pre-scan script results:
| broadcast-dhcp-discover: 
|   Response 1 of 1: 
|     Interface: eth0
|     IP Offered: 192.168.1.111
|     DHCP Message Type: DHCPOFFER
|     Server Identifier: 192.168.1.254
|     IP Address Lease Time: 1d00h00m00s
|     Renewal Time Value: 12h00m00s
|     Rebinding Time Value: 21h00m00s
|     Broadcast Address: 192.168.1.255
|     Hostname: Host-005
|     Domain Name Server: 192.168.1.254
|     Domain Name: lan
|     Router: 192.168.1.254
|_    Subnet Mask: 255.255.255.0
```

## DNS

* AD DNS
    * LDAP: `nslookup -type=srv _ldap._tcp.dc._msdcs.<域名>`
    * KDC: `nslookup -type=srv _kerberos._tcp.<域名>`
    * 全局编目 (Global catalog): `nslookup -type=srv _ldap._tcp.<域名>`

## NBT-NS

NS (名称服务) 是 NBT 的一个组件，为 NETBIOS 名称提供名称解析服务。在 NBT 的上下文中，NS 负责将 NETBIOS 名称映射到 IP 地址。

NBT NS 使用分布式数据库来存储 NETBIOS 名称与 IP 地址的映射关系。网络中的每台计算机负责在数据库中注册自己的名称和 IP 地址，并在需要时将名称解析为 IP 地址。当计算机需要将 NETBIOS 名称解析为 IP 地址时，它会向网络中另一台计算机上的 NBT NS 服务发送查询请求。如果已知该名称，NBT NS 服务将返回与之对应的 IP 地址。它基于 `UDP 协议，端口 137` 工作。

* 获取名称：`nbtscan -r 192.168.1.0/24`
* 获取单个 IP 的名称：`nmblookup -A <IP>`

## MDNS

MDNS (多播域名系统) 是一种用于零配置网络（也称为 "zeroconf"）的协议。它允许本地网络中的设备自动发现彼此并将主机名解析为 IP 地址，无需集中式的 DNS 服务器。

MDNS 通过使用多播地址发送 DNS 查询和响应来工作。当设备想要将主机名解析为 IP 地址时，它会向特定的多播地址（IPv4 为 224.0.0.251，IPv6 为 ff02::fb）发送多播 DNS 查询。网络中任何正在监听多播 DNS 查询且主机名匹配的设备都会返回其 IP 地址。

```ps1
mdns-scan
```

## ARP

ARP (地址解析协议) 是一种网络协议，用于在局域网 (LAN) 中将 IP 地址映射到 MAC (媒体访问控制) 地址。

* ARP 邻居

    ```ps1
    :~$ ip neigh
    192.168.122.1 dev enp1s0 lladdr 52:54:00:ff:0a:2c STALE
    192.168.122.98 dev enp1s0 lladdr 52:54:00:ff:aa:bb STALE
    ```

* 使用 `nmap` 进行 ARP 扫描 —— 注意：需要 root 权限。使用 `--packet-trace` 可以检查 nmap 发送的包。

    ```ps1
    :~# nmap -sn -n 192.168.122.0/24 
    Starting Nmap 7.93 ( https://nmap.org )
    Nmap scan report for 192.168.122.1
    Host is up (0.00032s latency).
    MAC Address: 52:54:00:FF:0A:2C (QEMU virtual NIC)
    ```

* 使用 `arp-scan` 进行 ARP 扫描

    ```ps1
    root@kali:~# arp-scan -l
    Interface: eth0, datalink type: EN10MB (Ethernet)
    Starting arp-scan 1.9 with 256 hosts (http://www.nta-monitor.com/tools/arp-scan/)
    172.16.193.1 00:50:56:c0:00:08 VMware, Inc.
    172.16.193.2 00:50:56:f1:18:a8 VMware, Inc.
    172.16.193.254 00:50:56:e5:7b:87 VMware, Inc.
    ```

* 使用 `arpspoof` 进行 ARP 欺骗

    ```ps1
    arpspoof [-i 网卡名] [-c own|host|both] [-t 目标] [-r] 主机
    arpspoof -i wlan0 -t 10.0.0.X 10.0.0.Y
    ```

* 使用 `Bettercap` 进行 ARP 欺骗

    ```ps1
    sudo bettercap -iface wlan0
    net.probe on
    set arp.spoof.targets <目标IP>
    arp.spoof on
    net.sniff on
    ```

## Ping

* 使用 `nmap` 进行 Ping 扫描：不扫描端口，不进行 DNS 解析。

    ```powershell
    nmap -sn -n --disable-arp-ping 192.168.1.1-254 | grep -v "host down"
    -sn : 禁用端口扫描。仅进行主机发现。
    -n : 从不进行 DNS 解析
    ```

## LDAP

* 匿名绑定 (Null bind) 连接：`ldapsearch -x -h <IP> -s base`

## 端口扫描与枚举 (Port Scans and Enumeration)

### Nmap

* 基础型 NMAP 扫描

```bash
sudo nmap -sSV -p- 192.168.0.1 -oA 输出文件名 -T4
sudo nmap -sSV -oA 输出文件名 -T4 -iL 输入文件.csv

• -sSV 参数定义了发送到服务器的包类型，并指示 Nmap 尝试确定开放端口上的任何服务
• -p- 参数指示 Nmap 检查所有 65,535 个端口（默认仅检查最常用的 1,000 个）
• 192.168.0.1 是待扫描的 IP 地址
• -oA 输出文件名 指示 Nmap 使用文件名“输出文件名”同时以其三种主要格式输出结果
• -iL 输入文件 指示 Nmap 使用提供的文件作为输入源
```

* CTF 比赛型 NMAP 扫描

此配置足以对 CTF 虚拟机进行基础检查：

```bash
nmap -sV -sC -oA ~/nmap-initial 192.168.1.1

-sV : 探测开放端口以确定服务/版本信息
-sC : 启用默认脚本
-oA : 保存结果

在执行此快速命令后，你可以添加 "-p-" 运行全端口扫描，同时利用已有结果开始工作。
```

* 强力型 NMAP 扫描

```bash
nmap -A -T4 scanme.nmap.org
• -A: 启用操作系统探测、版本探测、脚本扫描和路由追踪
• -T4: 定义任务的时序（取值 0-5，越高越快）
```

* 使用 searchsploit 探测漏洞服务

```bash
nmap -p- -sV -oX a.xml IP地址; searchsploit --nmap a.xml
```

* 生成精美的扫描报告

```bash
nmap -sV IP地址 -oX scan.xml && xsltproc scan.xml -o "`date +%m%d%y`_report.html"
```

* NMAP 脚本 (Scripts)

```bash
nmap -sC : 等同于 --script=default

nmap --script 'http-enum' -v web.xxxx.com -p80 -oN http-enum.nmap
PORT   STATE SERVICE
80/tcp open  http
| http-enum:
|   /phpmyadmin/: phpMyAdmin
|   /.git/HEAD: Git 文件夹
|   /css/: 潜在有趣的目录，列出于 'apache/2.4.10 (debian)'
|_  /image/: 潜在有趣的目录，列出于 'apache/2.4.10 (debian)'

nmap --script smb-enum-users.nse -p 445 [目标主机]
Host script results:
| smb-enum-users:
|   METASPLOITABLE\backup (RID: 1068)
|     Full name:   backup
|     Flags:       Account disabled, Normal user account
|   METASPLOITABLE\bin (RID: 1004)
|     Full name:   bin
|     Flags:       Account disabled, Normal user account
|   METASPLOITABLE\msfadmin (RID: 3000)
|     Full name:   msfadmin,,,
|     Flags:       Normal user account

列出 Nmap 脚本：ls /usr/share/nmap/scripts/
```

### 使用 nc 和 ping 进行网络扫描

有时我们希望在不使用 nmap 等工具的情况下进行网络扫描。我们可以使用 `ping` 和 `nc` 命令来检查主机是否在线以及哪些端口是开放的。

检查 /24 范围内的在线主机：

```bash
for i in `seq 1 255`; do ping -c 1 -w 1 192.168.1.$i > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "192.168.1.$i is UP"; fi ; done
```

检查特定主机的开放端口：

```bash
for i in {21,22,80,139,443,445,3306,3389,8080,8443}; do nc -z -w 1 192.168.1.18 $i > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "192.168.1.18 has port $i open"; fi ; done
```

同时对 /24 范围进行以上两项检查：

```bash
for i in `seq 1 255`; do ping -c 1 -w 1 192.168.1.$i > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "192.168.1.$i is UP:"; for j in {21,22,80,139,443,445,3306,3389,8080,8443}; do nc -z -w 1 192.168.1.$i $j > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "\t192.168.1.$i has port $j open"; fi ; done ; fi ; done
```

非单行命令版本：

```bash
for i in `seq 1 255`; 
do 
    ping -c 1 -w 1 192.168.1.$i > /dev/null 2>&1; 
    if [ $? -eq 0 ]; 
    then 
        echo "192.168.1.$i is UP:"; 
        for j in {21,22,80,139,443,445,3306,3389,8080,8443}; 
        do 
            nc -z -w 1 192.168.1.$i $j > /dev/null 2>&1; 
            if [ $? -eq 0 ]; 
            then 
                echo "\t192.168.1.$i has port $j open"; 
            fi ; 
        done ; 
    fi ; 
done
```

### 使用 PowerShell 进行网络扫描

```powershell
# ping 扫描探测
tnc 8.8.8.8

# 端口扫描探测
tnc 8.8.8.8 -port 443
```

### Masscan

```powershell
masscan -iL ips-online.txt --rate 10000 -p1-65535 --only-open -oL masscan.out
masscan -e tun0 -p1-65535,U:1-65535 10.10.10.97 --rate 1000

# 寻找网络中的机器
sudo masscan --rate 500 --interface tap0 --router-ip $ROUTER_IP --top-ports 100 $NETWORK -oL masscan_machines.tmp
cat masscan_machines.tmp | grep open | cut -d " " -f4 | sort -u > masscan_machines.lst

# 寻找单台主机的开放端口
sudo masscan --rate 1000 --interface tap0 --router-ip $ROUTER_IP -p1-65535,U:1-65535 $MACHINE_IP --banners -oL $MACHINE_IP/scans/masscan-ports.lst


# 获取 TCP Banner 和服务信息
TCP_PORTS=$(cat $MACHINE_IP/scans/masscan-ports.lst| grep open | grep tcp | cut -d " " -f3 | tr '\n' ',' | head -c -1)
[ "$TCP_PORTS" ] && sudo nmap -sT -sC -sV -v -Pn -n -T4 -p$TCP_PORTS --reason --version-intensity=5 -oA $MACHINE_IP/scans/nmap_tcp $MACHINE_IP

# 获取 UDP Banner 和服务信息
UDP_PORTS=$(cat $MACHINE_IP/scans/masscan-ports.lst| grep open | grep udp | cut -d " " -f3 | tr '\n' ',' | head -c -1)
[ "$UDP_PORTS" ] && sudo nmap -sU -sC -sV -v -Pn -n -T4 -p$UDP_PORTS --reason --version-intensity=5 -oA $MACHINE_IP/scans/nmap_udp $MACHINE_IP
```

### Reconnoitre

依赖项：

* nbtscan
* nmap

```powershell
python2.7 ./reconnoitre.py -t 192.168.1.2-252 -o ./results/ --pingsweep --hostnames --services --quick
```

如果运行 nbtscan 出现段错误 (segfault)，请参阅以下引用。
> 在广播地址 (.0) 上权限被拒绝，在网关 (.1) 上出现段错误 —— 此处所有其他地址似乎都正常。缓解该问题的方法：nbtscan 192.168.0.2-255

## Netdiscover

```powershell
netdiscover -i eth0 -r 192.168.1.0/24
Currently scanning: Finished!   |   Screen View: Unique Hosts

20 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 876
_____________________________________________________________________________
IP            At MAC Address     Count     Len  MAC Vendor / Hostname
-----------------------------------------------------------------------------
192.168.1.AA    68:AA:AA:AA:AA:AA     15     630  Sagemcom
192.168.1.XX    52:XX:XX:XX:XX:XX      1      60  Unknown vendor
192.168.1.YY    24:YY:YY:YY:YY:YY      1      60  QNAP Systems, Inc.
192.168.1.ZZ    b8:ZZ:ZZ:ZZ:ZZ:ZZ      3     126  HUAWEI TECHNOLOGIES CO.,LTD  
```

## Responder

```powershell
responder -I eth0 -A # 查看 NBT-NS, BROWSER, LLMNR 请求，但不响应。
responder.py -I eth0 -wrf
```

此外，你也可以使用 [Windows 版本](https://github.com/lgandx/Responder-Windows)。

## MITM (中间人攻击)

* WSUS 投毒
* ARP 投毒
* DHCP 投毒：`responder --interface "eth0" --DHCP --wpad`

### Bettercap

```powershell
bettercap -X --proxy --proxy-https -T <目标IP>
# 使用 bettercap 进行欺骗、探测和嗅探
# 拦截 HTTP 和 HTTPS 请求，
# 仅针对特定 IP
```

### 使用 OpenSSL 进行 SSL MITM

如果你能修改客户端的 `/etc/hosts` 文件，该代码片段允许你利用 OpenSSL 单独探测/修改存在 MITM 漏洞的 SSL 流量。

```powershell
sudo echo "[OPENSSL服务器地址] [待劫持的目标域名]" >> /etc/hosts  # 在客户端主机上执行
```

在我们的 MITM 服务器上，如果客户端接受自签名证书（如果你拥有合法服务器的私钥，也可以使用合法证书）：

```powershell
openssl req -subj '/CN=[待劫持的目标域名]' -batch -new -x509 -days 365 -nodes -out server.pem -keyout server.pem
```

在我们的 MITM 服务器上搭建基础设施：

```powershell
mkfifo response
sudo openssl s_server -cert server.pem -accept [监听网卡地址]:[端口] -quiet < response | tee | openssl s_client -quiet -servername [待劫持的目标域名] -connect [待劫持的目标IP]:[端口] | tee | cat > response
```

在此示例中，流量仅使用 `tee` 显示，但我们也可以使用 `sed` 等工具对其进行修改。

## 参考资料 (References)

* [Pwning the Domain: Credentialess/Username - hadess - February 7, 2024](https://hadess.io/pwning-the-domain-credentialess-username/)
