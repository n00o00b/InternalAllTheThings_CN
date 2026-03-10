# CI/CD - BuildKite

BuildKite 构建的配置文件位于 `.buildkite/*.yml`。
BuildKite 的构建环境通常是自托管的，这意味着你可能会获得运行运行器（runner）的 Kubernetes 集群或托管云环境的过度权限。

为了在构建拉取请求（pull request）的工作流中运行 OS 命令，只需在步骤中添加 `command` 指令即可。

```yaml
steps:
  - label: "Example Test"
    command: echo "Hello!"
```
