# CI/CD - Azure DevOps

## Azure Pipelines

Azure Pipelines 的配置文件通常位于仓库的根目录下，名为 `azure-pipelines.yml`。
你可以根据其触发器指令判断流水线是否构建拉取请求。查找 `pr:` 指令：

```yaml
trigger:
  branches:
      include:
      - master
      - refs/tags/*
pr:
- master
```

## 密钥提取 (Secret Extractions)

针对以下服务连接 (service connections) 提取密钥：

* AzureRM
* GitHub
* AWS
* SonarQube
* SSH

```ps1
nord-stream.py devops ... --build-yaml test.yml --build-type ssh  
```

## 参考资料 (References)

* [Azure DevOps CICD Pipelines - Command Injection with Parameters, Variables and a discussion on Runner hijacking - Sana Oshika - May 1 2023](https://pulsesecurity.co.nz/advisories/Azure-Devops-Command-Injection)
