# CI/CD - Drone CI

Drone 构建的配置文件位于 `.drone.yml`。
Drone 的构建环境通常是自托管的，这意味着你可能会获得运行运行器（runner）的 Kubernetes 集群或托管云环境的过度权限。

为了在构建拉取请求的工作流中运行 OS 命令，只需在步骤中添加 `commands` 指令即可。

```yaml
steps:
  - name: do-something
    image: some-image:3.9
    commands:
      - {Payload}
```
