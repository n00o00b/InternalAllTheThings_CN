# 终端检测与响应 (Endpoint Detection and Response / EDR)

终端检测与响应 (EDR) 是一种安全解决方案，它结合了实时监控、数据收集和高级分析，用于探测、调查和响应终端层面的网络威胁。通过利用机器学习算法和行为分析，EDR 工具可以识别恶意活动，自动执行遏制和补救操作，并提供取证见解，以增强组织的整体安全态势。

## 静态检测 (Static Detection)

**机制**：静态检测是 EDR 和反病毒软件中使用的一种安全技术，它在不执行文件和应用程序的情况下对其进行分析，通常基于预定义的签名或已知的恶意模式。

**绕过方式**：

- 混淆字符串 (Obfuscate strings)
- 动态解析字符串 (Dynamically resolving strings)
- 动态解析导入，减少导入地址表 (IAT)
- 自定义 `GetProcAddress` 和 `GetModuleHandle`
- API 哈希 (API Hashing)

## 用户行为分析 (User Behavioural Analysis)

**机制**：用户行为分析 (UBA) 监控并分析用户活动和模式，以检测异常和潜在威胁。

**绕过方式**：

- 学习运维安全 (OPSEC) 方法

## 用户层 Windows 函数监控 (Usermode Windows Function Monitoring)

**机制**：用户层 Windows 函数监控是一种跟踪并分析用户空间进程中 Windows API（应用程序编程接口）调用和函数执行的技术。

**绕过方式**：

- 解除挂钩 (Unhooking)
- 间接系统调用 (Indirect syscalls)

## 调用堆栈分析 (Call Stack Analysis)

**机制**：通过调用堆栈 (Call Stack) 链检查函数调用的来源。

**绕过方式**：

- 待补充 (TODO)
- 待补充 (TODO)

## 进程分析 (Process Analysis)

**机制**：进程分析包括检查内存区域、识别远程进程访问以及评估子进程，以深入了解进程关系，发现隐藏或可疑的活动。

**绕过方式**：

- 避免使用 RWX 内存区域（将 RW 修改为 RX）
- 断开父子进程链接（例如：阻止 `word.exe` 生成 `cmd.exe`）
- 待补充 (TODO)

## 内核回调 (Kernel Callbacks)

**机制**：在 EDR 上下文中，内核回调是由内核驱动程序注册的函数，这些函数会响应操作系统内核中的特定事件或操作而触发。

**绕过方式**：

- 待补充 (TODO)

## 使用 WDAC 禁用 EDR 组件

将 WDAC 策略文件 `SiPolicy.p7b` 放置在 `C:\Windows\System32\CodeIntegrity\` 目录下并重启机器。

```ps1
smbmap -u Administrator -p P@ssw0rd -H 192.168.4.4 --upload "/home/kali/SiPolicy.p7b" "ADMIN\$/System32/CodeIntegrity/SiPolicy.p7b"
smbmap -u Administrator -p P@ssw0rd -H 192.168.4.4 -x "shutdown /r /t 0"
```

使用 Krueger（一款 .NET 后渗透工具）：

- [logangoins/Krueger](https://github.com/logangoins/Krueger) - 使用 WDAC 远程终止 EDR 的概念验证 (PoC) .NET 工具。

    ```ps1
    inlineExecute-Assembly --dotnetassembly C:\Tools\Krueger.exe --assemblyargs --host ms01
    ```

## 参考资料 (References)

- [Flying Under the Radar: Part 1: Resolving Sensitive Windows Functions with x64 Assembly - theepicpowner - Apr 24, 2024](https://theepicpowner.gitlab.io/posts/Flying-Under-the-Radar-Part-1/)
- [Malware AV/VM evasion - part 16: WinAPI GetProcAddress implementation. Simple C++ example - cocomelonc](https://cocomelonc.github.io/malware/2023/04/16/malware-av-evasion-16.html)
- [Custom GetProcAddress And GetModuleHandle Implementation (X64) - daax - December 15, 2016](https://revers.engineering/custom-getprocaddress-and-getmodulehandle-implementation-x64/)
- [Weaponizing WDAC: Killing the Dreams of EDR - Jonathan Beierle and Logan Goins - December 20, 2024](https://beierle.win/2024-12-20-Weaponizing-WDAC-Killing-the-Dreams-of-EDR/)
