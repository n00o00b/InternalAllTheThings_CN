# 代理绕过 (Proxy Bypass)

> HTTP 代理服务器充当客户端（如 Web 浏览器）和 Web 服务器之间的中间人。它处理客户端对 Web 资源的请求，从目标服务器获取资源，并将其返回给客户端。

## 目录 (Summary)

* [方法论 (Methodology)](#methodology)
    * [发现代理配置 (Discover Proxy Configuration)](#discover-proxy-configuration)
    * [PAC 代理 (PAC Proxy)](#pac-proxy)
    * [常见绕过方法 (Common Bypass)](#common-bypass)
* [参考资料 (References)](#references)

## 方法论 (Methodology)

### 发现代理配置 (Discover Proxy Configuration)

* Windows，在注册表键 `DefaultConnectionSettings` 中：

    ```ps1
    Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections\DefaultConnectionSettings
    Software\Microsoft\Windows\CurrentVersion\Internet Settings\ProxyServer
    ```

* Windows 命令：

    ```ps1
    netsh winhttp show proxy
    ```

* Linux，在环境变量 `http_proxy` 和 `https_proxy` 中：

    ```ps1
    env
    cat /etc/profile.d/proxy.conf
    ```

### PAC 代理 (PAC Proxy)

PAC (代理自动配置) 是一种自动确定 Web 流量是否应通过代理服务器的方法。它使用一个包含名为 `FindProxyForURL(url, host)` 的 JavaScript 函数的 .pac 文件。

* proxy.pac
* wpad.dat

**示例**：

```ps1
function FindProxyForURL(url, host) {
    if (dnsDomainIs(host, '.example.com')) {
        return 'DIRECT';
    }
    return 'PROXY proxy.example.com:8080';
}
```

**工具**：

* [PortSwigger - Proxy Auto Config](https://portswigger.net/bappstore/7b3eae07aa724196ab85a8b64cd095d1) - 此扩展程序可自动配置 Burp 的上游代理 (upstream proxies)，以匹配桌面代理设置。这包括对代理自动配置 (PAC) 脚本的支持。

### 常见绕过方法 (Common Bypass)

* 尝试多种连接互联网的方式：
    * IP 地址。
    * 归类为“健康/金融 (Health/Finance)”类的域名。

* 使用同一环境中的另一个可访问的代理。

* URL 的弱正则表达式可能会被滥用以绕过代理配置：

    ```ps1
    user:pass@domain/endpoint?parameter#hash
    例如: microsoft.com:microsoft.com@microsoft.com.evil.com/microsoft.com?microsoft.com#microsoft.com
    ```

* 受信任的网站：[Living Off Trusted Sites (LOTS) Project](https://lots-project.com/)
    * Amazon Cloud: AWS 端点
    * Microsoft Cloud: Azure 端点
    * Google Cloud: GCP 端点
    * live.sysinternals.com

* 用户代理 (User-Agents)
    * 工具相关的 User-Agent：curl, python, powershell

        ```ps1
        User-Agent: curl/8.11.0
        User-Agent: python-requests/2.32.3
        User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; fr-FR) WindowsPowerShell/5.1.26100.2161
        ```

    * 平台相关的 User-Agent：Android/iOS/平板电脑

        ```ps1
        Mozilla/5.0 (Linux; Android 14; Pixel 9 Build/AD1A.240905.004; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/129.0.6668.78 Mobile Safari/537.36 [FB_IAB/FB4A;FBAV/484.0.0.63.83;IABMV/1;] 
        Mozilla/5.0 (iPhone; CPU iPhone OS 18_0_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15E148 [FBAN/FBIOS;FBAV/485.1.0.45.110;FBBV/665337277;FBDV/iPhone17,1;FBMD/iPhone;FBSN/iOS;FBSV/18.0.1;FBSS/3;FBCR/;FBID/phone;FBLC/it_IT;FBOP/80] 
        ```

* 域前置 (Domain Fronting)
* 协议
    * TCP
    * Websocket (HTTP)
    * DNS 数据外泄 (DNS Exfiltration)

## 参考资料 (References)

* [Proxy managed by enterprise? No problem! Abusing PAC and the registry to get burpin’ - Thomas Grimée - August 17, 2021](https://blog.nviso.eu/2021/08/17/proxy-managed-by-enterprise-no-problem-abusing-pac-and-the-registry-to-get-burpin/)
* [Proxy: Internal Proxy - MITRE ATT&CK - March 14, 2020](https://attack.mitre.org/versions/v16/techniques/T1090/001/)
