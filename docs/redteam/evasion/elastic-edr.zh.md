# Elastic EDR

> Elastic EDR (终端检测与响应) 是 Elastic Security 的一个组件，旨在解决终端层面的网络安全威胁。它在预防、检测和响应勒索软件和恶意软件等网络威胁方面发挥着至关重要的作用。

* [peasead/elastic-container](https://github.com/peasead/elastic-container) - 搭建一个包含 Kibana、Fleet 和检测引擎 (Detection Engine) 的简单 Elastic 容器。

## 安装与设置 (Setup)

* 首先，你需要安装 `docker` 和 `docker-compose` 插件。

    ```ps1
    # 添加 Docker 官方 GPG 密钥：
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # 将仓库添加到 Apt 源：
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update

    # 从 apt 安装 docker
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

* 你可能需要向默认用户授予 `docker` 权限。

    ```ps1
    sudo groupadd docker
    sudo usermod -aG docker $USER
    ```

* 安装运行 Elastic 脚本所需的依赖。

    ```ps1
    apt-get update
    apt-get install jq git curl
    ```

* 克隆项目。

    ```ps1
    git clone https://github.com/peasead/elastic-container
    cd elastic-container
    ```

* 编辑 `.env` 文件以设置凭据和激活规则。

    ```ps1
    ELASTIC_PASSWORD="changeme"
    KIBANA_PASSWORD="changeme"
    STACK_VERSION="8.11.2"
    WindowsDR=1
    LICENSE=trial # 启用白金版功能
    ```

* 下载镜像并运行容器。

    ```ps1
    chmod +x ./elastic-container.sh
    ./elastic-container.sh start
    ```

* 访问 Elastic EDR 界面：`https://localhost:5601`
* 依次点击 Fleet > `Add agent`
* 选择 Enroll in Fleet（推荐）
* 复制 Windows PowerShell 命令，如果使用的是不受信任的证书，请在命令末尾添加 `--insecure` 标志。

    ```ps1
    powershell Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-7.15.1-windows-x86_64.zip -outfile elastic-agent-7.15.1-windows-x86_64.zip
    Expand-Archive -Path elastic-agent-7.15.1-windows-x86_64.zip -DestinationPath C:\ElasticAgent
    C:\ElasticAgent\elastic-agent-7.15.1-windows-x86_64\elastic-agent.exe install -f --fleet-server-es={{ fleet_server_es }} --fleet-server-service-token={{ fleet_token }} --fleet-server-policy={{ fleet_policy }}
    ```

* 依次点击 Fleet > Integrations > Elastic Defend
    * 将 `Prevent`（预防）切换为 `Detect`（检测），以保持程序运行。
    * 启用以下功能以收集更多数据：

        ```ps1
        windows.advanced.memory_protection.shellcode_collect_sample
        windows.advanced.memory_protection.memory_scan_collect_sample
        windows.advanced.memory_protection.shellcode_enhanced_pe_parsing
        ```

* 销毁容器。

    ```ps1
    ./elastic-container.sh destroy
    ```

## 参考资料 (References)

* [The Elastic Container Project for Security Research - Andrew Pease, Colson Wilhoit, Derek Ditch - 1 March 2023](https://www.elastic.co/security-labs/the-elastic-container-project)
* [Cyber Security Lab Basics - Installing EDR in Malware Development Lab - AhmedS Kasmani](https://www.youtube.com/watch?v=1luhjL7TN9U)
* [Setting Up Elastic 8 with Kibana, Fleet, Endpoint Security, and Windows Log Collection - IppSec - 10 oct. 2022](https://youtu.be/Ts-ofIVRMo4)
