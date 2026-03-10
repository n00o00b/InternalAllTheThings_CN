# CI/CD - CircleCI

CircleCI 构建的配置文件位于 `.circleci/config.yml`。
默认情况下，CircleCI 流水线不会构建分叉的拉取请求（forked pull request）。这是一个需要由流水线所有者启用的可选功能。

为了在构建拉取请求的工作流中运行 OS 命令，只需在步骤中添加 `run` 指令即可。

```yaml
jobs:
  build:
    docker:
     - image: cimg/base:2022.05
    steps:
        - run: echo "Say hello to YAML!"
```
