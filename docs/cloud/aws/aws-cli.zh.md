# AWS - 命令行界面 (CLI)

AWS Command Line Interface (命令行界面，CLI) 是一个统一的工具，用于从命令行管理 AWS 服务。使用 AWS CLI，你可以控制多个 AWS 服务、自动化任务，并通过配置文件 (profiles) 管理配置。

## 设置 AWS CLI (Set up AWS CLI)

第一次安装并配置 AWS CLI:

```ps1
aws configure
```

这将提示你输入：

* AWS Access Key ID (AWS 访问密钥 ID)
* AWS Secret Access Key (AWS 私有访问密钥)
* Default region name (默认区域名称)
* Default output format (默认输出格式)

## 创建配置文件 (Creating Profiles)

你可以在 `~/.aws/credentials` 和 `~/.aws/config` 中配置多个配置文件 (profiles)。

* `~/.aws/credentials` (存储凭据)

    ```ini
    [default]
    aws_access_key_id = <default-access-key>
    aws_secret_access_key = <default-secret-key>

    [dev-profile]
    aws_access_key_id = <dev-access-key>
    aws_secret_access_key = <dev-secret-key>

    [prod-profile]
    aws_access_key_id = <prod-access-key>
    aws_secret_access_key = <prod-secret-key>
    ```

* `~/.aws/config` (存储区域和输出设置)

    ```ini
    [default]
    region = us-east-1
    output = json

    [profile dev-profile]
    region = us-west-2
    output = yaml

    [profile prod-profile]
    region = eu-west-1
    output = json
    ```

你也可以通过命令行创建配置文件：

```ps1
aws configure --profile dev-profile
```

## 使用配置文件 (Using Profiles)

运行 AWS CLI 命令时，你可以通过添加 `--profile` 标志来指定要使用的配置文件：

```ps1
aws s3 ls --profile dev-profile
```

如果没有指定配置文件，则使用 **default** (默认) 配置文件。
