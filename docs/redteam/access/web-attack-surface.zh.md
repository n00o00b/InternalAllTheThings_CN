# Web 攻击面 (Web Attack Surface)

## 目录 (Summary)

* [枚举子域名 (Enumerate Subdomains)](#enumerate-subdomains)
    * [子域名数据库 (Subdomains Databases)](#subdomains-databases)
    * [爆破子域名 (Bruteforce Subdomains)](#bruteforce-subdomains)
    * [证书透明度日志 (Certificate Transparency Logs)](#certificate-transparency-logs)
    * [DNS 解析 (DNS Resolution)](#dns-resolution)
    * [技术探测 (Technology Discovery)](#technology-discovery)
* [子域名接管 (Subdomain Takeover)](#subdomain-takover)
* [参考资料 (References)](#references)

## 枚举子域名 (Enumerate Subdomains)

子域名枚举是识别与主域名相关联的所有子域名的过程（例如，为 `example.com` 查找 `blog.example.com`、`shop.example.com` 等）。

### 子域名数据库

许多数据库和工具聚合了来自各种在线资源的数据，如 DNS 数据库、证书透明度日志、API（例如 Shodan、VirusTotal）和其他公开可用的资源，以编制一份全面的潜在子域名列表。

* [projectdiscovery/chaos-client](https://github.com/projectdiscovery/chaos-client) - 用于与 Chaos DB API 通信的 Go 客户端。

  ```ps1
  chaos -d hackerone.com
  ```

* [projectdiscovery/subfinder](https://github.com/projectdiscovery/subfinder) - 快速的被动子域名枚举工具。

  ```ps1
  subfinder -d hackerone.com
  ```

* [owasp-amass/amass](https://github.com/owasp-amass/amass) - 深入的攻击面映射和资产发现工具。

  ```ps1
  amass enum -d example.com
  ```

* [Findomain/Findomain](https://github.com/Findomain/Findomain) - 完整的域名识别解决方案。

  ```ps1
  findomain -t example.com -u /tmp/example.com.out
  ```

### 爆破子域名 (Bruteforce Subdomains)

子域名爆破是一种通过针对目标域名系统地尝试潜在子域名名称来发现子域名的技术。这是通过使用预定义的常用或可能的子域名名称列表（称为字典，Wordlist）来完成的。字典中的每个单词都会附加到目标域名（例如 admin.example.com、mail.example.com）后，以检查它是否解析为有效的子域名。

* [assetnote/wordlists](https://github.com/assetnote/wordlists)
* [danielmiessler/SecLists/Discovery/DNS](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS)
* [jhaddix/all.txt](https://gist.github.com/jhaddix/f64c97d0863a78454e44c2f7119c2a6a)

与依赖于现有数据源的被动子域名枚举不同，爆破会主动查询 DNS 记录，以发现可能未列在公共数据库中的活动子域名。

* [infosec-au/altdns](https://github.com/infosec-au/altdns) - 生成子域名的排列、更改和变异，然后对它们进行解析。

  ```powershell
  altdns.py -i /tmp/inputdomains.txt -o /tmp/out.txt -w ./words.txt
  ```

* [owasp-amass/amass](https://github.com/owasp-amass/amass) - 深入的攻击面映射和资产发现工具。

  ```ps1
  amass enum -active -brute -o /tmp/hosts.txt -d $1
  ```

* [projectdiscovery/dnsx](https://github.com/projectdiscovery/dnsx) - 一个快速且多用途的 DNS 工具包，允许你使用用户提供的解析服务器 (resolvers) 运行多个你选择的 DNS 查询。

  ```ps1
  dnsx -silent -d facebook.com -w dns_worldlist.txt
  ```

* [subfinder/goaltdns](https://github.com/subfinder/goaltdns) - 一个用 Go 语言编写的排列生成工具。

  ```ps1
  altdns -l ./input_domains.txt -o ./output.txt
  ```

### 证书透明度日志 (Certificate Transparency Logs)

证书透明度 (CT) 日志是记录由证书颁发机构 (CA) 颁发的所有 SSL/TLS 证书的公共数据库。这些日志旨在通过使证书的监控和审计变得更加容易，来提高 SSL/TLS 生态系统的安全性和透明度。

* [CertStream Calidog](https://certstream.calidog.io/)
* [Meta Certificate Transparency](https://developers.facebook.com/docs/certificate-transparency)
* [Google Certificate Transparency](certificate.transparency.dev)

### DNS 解析 (DNS Resolution)

生成潜在子域名列表后，下一步是将它们解析为 DNS 记录（A 和 AAAA），以获取它们的 IPv4 和 IPv6 地址。

* [blechschmidt/massdns](https://github.com/blechschmidt/massdns)

  ```ps1
  cat /tmp/results_subfinder.txt | massdns -r ./resolvers.txt -t A -o S -w /tmp/results_subfinder_resolved.txt
  ```

* [projectdiscovery/dnsx](https://github.com/projectdiscovery/dnsx) - 一个快速且多用途的 DNS 工具包，允许你使用用户提供的解析服务器运行多个你选择的 DNS 查询。

  ```ps1
  subfinder -silent -d hackerone.com | dnsx -silent -a -resp
  subfinder -silent -d hackerone.com | dnsx -silent -cname -resp
  subfinder -silent -d hackerone.com | dnsx -silent  -asn
  echo 173.0.84.0/24 | dnsx -silent -resp-only -ptr
  echo AS17012 | dnsx -silent -resp-only -ptr 
  ```

## 技术探测 (Technology Discovery)

技术探测是识别网站或数字基础设施所使用的底层技术、软件和框架的过程。这通常包括探测 Web 服务器、CMS 平台、编程语言、数据库、JavaScript 库和其他软件组件。

* [projectdiscovery/httpx](https://github.com/projectdiscovery/httpx) - 快速且多用途的 HTTP 工具包，允许使用 retryablehttp 库运行多个探测。

  ```ps1
  httpx -u 'https://example.com' -title -tech-detect -status-code -follow-redirects
  ```

* [projectdiscovery/wappalyzergo](https://github.com/projectdiscovery/wappalyzergo) - Wappalyzer 技术探测库的高性能 Go 实现。
* [michenriksen/aquatone](https://github.com/michenriksen/aquatone) - 用于域名概览 (Domain Flyovers) 的工具。

  ```ps1
  cat hosts.txt | aquatone -ports 80,443,3000,3001
  ```

* [rverton/webanalyze](https://github.com/rverton/webanalyze) - Wappalyzer 的 Go 语言移植版本。

  ```ps1
  webanalyze -host example.com -crawl 1
  ```

* [wappalyzer](https://www.wappalyzer.com/) - 识别网站上的技术。

## 子域名接管 (Subdomain Takeover)

子域名接管是一种安全漏洞类型，发生在子域名（如 `sub.example.com`）虽然仍处于活动状态，但其 DNS 记录指向的服务或平台（如 AWS S3、GitHub Pages 或 Heroku）已不再活动或未正确配置时。这种情况允许攻击者声明该未被声明的资源并控制子域名，从而能够托管恶意内容或冒充合法网站。

例如，如果 `sub.example.com` 指向一个已被删除或弃用的 AWS S3 存储桶 (bucket)，攻击者可以创建一个同名的新 S3 存储桶，从而获得对该子域名的控制权，并可能导致安全风险，如钓鱼攻击或对主域名造成声誉损害。

有关服务列表和声称具有挂空 (dangling) DNS 记录的子域名的指南，请参阅 [EdOverflow/can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz)。

* [projectdiscovery/nuclei-templates/http/takeovers](https://github.com/projectdiscovery/nuclei-templates/tree/main/http/takeovers) - 社区精选的 nuclei 引擎模板列表，用于查找安全漏洞。

    ```powershell
    nuclei -t nuclei-templates/http/takeovers -u https://example.com
    ```

* [anshumanbh/tko-subs](https://github.com/anshumanbh/tko-subs) - 帮助探测并接管具有无效 DNS 记录的子域名的工具。

    ```powershell
    ./bin/tko-subs -domains=./lists/domains_tkos.txt -data=./lists/providers-data.csv  
    ```

## 参考资料 (References)

* [Subdomain Takeover: Proof Creation for Bug Bounties - Patrik Hudak (@0xpatrik) - May 21, 2018](https://0xpatrik.com/takeover-proofs/)
* [Subdomain Takeover: Basics - Patrik Hudak (@0xpatrik) - June 27, 2018](https://0xpatrik.com/subdomain-takeover-basics/)
