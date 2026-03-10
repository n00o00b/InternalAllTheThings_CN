# Mythic C2

## 目录 (Summary)

* [安装 (Installation)](#installation)
* [Agent (Agents)](#agents)
* [配置文件 (Profiles)](#profiles)
* [参考资料 (References)](#references)

## 安装 (Installation)

```ps1
sudo apt-get install build-essential
git clone https://github.com/its-a-feature/Mythic --depth 1
./install_docker_ubuntu.sh
./install_docker_debian.sh
cd Mythic
sudo make
sudo ./mythic-cli start
```

## Agent (Agents)

* [Mythic 社区 Agent 功能矩阵 (Mythic Community Agent Feature Matrix)](https://mythicmeta.github.io/overview/agent_matrix.html)

Agent 可以在此处找到：[https://github.com/MythicAgents](https://github.com/MythicAgents)

```ps1
./mythic-cli install github https://github.com/MythicAgents/Medusa # 兼容 Python 2.7 和 3.8 的 Mythic Agent
./mythic-cli install github https://github.com/MythicAgents/Hannibal # 使用 PIC C 编写的 Mythic Agent
./mythic-cli install github https://github.com/MythicAgents/thanatos # 使用 Rust 编写，针对 Linux 和 Windows 主机的 Mythic C2 agent
./mythic-cli install github https://github.com/MythicAgents/poseidon # 使用 Golang 为 Linux/MacOS 编写的 Mythic Agent
./mythic-cli install github https://github.com/MythicAgents/Apollo # 使用 C# 和 .NET Framework 4.0 编写的 Mythic Agent 
./mythic-cli install github https://github.com/MythicAgents/Athena # 使用 .NET 编写的 Mythic Agent
./mythic-cli install github https://github.com/MythicAgents/Xenon # 使用 C 编写并兼容 httpx profile 的 Mythic Agent
```

## 配置文件 (Profiles)

C2 配置文件 (C2 Profiles) 可以在此处找到：[https://github.com/MythicC2Profiles](https://github.com/MythicC2Profiles)

```ps1
./mythic-cli install github https://github.com/MythicC2Profiles/httpx
./mythic-cli install github https://github.com/MythicC2Profiles/http
./mythic-cli install github https://github.com/MythicC2Profiles/websocket
./mythic-cli install github https://github.com/MythicC2Profiles/dns
./mythic-cli install github https://github.com/MythicC2Profiles/dynamichttp
./mythic-cli install github https://github.com/MythicC2Profiles/smb
./mythic-cli install github https://github.com/MythicC2Profiles/tcp
```

## SSL

如果你想使用 SSL，请将你的密钥 (key) 和证书 (cert) 放入 `C2_Profiles/HTTP/c2_code` 文件夹中，并更新 `key_path` 和 `cert_path` 变量以包含这些文件的名称。

使用 Let's Encrypt 的 certbot 获取域名的密钥和证书：

```ps1
sudo apt install certbot
certbot certonly --standalone -d "example.com" --register-unsafely-without-email --non-interactive --agree-tos
```

在 Agent 容器中添加文件：

```ps1
docker cp /etc/letsencrypt/archive/example.com/fullchain1.pem http:/Mythic/http/c2_code/fullchain.pem
docker cp /etc/letsencrypt/archive/example.com/privkey1.pem http:/Mythic/http/c2_code/privkey.pem
```

或者，如果你将 `use_ssl` 指定为 true 且磁盘上还没有任何证书，则 profile 会自动为你生成一些自签名证书以供使用。

## 参考资料 (References)

* [Mythic 文档 (Mythic Documentation)](https://docs.mythic-c2.net)
