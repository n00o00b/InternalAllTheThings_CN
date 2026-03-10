# AWS - 访问令牌与机密 (Access Token & Secrets)

## URL 服务 (URL Services)

| 服务 (Service) | URL                   |
|--------------|-----------------------|
| s3           | `https://{user_provided}.s3.amazonaws.com` |
| cloudfront   | `https://{random_id}.cloudfront.net` |
| ec2          | `https://ec2-{ip-seperated}.compute-1.amazonaws.com` |
| es           | `https://{user_provided}-{random_id}.{region}.es.amazonaws.com` |
| elb          | `http://{user_provided}-{random_id}.{region}.elb.amazonaws.com:80/443` |
| elbv2        | `https://{user_provided}-{random_id}.{region}.elb.amazonaws.com` |
| rds          | `mysql://{user_provided}.{random_id}.{region}.rds.amazonaws.com:3306` |
| rds          | `postgres://{user_provided}.{random_id}.{region}.rds.amazonaws.com:5432` |
| route 53     | `{user_provided}` |
| execute-api  | `https://{random_id}.execute-api.{region}.amazonaws.com/{user_provided}` |
| cloudsearch  | `https://doc-{user_provided}-{random_id}.{region}.cloudsearch.amazonaws.com` |
| transfer     | `sftp://s-{random_id}.server.transfer.{region}.amazonaws.com` |
| iot          | `mqtt://{random_id}.iot.{region}.amazonaws.com:8883` |
| iot          | `https://{random_id}.iot.{region}.amazonaws.com:8443` |
| iot          | `https://{random_id}.iot.{region}.amazonaws.com:443` |
| mq           | `https://b-{random_id}-{1,2}.mq.{region}.amazonaws.com:8162` |
| mq           | `ssl://b-{random_id}-{1,2}.mq.{region}.amazonaws.com:61617` |
| kafka        | `b-{1,2,3,4}.{user_provided}.{random_id}.c{1,2}.kafka.{region}.amazonaws.com` |
| kafka        | `{user_provided}.{random_id}.c{1,2}.kafka.useast-1.amazonaws.com` |
| cloud9       | `https://{random_id}.vfs.cloud9.{region}.amazonaws.com` |
| mediastore   | `https://{random_id}.data.mediastore.{region}.amazonaws.com` |
| kinesisvideo | `https://{random_id}.kinesisvideo.{region}.amazonaws.com` |
| mediaconvert | `https://{random_id}.mediaconvert.{region}.amazonaws.com` |
| mediapackage | `https://{random_id}.mediapackage.{region}.amazonaws.com/in/v1/{random_id}/channel` |

## 访问密钥 ID 和私有密钥 (Access Key ID & Secret)

IAM 使用以下前缀来指示每个唯一 ID 适用的资源类型。前四个字符是根据密钥类型而定的前缀。

| 前缀 (Prefix)       | 资源类型 (Resource type)           |
|--------------|-------------------------|
| ABIA | AWS STS 服务不记名令牌 (AWS STS service bearer token) |
| ACCA | 特定于上下文的凭据 (Context-specific credential) |
| AGPA | 用户组 (User group) |
| AIDA | IAM 用户 (IAM user) |
| AIPA | Amazon EC2 实例配置文件 (Amazon EC2 instance profile) |
| AKIA | 访问密钥 (Access key) |
| ANPA | 托管策略 (Managed policy) |
| ANVA | 托管策略中的版本 (Version in a managed policy) |
| APKA | 公钥 (Public key) |
| AROA | 角色 (Role) |
| ASCA | 证书 (Certificate) |
| ASIA | 临时 (AWS STS) 访问密钥 (Temporary (AWS STS) access key) |

字符串的其余部分是 Base32 编码的，可用于恢复帐户 ID。

```py
import base64
import binascii

def AWSAccount_from_AWSKeyID(AWSKeyID):
    
    trimmed_AWSKeyID = AWSKeyID[4:] #移除 KeyID 前缀
    x = base64.b32decode(trimmed_AWSKeyID) #base32 解码
    y = x[0:6]
    
    z = int.from_bytes(y, byteorder='big', signed=False)
    mask = int.from_bytes(binascii.unhexlify(b'7fffffffff80'), byteorder='big', signed=False)
    
    e = (z & mask)>>7
    return (e)


print ("account id:" + "{:012d}".format(AWSAccount_from_AWSKeyID("ASIAQNZGKIQY56JQ7WML")))
```

## 区域 (Regions)

* US Standard (美国标准) - [s3.amazonaws.com](http://s3.amazonaws.com)
* Ireland (爱尔兰) - [s3-eu-west-1.amazonaws.com](http://s3-eu-west-1.amazonaws.com)
* Northern California (北加利福尼亚) - [s3-us-west-1.amazonaws.com](http://s3-us-west-1.amazonaws.com)
* Singapore (新加坡) - [s3-ap-southeast-1.amazonaws.com](http://s3-ap-southeast-1.amazonaws.com)
* Tokyo (东京) - [s3-ap-northeast-1.amazonaws.com](http://s3-ap-northeast-1.amazonaws.com)

## 通过 API 密钥获取 AWS 控制台访问权限 (Gaining AWS Console Access via API Keys)

一个将会把你的 AWS CLI 凭据转换为 AWS 控制台访问权限的实用程序。

* 使用 [NetSPI/aws_consoler](https://github.com/NetSPI/aws_consoler)

    ```powershell
    $> aws_consoler -v -a AKIA[REDACTED] -s [REDACTED]
    2020-03-13 19:44:57,800 [aws_consoler.cli] INFO: Validating arguments...
    2020-03-13 19:44:57,801 [aws_consoler.cli] INFO: Calling logic.
    2020-03-13 19:44:57,820 [aws_consoler.logic] INFO: Boto3 session established.
    2020-03-13 19:44:58,193 [aws_consoler.logic] WARNING: Creds still permanent, creating federated session.
    2020-03-13 19:44:58,698 [aws_consoler.logic] INFO: New federated session established.
    2020-03-13 19:44:59,153 [aws_consoler.logic] INFO: Session valid, attempting to federate as arn:aws:sts::123456789012:federated-user/aws_consoler.
    2020-03-13 19:44:59,668 [aws_consoler.logic] INFO: URL generated!
    https://signin.aws.amazon.com/federation?Action=login&Issuer=consoler.local&Destination=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3Fregion%3Dus-east-1&SigninToken=[REDACTED]
    ```

## 参考资料 (References)

* [A short note on AWS KEY ID - Tal Be'ery - Oct 27, 2023](https://medium.com/@TalBeerySec/a-short-note-on-aws-key-id-f88cc4317489)
* [Gaining AWS Console Access via API Keys - Ian Williams - March 18th, 2020](https://blog.netspi.com/gaining-aws-console-access-via-api-keys/)
