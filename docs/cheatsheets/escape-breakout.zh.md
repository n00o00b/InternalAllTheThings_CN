# Kiosk 逃逸与越狱 (Kiosk Escape and Jail Breakout)

## 目录 (Summary)

* [方法论 (Methodology)](#%e6%96%b9%e6%b3%95%e8%ae%ba-methodology)
* [获取命令 Shell (Gaining a command shell)](#%e8%8e%b7%e5%8f%96%e5%91%bd%e4%bb%a4-shell-gaining-a-command-shell)
* [粘滞键 (Sticky Keys)](#%e7%b2%98%e6%bb%9a%e9%94%ae-sticky-keys)
* [对话框 (Dialog Boxes)](#%e5%af%b9%e8%af%9d%e6%a1%86-dialog-boxes)
    * [创建新文件](#%e5%88%9b%e5%bb%ba%e6%96%b0%e6%96%87%e4%bb%b6)
    * [打开新的 Windows 资源管理器实例](#%e6%89%93%e5%bc%80%e6%96%b0%e7%9a%84-windows-%e8%b5%84%e6%ba%90%e7%ae%a1%e7%90%86%e5%99%a8%e5%ae%9e%e4%be%8b)
    * [探索右键菜单 (Context Menus)](#%e6%8e%a2%e7%b4%a2%e5%8f%b3%e9%94%ae%e8%8f%9c%e5%8d%95-context-menus)
    * [另存为 (Save as)](#%e5%8f%a6%e5%ad%98%e4%b8%ba-save-as)
    * [输入框 (Input Boxes)](#%e8%be%93%e5%85%a5%e6%a1%86-input-boxes)
    * [绕过文件限制](#%e7%bb%95%e8%bf%87%e6%96%87%e4%bb%b6%e9%99%90%e5%88%b6)
* [Internet Explorer](#internet-explorer)
* [Shell URI 处理器 (Shell URI Handlers)](#shell-uri-%e5%a4%84%e7%90%86%e5%99%a8-shell-uri-handlers)
* [参考资料 (References)](#%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99-references)

## 工具 (Tools)

* [kiosk.vsim.xyz](https://kiosk.vsim.xyz/) —— 用于基于浏览器的 Kiosk 模式测试工具。

## 方法论 (Methodology)

* 显示全局变量及其权限：`export -p`
* 使用 `sudo`/`su` 切换到另一个用户
* 基础提权：如 CVE、sudo 配置错误等。详见：[Linux 提权](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/linux-privilege-escalation/) / [Windows 提权](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/)
* 列出受限 Shell 中的默认命令：`compgen -c`
* 容器逃逸（如果运行在 `Docker`/`LXC` 容器内）
* 网络跳转 (Pivot)
    * 扫描网络上的其他机器或尝试 SSRF 利用
    * 获取云资产的元数据，见 `cloud/aws` 和 `cloud/azure` 部分
* 使用 Shell 内置的通配符 (Globbing) 功能：`echo *`, `echo .*`, `echo /*`

## 获取命令 Shell (Gaining a command shell)

* **快捷键**
    * [Window] + [R] -> cmd
    * [CTRL] + [SHIFT] + [ESC] -> 任务管理器 (Task Manager)
    * [CTRL] + [ALT] + [DELETE] -> 任务管理器 (Task Manager)
* **通过文件浏览器访问**：浏览至包含二进制文件的文件夹（例如 `C:\windows\system32\`），右键点击并“打开”它。
* **拖放 (Drag-and-drop)**：将任何文件拖放到 cmd.exe 上。
* **超链接**：`file:///c:/Windows/System32/cmd.exe`
* **任务管理器**：`文件 (File)` > `运行新任务 (New Task (Run...))` > `cmd`
* **MSPAINT.exe**
    * 打开 MSPaint.exe 并将画布大小设置为：`宽度=6` 和 `高度=1` 像素。
    * 放大画布以便操作。
    * 使用颜色选择器，按如下值设置像素（从左到右）：

        ```ps1
        第1个: R: 10,  G: 0,   B: 0
        第2个: R: 13,  G: 10,  B: 13
        第3个: R: 100, G: 109, B: 99
        第4个: R: 120, G: 101, B: 46
        第5个: R: 0,   G: 0,   B: 101
        第6个: R: 0,   G: 0,   B: 0
        ```

    * 另存为 24 位位图 (*.bmp;*.dib)
    * 将其扩展名从 bmp 更改为 bat 并运行。
    * 生成的文件也可下载：[escape-breakout-mspaint.bmp](./files/escape-breakout-mspaint.bmp)

## 粘滞键 (Sticky Keys)

* 弹出粘滞键对话框
    * 通过 Shell URI：`shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}`
    * 快速按 5 次 [SHIFT] 键
* 访问“轻松使用设置中心 (Ease of Access Center)”
* 你将进入“设置粘滞键”页面，在“轻松使用设置中心”向上移动一个级别。
* 启动屏幕键盘 (OSK)
* 之后你便可以使用键盘快捷键了 (如 CTRL+N)。

## 对话框 (Dialog Boxes)

### 创建新文件

* 批处理文件 —— 右键 > 新建 > 文本文件 > 重命名为 .BAT (或 .CMD) > 编辑 > 打开
* 快捷方式 —— 右键 > 新建 > 快捷方式 > `%WINDIR%\system32`

## 打开新的 Windows 资源管理器实例

* 右键点击任何文件夹 > 选择 `在新窗口中打开`

## 探索右键菜单 (Exploring Context Menus)

* 右键点击任何文件/文件夹并探索右键菜单。
* 点击 `属性`，尤其是快捷方式的属性，可以通过 `打开文件所在位置` 获得进一步访问权限。

### 另存为 (Save as)

* “另存为” / “打开”选项
* “打印”功能 —— 选择“打印到文件”选项 (XPS/PDF 等)
* 通过路径访问 `\\127.0.0.1\c$\Windows\System32\` 并执行 `cmd.exe`

### 输入框 (Input Boxes)

许多输入框接受文件路径；尝试使用 UNC 路径（如 `//攻击者IP/` 或 `//127.0.0.1/c$` 或 `C:\`）进行输入测试。

### 绕过文件限制

在 `文件名` 框中输入 *.* 或 *.exe 或类似内容。

## Internet Explorer

### 下载并运行/打开

* 文本文件 -> 由记事本 (Notepad) 打开

### 菜单

* 地址栏
* 搜索菜单
* 帮助菜单
* 打印菜单
* 所有其他提供对话框的菜单

### 访问文件系统

在地址栏输入以下路径：

* file://C:/windows
* C:/windows/
* %HOMEDRIVE%
* \\127.0.0.1\c$\Windows\System32

### 未关联的协议

可以使用除常规 `http` 或 `https` 以外的其他协议逃逸基于浏览器的 Kiosk。
如果你能访问地址栏，可以使用任何已知协议（`irc`, `ftp`, `telnet`, `mailto` 等）来触发“打开方式”提示，并选择主机上安装的程序。
该程序随后将以 URI 作为参数启动，你需要选择一个接收该参数时不会崩溃的程序。
通过在 URI 中添加空格，可以向程序发送多个参数。

注意：此技术要求所选协议尚未与任何程序关联。

示例 —— 使用自定义配置启动 Firefox：

这是一个绝妙的技巧，因为使用自定义配置启动的 Firefox 可能不像默认配置那样经过严格的安全加固。

0. 需要安装 Firefox。
1. 在地址栏输入以下 URI：`irc://127.0.0.1 -P "Test"`
2. 按回车导航到该 URI。
3. 选择 Firefox 程序。
4. Firefox 将以配置文件 `Test` 启动。

在此示例中，这相当于运行以下命令：

```ps1
firefox irc://127.0.0.1 -P "Test"
```

## Shell URI 处理器 (Shell URI Handlers)

URI (统一资源标识符) 处理器是一种软件组件，它能够让 Web 浏览器或操作系统将 URI 传递给合适的应用程序进行进一步处理。

例如，当你点击网页中的 "mailto:" 链接时，你的设备知道应打开默认邮箱应用。这是因为 "mailto:" URI 方案已注册由邮箱应用处理。同样，"http:" 和 "https:" URI 通常由 Web 浏览器处理。

本质上，URI 处理器在 Web 内容和桌面应用之间架起了一座桥梁，使用户在不同类型的资源之间导航时获得无缝体验。

以下 URI 处理器可能会触发机器上的应用程序：

* shell:DocumentsLibrary
* shell:Librariesshell:UserProfiles
* shell:Personal
* shell:SearchHomeFolder
* shell:System shell:NetworkPlacesFolder
* shell:SendTo
* shell:Common Administrative Tools
* shell:MyComputerFolder
* shell:InternetFolder

## 参考资料 (References)

* [PentestPartners - Breaking out of Citrix and other restricted desktop environments](https://www.pentestpartners.com/security-blog/breaking-out-of-citrix-and-other-restricted-desktop-environments/)
* [Breaking Out! of Applications Deployed via Terminal Services, Citrix, and Kiosks - Scott Sutherland - May 22nd, 2013](https://blog.netspi.com/breaking-out-of-applications-deployed-via-terminal-services-citrix-and-kiosks/)
* [Escaping from KIOSKs - HackTricks](https://book.hacktricks.xyz/physical-attacks/escaping-from-gui-applications)
* [Breaking out of Windows Kiosks using only Microsoft Edge - Firat Acar - May 24, 2022](https://blog.nviso.eu/2022/05/24/breaking-out-of-windows-kiosks-using-only-microsoft-edge/)
* [HOW TO LAUNCH COMMAND PROMPT AND POWERSHELL FROM MS PAINT - 2022-05-14 - Rickard](https://tzusec.com/how-to-launch-command-prompt-and-powershell-from-ms-paint/)
