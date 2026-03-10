# AWS - 服务 - S3 存储桶 (S3 Buckets)

AWS S3 存储桶是一种基于云的存储容器，用于保存被称为“对象 (objects)”的文件，这些文件可以通过互联网访问。它具有高度的可扩展性，可以存储大量数据，如文档、图像和备份。S3 通过访问控制、加密和权限管理提供强大的安全性。它确保了高持久性和高可用性，非常适合从任何地方存储和检索数据。

## 工具 (Tools)

* [aws/aws-cli](https://github.com/aws/aws-cli) - 适用于 Amazon Web Services 的通用命令行界面

 ```ps1
 sudo apt install awscli
 ```

* [digi.ninja/bucket-finder](https://digi.ninja/projects/bucket_finder.php) - 搜索公开存储桶，如果启用了目录索引，则列出并下载所有文件

 ```powershell
 wget https://digi.ninja/files/bucket_finder_1.1.tar.bz2 -O bucket_finder_1.1.tar.bz2
 ./bucket_finder.rb my_words
 ./bucket_finder.rb --region ie my_words
 ./bucket_finder.rb --download --region ie my_words
 ./bucket_finder.rb --log-file bucket.out my_words
 ```

* [aws-sdk/boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) - 适用于 Python 的 Amazon Web Services (AWS) SDK

 ```python
 import boto3
 s3 = boto3.client('s3',aws_access_key_id='AKIAJQDP3RKREDACTED',aws_secret_access_key='igH8yFmmpMbnkcUaCqXJIRIozKVaREDACTED',region_name='us-west-1')

 try:
  result = s3.list_buckets()
  print(result)
 except Exception as e:
  print(e)
 ```

* [nccgroup/s3_objects_check](https://github.com/nccgroup/s3_objects_check) - 对 S3 对象有效权限进行白盒评估，以识别可公开访问的文件

    ```powershell
    python3 -m venv env && source env/bin/activate
    pip install -r requirements.txt
    python s3-objects-check.py -h
    python s3-objects-check.py -p whitebox-profile -e blackbox-profile
    ```

* [grayhatwarfare/buckets](https://buckets.grayhatwarfare.com/) - 搜索公开存储桶

## 凭据和配置文件 (Credentials and Profiles)

使用你的 `AWSAccessKeyId` 和 `AWSSecretKey` 创建一个配置文件，然后你就可以在 `aws` 命令中使用 `--profile nameofprofile`。

```js
aws configure --profile nameofprofile
AWS Access Key ID [None]: <AWSAccessKeyId>
AWS Secret Access Key [None]: <AWSSecretKey>
Default region name [None]: 
Default output format [None]: 
```

或者，你可以使用环境变量而不是创建配置文件。

```bash
export AWS_ACCESS_KEY_ID=ASIAZ[...]PODP56
export AWS_SECRET_ACCESS_KEY=fPk/Gya[...]4/j5bSuhDQ
export AWS_SESSION_TOKEN=FQoGZXIvYXdzE[...]8aOK4QU=
```

## 公开 S3 存储桶 (Public S3 Bucket)

开放的 S3 存储桶是指配置为允许公开访问（有意或无意）的 Amazon Simple Storage Service (Amazon S3) 存储桶。这意味着互联网上的任何人都有可能访问、读取甚至修改存储桶中存储的数据，具体取决于设置的权限。

* `http://s3.amazonaws.com/<bucket-name>`
* `http://<bucket-name>.s3.amazonaws.com`
* `https://<bucket-name>.region.amazonaws.com/<file>`

AWS S3 存储桶名称示例：[http://flaws.cloud.s3.amazonaws.com](http://flaws.cloud.s3.amazonaws.com)。

可以通过与目标相关的关键词暴力破解存储桶名称，或者使用 OSINT 工具（如 [buckets.grayhatwarfare.com](https://buckets.grayhatwarfare.com/)）搜索泄露的存储桶。

当启用文件列表 (file listing) 时，名称也会显示在 `<Name>` XML 标签内。

```xml
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<Name>adobe-REDACTED-REDACTED-REDACTED</Name>
```

## 存储桶交互 (Bucket Interations)

### 查找区域 (Find the Region)

使用 dig 或 nslookup 查找 Amazon Web Services (AWS) 服务（例如 S3 存储桶）的区域，请查询该服务域或端点的 DNS 记录。

```bash
$ dig flaws.cloud
;; ANSWER SECTION:
flaws.cloud.    5    IN    A    52.218.192.11

$ nslookup 52.218.192.11
Non-authoritative answer:
11.192.218.52.in-addr.arpa name = s3-website-us-west-2.amazonaws.com.
```

### 列出文件 (List Files)

使用 AWS CLI 列出 AWS S3 存储桶中的文件，可以使用以下命令：

```bash
aws s3 ls <target> [--options]
aws s3 ls s3://bucket-name --no-sign-request --region <insert-region-here>
aws s3 ls s3://flaws.cloud/ --no-sign-request --region us-west-2
```

### 复制、上传和下载文件 (Copy, Upload and Download Files)

* **复制 (Copy)**

  ```bash
  aws s3 cp <source> <target> [--options]
  aws s3 cp local.txt s3://bucket-name/remote.txt --acl authenticated-read
  aws s3 cp login.html s3://bucket-name --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers
  ```

* **上传 (Upload)**

  ```bash
  aws s3 mv <source> <target> [--options]
  aws s3 mv test.txt s3://hackerone.files
  SUCCESS : "move: ./test.txt to s3://hackerone.files/test.txt"
  ```

* **下载 (Download)**

  ```bash
  aws s3 sync <source> <target> [--options]
  aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ . --no-sign-request --region us-west-2
  ```

### 列出文件版本 (List File Versions)

当 AWS S3 存储桶启用版本控制 (versioning) 时，使用 AWS CLI 列出文件历史记录：

```bash
aws s3api list-object-versions --bucket <bucket-name> [--options]
aws s3api list-object-versions --bucket <bucket-name> --prefix <file-path>
```

### 下载特定文件版本 (Download a Specific File Version)

```bash
aws s3api get-object --bucket <bucket-name> --key <source> --version-id <id> <target>
```

## 参考资料 (References)

* [There's a Hole in 1,951 Amazon S3 Buckets - Mar 27, 2013 - Rapid7 willis](https://community.rapid7.com/community/infosec/blog/2013/03/27/1951-open-s3-buckets)
* [Bug Bounty Survey - AWS Basic test](https://web.archive.org/web/20180808181450/https://twitter.com/bugbsurveys/status/860102244171227136)
* [flaws.cloud Challenge based on AWS vulnerabilities - Scott Piper - Summit Route](http://flaws.cloud/)
* [flaws2.cloud Challenge based on AWS vulnerabilities - Scott Piper - Summit Route](http://flaws2.cloud)
* [Guardzilla video camera hardcoded AWS credential - INIT_6 - December 27, 2018](https://blackmarble.sh/guardzilla-video-camera-hard-coded-aws-credentials/)
* [AWS PENETRATION TESTING PART 1. S3 BUCKETS - VirtueSecurity](https://www.virtuesecurity.com/aws-penetration-testing-part-1-s3-buckets/)
* [AWS PENETRATION TESTING PART 2. S3, IAM, EC2 - VirtueSecurity](https://www.virtuesecurity.com/aws-penetration-testing-part-2-s3-iam-ec2/)
* [A Technical Analysis of the Capital One Hack - CloudSploit - Aug 2 2019](https://blog.cloudsploit.com/a-technical-analysis-of-the-capital-one-hack-a9b43d7c8aea?gi=8bb65b77c2cf)
