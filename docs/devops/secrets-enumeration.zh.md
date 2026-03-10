# 硬编码密钥枚举 (Hardcoded Secrets Enumeration)

## 工具 (Tools)

- [synacktiv/nord-stream](https://github.com/synacktiv/nord-stream) - 列出 CI/CD 环境中存储的密钥，并通过部署恶意流水线来提取它们。
- [xforcered/SCMKit](https://github.com/xforcered/SCMKit) - 源码管理测试工具包 (Source Code Management Attack Toolkit)。

## 在仓库、文件与代码中搜索 (Search inside Repositories, Files and Codes)

- 发现特定 SCM 系统中正使用的仓库

    ```ps1
    SCMKit.exe -s gitlab -m listrepo -c userName:password -u https://gitlab.something.local
    SCMKit.exe -s gitlab -m listrepo -c apiKey -u https://gitlab.something.local
    ```

- 在特定 SCM 系统中通过仓库名称搜索仓库

    ```ps1
    SCMKit.exe -s github -m searchrepo -c userName:password -u https://github.something.local -o "某个搜索词"
    SCMKit.exe -s gitlab -m searchrepo -c apikey -u https://gitlab.something.local -o "某个搜索词"
    ```

- 在特定 SCM 系统中搜索包含给定关键词的代码

    ```ps1
    SCMKit.exe -s github -m searchcode -c userName:password -u https://github.something.local -o "某个搜索词"
    SCMKit.exe -s github -m searchcode -c apikey -u https://github.something.local -o "某个搜索词"
    ```

- 在特定 SCM 系统中搜索文件名包含给定关键词的仓库文件

    ```ps1
    SCMKit.exe -s gitlab -m searchfile -c userName:password -u https://gitlab.something.local -o "某个搜索词"
    SCMKit.exe -s gitlab -m searchfile -c apikey -u https://gitlab.something.local -o "某个搜索词"
    ```

- 在 GitLab 中列出当前用户拥有的代码片段 (Snippets)

    ```ps1
    SCMKit.exe -s gitlab -m listsnippet -c userName:password -u https://gitlab.something.local
    SCMKit.exe -s gitlab -m listsnippet -c apikey -u https://gitlab.something.local
    ```

## 参考资料 (References)

- [CI/CD SECRETS EXTRACTION, TIPS AND TRICKS - Hugo Vincent, Théo Louis-Tisserand - 01/03/2023](https://www.synacktiv.com/publications/cicd-secrets-extraction-tips-and-tricks.html)
