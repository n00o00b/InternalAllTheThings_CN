# AWS - 枚举 (Enumerate)

## 收集器 (Collectors)

* [nccgroup/ScoutSuite](https://github.com/nccgroup/ScoutSuite/wiki) - 多云安全审计工具 (Multi-Cloud Security Auditing Tool)

    ```powershell
    $ python scout.py PROVIDER --help
    # --session-token 是可选的，仅用于临时凭据（即角色假设 (role assumption)）。
    $ python scout.py aws --access-keys --access-key-id <AKIAIOSFODNN7EXAMPLE> --secret-access-key <wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY> --session-token <token>
    $ python scout.py azure --cli
    ```

* [RhinoSecurityLabs/pacu](https://github.com/RhinoSecurityLabs/pacu) - 使用具有丰富功能集的可扩展模块集合来利用 AWS 环境中的配置缺陷

    ```powershell
    $ bash install.sh
    $ python3 pacu.py
    set_keys/swap_keys
    run <module_name> [--keyword-arguments]
    run <module_name> --regions eu-west-1,us-west-1
    ```

* [salesforce/cloudsplaining](https://github.com/salesforce/cloudsplaining) - 一个 AWS IAM 安全评估工具，可识别对最小权限的违反情况并生成按风险优先排序的报告

    ```powershell
    pip3 install --user cloudsplaining
    cloudsplaining download --profile myawsprofile
    cloudsplaining scan --input-file default.json
    ```

* [duo-labs/cloudmapper](https://github.com/duo-labs/cloudmapper) - CloudMapper 可帮助你分析 Amazon Web Services (AWS) 环境

    ```powershell
    sudo apt-get install autoconf automake libtool python3.7-dev python3-tk jq awscli build-essential
    pipenv install --skip-lock
    pipenv shell
    report: 生成 HTML 报告。包括帐户摘要和审计发现。
    iam_report: 为帐户的 IAM 信息生成 HTML 报告。
    audit: 检查潜在的错误配置。
    collect: 收集有关帐户的元数据。
    find_admins: 查看 IAM 策略以识别管理员用户和角色，或者具有特定权限的主体。
    ```

* [cyberark/SkyArk](https://github.com/cyberark/SkyArk) - 发现已扫描的 AWS 环境中权限最高的用户，包括 AWS 影子管理员 (AWS Shadow Admins)

    ```powershell
    $ powershell -ExecutionPolicy Bypass -NoProfile
    PS C> Import-Module .\SkyArk.ps1 -force
    PS C> Start-AWStealth
    PS C> Scan-AWShadowAdmins  
    ```

* [BishopFox/CloudFox](https://github.com/BishopFox/CloudFox/) - 自动化云渗透测试的态势感知。专为白盒枚举（SecurityAudit/ReadOnly 类型权限）设计，但也可用作黑盒（找到凭据）。

    ```ps1
    cloudfox aws --profile [profile-name] all-checks
    ```

* [toniblyx/Prowler](https://github.com/toniblyx/prowler) - AWS 安全最佳实践评估、审计、事件响应、持续监控、加固和取证就绪。它遵循 CIS Amazon Web Services Foundations Benchmark 的指南，以及包括 GDPR 和 HIPAA 在内的数十项额外检查（+100）。

    ```powershell
    pip install awscli ansi2html detect-secrets
    sudo apt install jq
    ./prowler -E check42,check43
    ./prowler -p custom-profile -r us-east-1 -c check11
    ./prowler -A 123456789012 -R ProwlerRole
    ```

* [nccgroup/PMapper](https://github.com/nccgroup/PMapper) - 一款快速评估 AWS 中 IAM 权限的工具

    ```powershell
    pip install principalmapper
    pmapper graph --create
    pmapper visualize --filetype png
    pmapper analysis --output-type text

    # 确定 PowerUser 是否可以提升权限
    pmapper query "preset privesc user/PowerUser"
    pmapper argquery --principal user/PowerUser --preset privesc

    # 查找可以提升权限的所有主体
    pmapper query "preset privesc *"
    pmapper argquery --principal '*' --preset privesc

    # 查找 PowerUser 可以访问的所有主体
    pmapper query "preset connected user/PowerUser *"
    pmapper argquery --principal user/PowerUser --resource '*' --preset connected

    # 查找可以访问 PowerUser 的所有主体
    pmapper query "preset connected * user/PowerUser"
    pmapper argquery --principal '*' --resource user/PowerUser --preset connected
    ```

## AWS - 枚举 IAM 权限 (Enumerate IAM permissions)

使用 [andresriancho/enumerate-iam](https://github.com/andresriancho/enumerate-iam) 枚举与 AWS 凭据集相关联的权限

```powershell
git clone git@github.com:andresriancho/enumerate-iam.git
pip install -r requirements.txt
./enumerate-iam.py --access-key AKIA... --secret-key StF0q...
2019-05-10 15:57:58,447 - 21345 - [INFO] Starting permission enumeration for access-key-id "AKIA..."
2019-05-10 15:58:01,532 - 21345 - [INFO] Run for the hills, get_account_authorization_details worked!
2019-05-10 15:58:01,537 - 21345 - [INFO] -- {
    "RoleDetailList": [
        {
            "Tags": [],
            "AssumeRolePolicyDocument": {
                "Version": "2008-10-17",
                "Statement": [
                    {
...
2019-05-10 15:58:26,709 - 21345 - [INFO] -- gamelift.list_builds() worked!
2019-05-10 15:58:26,850 - 21345 - [INFO] -- cloudformation.list_stack_sets() worked!
2019-05-10 15:58:26,982 - 21345 - [INFO] -- directconnect.describe_locations() worked!
2019-05-10 15:58:27,021 - 21345 - [INFO] -- gamelift.describe_matchmaking_rule_sets() worked!
2019-05-10 15:58:27,311 - 21345 - [INFO] -- sqs.list_queues() worked!
```

## 参考资料 (References)

* [An introduction to penetration testing AWS - Akimbocore - HollyGraceful - 06 August 2021](https://akimbocore.com/article/introduction-to-penetration-testing-aws/)
* [AWS CLI Cheatsheet - apolloclark](https://gist.github.com/apolloclark/b3f60c1f68aa972d324b)
* [AWS - Cheatsheet - @Magnussen](https://www.magnussen.funcmylife.fr/article_35)
* [Pacu Open source AWS Exploitation framework - RhinoSecurityLabs](https://rhinosecuritylabs.com/aws/pacu-open-source-aws-exploitation-framework/)
* [PACU Spencer Gietzen - 30 juil. 2018](https://youtu.be/XfetW1Vqybw?list=PLBID4NiuWSmfdWCmYGDQtlPABFHN7HyD5)
