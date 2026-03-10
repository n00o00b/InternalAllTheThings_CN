# CI/CD 攻击 (CI/CD Attacks)

> CI/CD 流水线 (Pipelines) 通常由不受信任的操作触发，例如公开 Git 仓库的分叉拉取请求 (forked PR) 和新提交的问题 (issues)。这些系统通常包含敏感密钥或在特权环境中运行。攻击者可以通过提交经过精心设计的 Payload 来触发流水线，从而在这些系统中获得远程代码执行 (RCE)。此类漏洞也被称为污染流水线执行 (Poisoned Pipeline Execution, PPE)。

## 目录 (Summary)

- [工具 (Tools)](#tools)
- [CI/CD 产品 (CI/CD Products)](#summary)
    - [GitHub Actions](./cicd-github-actions)
    - [Gitlab CI](./cicd-gitlab-ci)
    - [Azure Pipelines (Azure DevOps)](./cicd-azure-devops)
    - [Circle CI](./cicd-circle-ci)
    - [Drone CI](./cicd-drone-ci)
    - [BuildKite](./cicd-buildkite)
- [硬编码密钥枚举 (Hardcoded Secrets Enumeration)](./secrets-enumeration)
- [包管理器与构建文件 (Package Managers and Build Files)](./package-managers)
- [参考资料 (References)](#references)

## 工具 (Tools)

- [praetorian-inc/gato](https://github.com/praetorian-inc/gato) - GitHub 自托管运行器 (Self-Hosted Runner) 枚举与攻击工具。
- [AdnaneKhan/Gato-X](https://github.com/AdnaneKhan/Gato-X) - Gato (Github Attack TOolkit) 的分支版本 —— 极致版。
- [messypoutine/gravy-overflow](https://github.com/messypoutine/gravy-overflow) - 一个 GitHub Actions 供应链 CTF / Goat 练习场。
- [xforcered/SCMKit](https://github.com/xforcered/SCMKit) - 源码管理 (SCM) 攻击工具包。
- [synacktiv/octoscan](https://github.com/synacktiv/octoscan) - Octoscan 是针对 GitHub Actions 工作流的静态漏洞扫描器。
- [synacktiv/gh-hijack-runner](https://github.com/synacktiv/gh-hijack-runner) - 一个用于创建虚假 GitHub 运行器并劫持流水线作业以泄露 CI/CD 密钥的 Python 脚本。
- [synacktiv/nord-stream](https://github.com/synacktiv/nord-stream) - 列出 CI/CD 环境中存储的密钥，并通过部署恶意流水线来提取它们。
- [praetorian-inc/glato](https://github.com/praetorian-inc/glato) - GitLab 攻击工具包 (GitLab Attack TOolkit)。

## 参考资料 (References)

- [Poisoned Pipeline Execution](https://web.archive.org/web/20240226215436/https://www.cidersecurity.io/top-10-cicd-security-risks/poisoned-pipeline-execution-ppe/)
- [DEF CON 25 - Exploiting Continuous Integration (CI) and Automated Build systems - spaceB0x - 2 nov. 2017](https://youtu.be/mpUDqo7tIk8)
- [Controlling the Source: Abusing Source Code Management Systems - Brett Hawkins - August 9, 2022](https://securityintelligence.com/posts/abusing-source-code-management-systems/)
- [Fixing Typos and Breaching Microsoft’s Perimeter - John Stawinski IV - April 15, 2024](https://johnstawinski.com/2024/04/15/fixing-typos-and-breaching-microsofts-perimeter/)
