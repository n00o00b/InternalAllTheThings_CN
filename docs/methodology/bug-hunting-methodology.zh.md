# 漏洞挖掘方法论 (Bug Hunting Methodology)

## 被动侦察 (Passive Recon)

* 使用 [shodan.io](https://www.shodan.io/)、[fofa.info](https://en.fofa.info/)、[zoomeye.ai](https://www.zoomeye.ai/) 或 [odin.io](https://search.odin.io/hosts) 来检测类似的应用。

  ```ps1
  # https://github.com/glennzw/shodan-hq-nse
  nmap --script shodan-hq.nse --script-args 'apikey=<你的ShodanAPI密钥>,target=<目标域名>'
  ```

* 使用相同的 favicon 搜索类似的网站：[pielco11/fav-up](https://github.com/pielco11/fav-up)；或者搜索略有不同的图标：[profundis.io/favicon-matcher](https://profundis.io/tools/favicon-matcher)

  ```ps1
  python3 favUp.py --favicon-file favicon.ico -sc
  python3 favUp.py --favicon-url https://隐藏在Cloudflare后的域名/assets/favicon.ico -sc
  python3 favUp.py --web 隐藏在Cloudflare后的域名 -s
  ```

* 在短链接 (Shortener URLs) 中搜索：[shorteners.grayhatwarfare.com](https://shorteners.grayhatwarfare.com/)、[utkusen/urlhunter](https://github.com/utkusen/urlhunter)

  ```ps1
  urlhunter --keywords keywords.txt --date 2020-11-20
  ```

* 在储存桶 (Buckets) 中搜索：[buckets.grayhatwarfare.com](https://buckets.grayhatwarfare.com/)

* 使用 [往昔之门 (The Wayback Machine)](https://archive.org/web/) 来探测被遗忘的端点。

  ```powershell
  # 寻找 JS 文件、旧链接
  curl -sX GET "http://web.archive.org/cdx/search/cdx?url=<目标域名.com>&output=text&fl=original&collapse=urlkey&matchType=prefix"
  ```

* 使用 [laramies/theHarvester](https://github.com/laramies/theHarvester)

  ```python
  python theHarvester.py -b all -d domain.com
  ```

* 使用 [michenriksen/GitRob](https://github.com/michenriksen/gitrob.git) 在 [GitHub](https://github.com) 仓库中查找私密信息。

  ```bash
  gitrob analyze johndoe --site=https://github.acme.com --endpoint=https://github.acme.com/api/v3 --access-tokens=token1,token2
  ```

* 执行 Google Dorking 搜索：[ikuamike/GoogleDorking.md](https://gist.github.com/ikuamike/c2611b171d64b823c1c1956129cbc055)

  ```ps1
  site: *.example.com -www
  intext:"dhcpd.conf" "index of"
  intitle:"SSL Network Extender Login" -checkpoint.com
  ```

* 使用 HackerTarget 枚举子域名

  ```ps1
  curl --silent 'https://api.hackertarget.com/hostsearch/?q=targetdomain.com' | grep -o '\w.*targetdomain.com'
  ```

* 使用 CommonCrawl 枚举端点

  ```ps1
  echo "targetdomain.com" | xargs -I domain curl -s "http://index.commoncrawl.org/CC-MAIN-2018-22-index?url=*.targetdomain.com&output=json" | jq -r .url | sort -u
  ```

## 主动侦察 (Active Recon)

### 网络发现 (Network Discovery)

* 子域名枚举 (Subdomains enumeration)
    * 枚举已发现的子域名：[projectdiscovery/subfinder](https://github.com/projectdiscovery/subfinder)、[OWASP/Amass](https://github.com/OWASP/Amass)

    ```ps1
    subfinder -d hackerone.com
    amass enum -passive -dir /tmp/amass_output/ -d example.com -o dir/example.com
    ```

    * 子域名置换 (Permutate)：[infosec-au/altdns](https://github.com/infosec-au/altdns)
    * 子域名爆破：[Josue87/gotator](https://github.com/Josue87/gotator)
    * 使用 [blechschmidt/massdns](https://github.com/blechschmidt/massdns) 将子域名解析为 IP，记得使用像 [trickest/resolvers](https://github.com/trickest/resolvers) 这样优质的解析服务器列表。

    ```ps1
    massdns -r resolvers.txt -o S -w massdns.out subdomains.txt
    ```

    * 子域名接管 (Subdomain takeovers)：[EdOverflow/can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz)

* 网络发现
    * 使用 `nmap`、[robertdavidgraham/masscan](https://github.com/robertdavidgraham/masscan) 和 [projectdiscovery/naabu](https://github.com/projectdiscovery/naabu) 扫描 IP 范围。
    * 探测服务、版本和 Banner 信息。

* 审查最新的收购案例 (Acquisitions)

* ASN 枚举
    * [projectdiscovery/asnmap](https://github.com/projectdiscovery/asnmap)：`asnmap -a AS45596 -silent`
    * [asnlookup.com](http://www.asnlookup.com)

* DNS 区域传输 (DNS Zone Transfer)

  ```ps1
  host -t ns domain.local
  domain.local name server master.domain.local.

  host master.domain.local        
  master.domain.local has address 192.168.1.1
 
  dig axfr domain.local @192.168.1.1
  ```

### Web 发现 (Web Discovery)

#### 常见文件

* `security.txt`：一个提供联络信息（如电子邮件或 PGP 密钥）的文件，用于报告网站的安全问题。

  ```ps1
  Contact: mailto:security@example.com
  ```

* `sitemap.xml`：列出网站的所有重要 URL，以便搜索引擎可以有效地索引它们。

  ```ps1
  <urlset>
    <url><loc>https://example.com/</loc></url>
    <url><loc>https://example.com/about</loc></url>
  </urlset>
  ```

* `robots.txt`：告诉搜索引擎爬虫它们可以在你的网站上访问或不能访问哪些页面或文件。

  ```ps1
  User-agent: *
  Disallow: /admin/
  ```

#### 枚举文件与文件夹

枚举所有可访问的文件和子目录。一旦识别出底层技术，就优先使用有针对性的字典，而不是通用字典。使用 Assetnote ([https://wordlists.assetnote.io](https://wordlists.assetnote.io)) 等提供的技术特定字典，能显著提高覆盖率和效率。例如 `httparchive_parameters_top_1m_2026_01_27.txt`、`httparchive_directories_1m_2026_01_27.txt` 和 `httparchive_php_2026_01_27.txt`。

* [OJ/gobuster](https://github.com/OJ/gobuster)
* [ffuf/ffuf](https://github.com/ffuf/ffuf)
* [bitquark/shortscan](https://github.com/bitquark/shortscan)

  ```ps1
  ffuf -H 'User-Agent: Mozilla' -v -t 30 -w mydirfilelist.txt -b 'NAME1=VALUE1; NAME2=VALUE2' -u 'https://example.com/FUZZ'
  gobuster dir -a 'Mozilla' -e -k -l -t 30 -w mydirfilelist.txt -c 'NAME1=VALUE1; NAME2=VALUE2' -u 'https://example.com/'
  ```

识别并枚举可能被无意泄露的备份文件和临时文件。这些文件通常包含源代码、凭据或敏感配置数据，通常由编辑器、部署流程或手动备份创建。

* [mazen160/bfac](https://github.com/mazen160/bfac)

```bash
bfac --url http://example.com/test.php --level 4
bfac --list testing_list.txt
```

爬取网站的页面和资源，以识别额外的攻击面并扩大评估范围。

* [hakluke/hakrawler](https://github.com/hakluke/hakrawler)
* [projectdiscovery/katana](https://github.com/projectdiscovery/katana)

```ps1
katana -u https://tesla.com
echo https://google.com | hakrawler
```

#### Next.js 端点

在 Next.js 中，`window.__BUILD_MANIFEST` 是框架自动注入到客户端 JavaScript 包中的运行时全局变量。

转到 `DevTools->Console` 并执行以下 JavaScript 代码：

```js
console.log(window.__BUILD_MANIFEST)
console.log(__BUILD_MANIFEST.sortedPages)
```

如果在浏览器控制台中检查应用（针对生产构建版本），你可能会看到类似以下的内容：

```js
{__rewrites: {…}, /: Array(10), /404: Array(8), /500: Array(4), /_error: Array(1), …}
/: (10) ['static/chunks/2852872c-b605aca0298c2109.js', 'static/chunks/3748-2a8cf394c7270ee0.js']
/404: (8) ['static/chunks/2852872c-b605aca0298c2109.js', 'static/chunks/3748-2a8cf394c7270ee0.js']
/500: (4) ['static/chunks/3748-2a8cf394c7270ee0.js', 'static/chunks/1221-b44c330d41258365.js']
/[slug]: (30) ['static/chunks/2852872c-b605aca0298c2109.js', 'static/chunks/29107295-4cc022cea922dbb4.js']
/_error: ['static/chunks/pages/_error-6ddff449d199572c.js']
/about/[slug]: (31) ['static/chunks/2852872c-b605aca0298c2109.js']
```

#### JS 与 HTML 注释

检索源代码中的注释。

```html
<!-- HTML 注释 -->
// JS 注释
```

#### 互联网档案 (Internet Archive)

通过审查来自 Wayback Machine 和 Internet Archive 等来源的存档内容，识别历史 URL 和端点。

* [tomnomnom/waybackurls](https://github.com/tomnomnom/waybackurls)
* [lc/gau](https://github.com/lc/gau)

```ps1
gau --o example-urls.txt example.com
gau --blacklist png,jpg,gif example.com
```

#### 隐藏参数 (Hidden Parameters)

搜索“隐藏”参数：

* [PortSwigger/param-miner](https://github.com/PortSwigger/param-miner)
* [s0md3v/Arjun](https://github.com/s0md3v/Arjun)
* [Sh1Yo/x8](https://github.com/Sh1Yo/x8)

  ```ps1
  x8 -u "https://example.com/?something=1" -w <wordlist>
  ```

#### 技术栈映射 (Map Technologies)

* 使用 [projectdiscovery/httpx](https://github.com/projectdiscovery/httpx) 或 [projectdiscovery/wappalyzergo](https://github.com/projectdiscovery/wappalyzergo) 枚举 Web 服务
    * 网站图标 (Favicon) 哈希
    * JARM 指纹
    * ASN
    * 状态码
    * 服务
    * 技术（Github Pages, Cloudflare, Ruby, Nginx 等）

    ```ps1
    httpx -title -tech-detect -status-code -follow-redirects -jarm -asn -json -silent -ports 80,443 -l urls.txt
    ```

* 使用 [projectdiscovery/cdncheck](https://github.com/projectdiscovery/cdncheck) 查找 WAF，并使用 [christophetd/CloudFlair](https://github.com/christophetd/CloudFlair) 识别真实 IP。

  ```ps1
  echo www.hackerone.com | cdncheck -resp
  www.hackerone.com [waf] [cloudflare]
  ```

* 使用 [sensepost/gowitness](https://github.com/sensepost/gowitness) 为每个网站抓取屏幕截图。

#### 手动测试

通过代理探索网站：

* [Caido - 一款轻量级 Web 安全审计工具包](https://caido.io/)
* [ZAP - OWASP Zed Attack Proxy](https://www.zaproxy.org/)
* [Burp Suite - 社区版](https://portswigger.net/burp/communitydownload)

#### 自动化漏洞扫描器

* [projectdiscovery/nuclei](https://github.com/projectdiscovery/nuclei)：

  ```ps1
  nuclei -u https://example.com
  ```

* [Burp Suite 的 Web 漏洞扫描器](https://portswigger.net/burp/vulnerability-scanner)
* [sullo/nikto](https://github.com/sullo/nikto)

  ```ps1
  ./nikto.pl -h http://www.example.com
  ```

## 寻找 Web 漏洞 (Looking for Web Vulnerabilities)

* 探索网站并寻找本仓库中列出的漏洞：SQL 注入、XSS、CRLF、Cookies 等。
* 测试业务逻辑 (Business Logic) 弱点
    * 过高或负数值
    * 尝试所有功能并点击所有按钮
* [《Web 应用程序黑客手册》(The Web Application Hacker's Handbook) 核查清单](https://web.archive.org/web/20210126221152/https://gist.github.com/gbedoya/10935137)

* 订阅网站并为额外的功能付费以进行测试。

* 检查支付功能 —— [@gwendallecoguic](https://twitter.com/gwendallecoguic/status/988138794686779392)
  > 如果你测试的 Web 应用使用了外部支付网关，请查看文档以查找测试信用卡号，购买某些商品。如果该应用未禁用测试模式，则可以免费购买。

  摘自 [https://stripe.com/docs/testing](https://stripe.com/docs/testing#cards)：“使用以下任何测试卡号、将来有效的过期日期以及任何随机 CVC 号码，即可成功完成付款。每张测试卡的账单国家均设置为美国。”

  测试卡号与令牌

  | 卡号 (NUMBER)     | 品牌 (BRAND)   | 令牌 (TOKEN)   |
  | :-------------   | :------------- | :------------- |
  | 4242424242424242 | Visa           | tok_visa       |
  | 4000056655665556 | Visa (借记卡)   | tok_visa_debit |
  | 5555555555554444 | Mastercard     | tok_mastercard |

  国际测试卡号与令牌

  | 卡号 (NUMBER)     | 令牌 (TOKEN)   | 国家 (COUNTRY)  | 品牌 (BRAND)   |
  | :-------------   | :------------- | :------------- | :------------- |
  | 4000000400000008 | tok_at         | 奥地利 (AT)    | Visa           |
  | 4000000560000004 | tok_be         | 比利时 (BE)    | Visa           |
  | 4000002080000001 | tok_dk         | 丹麦 (DK)      | Visa           |
  | 4000002460000001 | tok_fi         | 芬兰 (FI)      | Visa           |
  | 4000002500000003 | tok_fr         | 法国 (FR)      | Visa           |

## 参考资料 (References)

* [Nmap CheatSheet - HackerTarget](https://hackertarget.com/nmap-cheatsheet-a-quick-reference-guide/)
* [Yahoo phpinfo.php disclosure - Patrik Fehrenbach - January 20, 2013](https://blog.wss.sh/bugbounty-yahoo-phpinfo-php-disclosure/)
* [Bug Bounty Masterclass - Wiz, Gal Nagli](https://www.wiz.io/bug-bounty-masterclass)
