# 源代码分析 (Source Code Analysis)

> 源代码分析是检查和审查软件程序代码以识别错误、漏洞和潜在改进的过程。这一过程可以由开发者手动执行，也可以通过自动化工具进行，这些工具会自动扫描代码中的安全风险、编码规范违规以及性能低下等问题。

## AI 辅助分析 (AI Analysis)

* [trailofbits/skills](https://github.com/trailofbits/skills) - Trail of BitsClaude Code 技能集，用于安全研究、漏洞检测和审计工作流。

```ps1
npm install -g @github/copilot
copilot
/login
/model
/plugin marketplace add trailofbits/skills
/plugin marketplace browse trailofbits
/plugin install ask-questions-if-underspecified@trailofbits
/plugin install static-analysis@trailofbits
/plugin install entry-point-analyzer@trailofbits
/plugin install semgrep-rule-creator@trailofbits
/plugin install semgrep-rule-variant-creator@trailofbits
/plugin install sharp-edges@trailofbits
/plugin install insecure-defaults@trailofbits
```

## Semgrep

> 适用于多种语言的轻量级静态分析工具。通过类似于源代码的模式 (Pattern) 来寻找漏洞变体。

**安装**：

* 二进制文件：[opengrep/opengrep](https://github.com/opengrep/opengrep) / [semgrep/semgrep](https://github.com/semgrep/semgrep)
* Ubuntu/WSL/Linux/macOS：`python3 -m pip install semgrep`
* macOS：`brew install semgrep`
* Docker

    ```ps1
    docker run -it -v "${PWD}:/src" semgrep/semgrep semgrep login
    docker run -e SEMGREP_APP_TOKEN=<TOKEN> --rm -v "${PWD}:/src" semgrep/semgrep semgrep ci
    ```

**Semgrep 规则**：

* [semgrep/semgrep-rules](https://github.com/semgrep/semgrep-rules) - 官方 Semgrep 规则库
* [trailofbits/semgrep-rules](https://github.com/trailofbits/semgrep-rules) - 由 Trail of Bits 开发的 Semgrep 查询规则
* [Decurity/semgrep-smart-contracts)](https://github.com/Decurity/semgrep-smart-contracts) - 基于 DeFi 漏洞利用的智能合约 Semgrep 规则
* [0xdea/semgrep-rules](https://github.com/0xdea/semgrep-rules) - 旨在促进漏洞研究的 Semgrep 规则集合。
* [elttam/semgrep-rules](https://github.com/elttam/semgrep-rules) - Elttam 的公共 Semgrep 规则仓库。

**其他工具**：

* [Orange-Cyberdefense/grepmarx](https://github.com/Orange-Cyberdefense/grepmarx) - 为 AppSec 爱好者打造的源代码静态分析平台，基于 Semgrep 引擎。

## SonarQube

> 持续检查 (Continuous Inspection)

**安装**

* Docker

    ```ps1
    docker run -d --name sonarqube -p 9000:9000 sonarqube:community
    ```

**配置**

* 访问 localhost:9000
* 使用 `admin:admin` 登录
* 创建本地项目
* 为项目生成令牌 (Token)
* 使用 `sonar-scanner-cli` 并配合生成的令牌执行扫描

    ```ps1
    docker run --rm -e SONAR_HOST_URL="http://10.10.10.10:9000" -v "/tmp/www:/usr/src" sonarsource/sonar-scanner-cli -Dsonar.projectKey=sonar-project-name -Dsonar.sources=. -Dsonar.host.url=http://10.10.10.10:9000 -Dsonar.token=sqp_redacted
    ```

* 查看“安全热点 (Security Hotspots)”选项卡：`http://10.10.10.10:9000/security_hotspots?id=sonar-project-name`

:warning: 在扫描文件夹之前，请移除失效的符号链接。

## Psalm

> 发现 PHP 应用错误的静态分析工具

**安装**

```ps1
composer require --dev vimeo/psalm
```

**配置**

* 创建项目并启动代码库扫描

    ```ps1
    ./vendor/bin/psalm --init
    ./vendor/bin/psalm --taint-analysis
    ./vendor/bin/psalm --report=results.sarif
    ```

* 使用 Sarif 查看器查看结果：[microsoft.github.io/sarif-web-component](https://microsoft.github.io/sarif-web-component/)

## CodeQL

> CodeQL：助力全球安全研究人员的库和查询工具，同时也是 GitHub 高级安全 (Advanced Security) 中代码扫描功能的核心引擎。

**安装**：

* [github/codeql](https://github.com/github/codeql)

**配置**

```ps1
codeql resolve packs
codeql resolve languages
codeql database create <数据库名称> --language=<语言标识符>
codeql database create --language=python <输出文件夹>/python-database
codeql database create --language=cpp <输出文件夹>/cpp-database
codeql database analyze <数据库路径> --format=<格式> --output=<输出路径> <查询规范>...
codeql database analyze /codeql-dbs/example-repo javascript-code-scanning.qls --sarif-category=javascript-typescript  --format=sarif-latest --output=/temp/example-repo-js.sarif
codeql database analyze <数据库路径> microsoft/coding-standards@1.0.0 github/security-queries --format=sarifv2.1.0 --output=query-results.sarif --download
```

## Snyk

> Snyk CLI 扫描并监控你的项目以查找安全漏洞。

**安装**

* [Snyk Security - Visual Studio](https://marketplace.visualstudio.com/items?itemName=snyk-security.snyk-vulnerability-scanner-vs)
* [Snyk Code / Snyk Open Source](https://app.snyk.io)

    ```ps1
    curl https://static.snyk.io/cli/latest/snyk-linux -o snyk
    chmod +x ./snyk
    mv ./snyk /usr/local/bin/ 

    docker run -it \
        -e "SNYK_TOKEN=<TOKEN>" \
        -v "<项目目录>:/project" \
        -v "/home/user/.gradle:/home/node/.gradle" \
    snyk/snyk:gradle:6.4 test --org=my-org-name
    ```

**配置**

```ps1
snyk auth
snyk ignore --file-path=<目录或文件路径>
snyk code test

# npm install snyk-to-html -g
snyk code test --json | snyk-to-html -o results-opensource.html
```

## 参考资料 (References)

* [Code auditing 101 - Rodolphe Ghio - August 2, 2025](https://blog.rodolpheg.xyz/posts/code-auditing--101/)
* [Detect PHP security vulnerabilities with Psalm - Matt Brown - June 23, 2020](https://psalm.dev/articles/detect-security-vulnerabilities-with-psalm)
* [Security Analysis in Psalm - Official Documentation](https://psalm.dev/docs/security_analysis/)
