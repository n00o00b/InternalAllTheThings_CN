# 安卓应用 (Android Application)

## 练习环境 (Lab)

* [payatu/diva-android](https://github.com/payatu/diva-android) - Android 平台下极其不安全且易受攻击的应用。
* [HTB VIP - Pinned](https://app.hackthebox.com/challenges/282) - Hack The Box 挑战。
* [HTB VIP - Manager](https://app.hackthebox.com/challenges/283) - Hack The Box 挑战。

## 提取 APK (Extract APK)

### ADB 方法

连接到 ADB shell 并列出/下载软件包。
你可能需要启用 `开发者模式 (Developer mode)` 和 `调试 (Debugging)` 才能连接 `adb`。

```powershell
adb shell pm list packages
adb shell pm path com.example.someapp
adb pull /data/app/com.example.someapp-2.apk
```

### 商店 (Stores)

警告：从非官方商店下载 APK 文件可能会危及设备的安全性。这些来源通常托管恶意软件。请务必使用受信任的官方应用商店进行下载。

* [Google Play](https://play.google.com/store/apps) - 官方商店
* [Apkpure.fr](https://apkpure.fr/fr/) - Google Play 的替代方案
* [Apkpure.co](https://apkpure.co) - Google Play 的替代方案
* [Aptoide](https://fr.aptoide.com/) - Google Play 的替代方案
* [Aurora Store](https://f-droid.org/fr/packages/com.aurora.store/) - Google Play 的替代方案

通过第三方工具从 Google Play 下载 APK：

* [apkcombo.com](https://apkcombo.com/downloader/)
* [apps.evozi.com](https://apps.evozi.com/apk-downloader/)

## 静态分析 (Static Analysis)

### 从 APK 提取内容

搜索 `flag`、`secret` 等字符串，默认的字符串文件是 `Resources/resources.arsc/res/values/strings.xml`。

```powershell
apktool d application.apk
```

### 将数据反编译为 Java 代码

* 将 `application.apk` 重命名为 `application.zip`：`mv application.apk application.zip`
* 提取 `classes.dex`：`unzip application.zip`
* 使用 `dex2jar` 获取 jar 文件：`/usr/bin/d2j-dex2jar classes.dex`
* 使用 `jadx`（利用全部 CPU 进程）：`jadx classes.dex -j $(grep -c ^processor /proc/cpuinfo) -d Downloads/app/ > /dev/null`

    ```powershell
    jadx-gui
    --deobf # 移除 AndroGuard 的混淆
    -e      # 为 Android Studio 生成 gradle 项目（易于查找函数）
    ```

要逆向 `.odex`，你需要提供 `/system/framework/arm`。幸运的是，既然我们有了固件，我们就拥有了这些。

```powershell
java -jar baksmali-2.3.4.jar x application.odex -d k107-mb-8.1/system/framework/arm -o application
apktool d application.apk 
apktool b rebuild_folder -o rebuilt.apk
```

### 反编译原生代码 (Native Code)

原生库通常表现为 `.so` 文件。
这些库默认包含在 APK 的 `/lib/<cpu>/lib<name>.so` 或 `/assets/<custom_name>` 路径下。

使用 `IDA`, `Radare2/Cutter` 或 `Ghidra` 进行逆向。

| 原生 CPU (CPU Native) | 库路径 (Library Path)        |
|----------------------|-----------------------------|
| "通用" 32位 ARM      | lib/armeabi/libcalc.so      |
| x86                  | lib/x86/libcalc.so          |
| x64                  | lib/x86_64/libcalc.so       |
| ARMv7                | lib/armeabi-v7a/libcalc.so  |
| ARM64                | lib/arm64-v8a/libcalc.so    |

:warning: 共享对象文件 (`.so`) 不一定需要嵌入在应用中。

### 签名并打包 APK

* `apktool` + `jarsigner`

    ```powershell
    apktool b ./application.apk
    keytool -genkey -v -keystore application.keystore -alias application -keyalg RSA -keysize 2048 -validity 10000
    jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore application.keystore application.apk application
    zipalign -v 4 application.apk application-signed.apk
    ```

* `apktool` + `signapk`

    ```powershell
    apktool b app-release
    ./signapk app-release/dist/app-release.apk
    ```

* [patrickfav/uber-apk-signer](https://github.com/patrickfav/uber-apk-signer) (仅限 Linux)

    ```powershell
    java -jar uber-apk-signer.jar --apks /path/to/apks
    ```

* [APK Toolkit v1.3](https://xdaforums.com/t/tool-apk-toolkit-v1-3-windows.4572881/) (仅限 Windows)

### 移动安全框架静态分析 (MobSF Static)

> 移动安全框架 (Mobile Security Framework, MobSF) 是一款自动化的、全方位的移动应用（Android/iOS/Windows）渗透测试、恶意软件分析和安全性评估框架，能够执行静态和动态分析。

* [MobSF - 文档](https://mobsf.github.io/docs/#/)
* [MobSF - Github](https://github.com/MobSF/Mobile-Security-Framework-MobSF)
* [MobSF - 在线演示](https://mobsf.live/)

运行 [MobSF/Mobile-Security-Framework-MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF)：

* 来自 DockerHub 的最新版本

    ```powershell
    docker run -it --name mobsf -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
    ```

* 在 Docker 容器上启用持久化

    ```powershell
    docker run -it --rm --name mobsf -p 8000:8000 -v <your_local_dir>:/root/.MobSF opensecurity/mobile-security-framework-mobsf:latest
    ```

### 在线资产 (Online Assets)

:warning: 将 APK 上传到不受控的网站会带来数据泄露、恶意软件、知识产权窃取和隐私泄露等风险。请仅使用受信任的平台，以确保应用的安全和完整。

* [appetize.io](https://appetize.io/) - 立即在浏览器中运行移动应用
* [mobsf.live](https://mobsf.live/) - MobSF 的演示版本
* [hybrid-analysis.com](https://www.hybrid-analysis.com/sample/573df0b1cb5ffc0a25306be5ec83483ed1b2acdba37dd93223b9f14f42b2fdea?environmentId=200) - APK 文件的沙箱分析

### React Native 和 Hermes

在 `assets` 文件夹中寻找 `index.android.bundle` 即可识别 React Native 应用。

```ps1
Hermes: pip install hbctool
╰─$ hbctool disasm index.android.bundle indexasm
[*] 正在将 'index.android.bundle' 反汇编到 'indexasm' 路径
[*] Hermes Bytecode [ Source Hash: 4013cb75f7e16d4474f5cf258edc45ee16585560, HBC Version: 74 ]
[*] 完成
```

### Flutter

通过 `MANIFEST.MF` 文件中的标识以及搜索 `libflutter.so` 来识别 Flutter 应用。

* [worawit/blutter](https://github.com/worawit/blutter) - Flutter 移动应用逆向工程工具

    ```ps1
    blutter jadx/resources/lib/arm64-v8a/ ./blutter_output
    ```

## 动态分析 (Dynamic Analysis)

安卓恶意软件的动态分析包括在受控环境中执行和监控应用，以观察其行为。该技术可检测恶意活动，如数据外泄、未经授权的访问和系统修改。此外，它还有助于对应用功能进行逆向工程，揭示隐藏的功能和潜在的漏洞，以便更好地缓解威胁。

### Burp Suite

* Proxy (代理) > Listen to all interfaces (监听所有接口)
* Import/Export CA certificate (导入/导出 CA 证书)
* `adb push burp.der /sdcard/burp.crt`
* 打开设备上的“设置”并搜索“安装证书 (Install Cert)”
* 点击“从 SD 卡安装证书“
* 配置 AVD 以使用该代理

```ps1
# 为安卓系统转换 Burp 证书
openssl x509 -inform DER -in burp.der -out burp.pem
openssl x509 -inform PEM -subject_hash_old -in burp.pem |head -1
mv burp.pem <hash output>.0

# 将证书推送到 AVD
emulator -list-avds
emulator -avd Pentesting_Device -writable-system
adb root
adb remount
adb push <hash>.0 /sdcard/

# 更改权限
adb shell
mv /sdcard/<hash>.0 /system/etc/security/cacerts/
chmod 644 /system/etc/security/cacerts/<hash>.0
chown root:root /system/etc/security/cacerts/<hash>.0
```

### Frida

* [Frida - 文档](https://frida.re/docs/android)
* [Frida - Github](https://github.com/frida/frida/)

从 Releases 下载 [`frida`](https://github.com/frida/frida/releases)。

```ps1
pip install frida-tools
unxz frida-server.xz
adb root # 可能需要
adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
```

有趣的 Frida 脚本：

* [Universal Android SSL Pinning Bypass with Frida](https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/) -  `frida --codeshare pcipolloni/universal-android-ssl-pinning-bypass-with-frida -f YOUR_BINARY`
* [frida-multiple-unpinning](https://codeshare.frida.re/@akabe1/frida-multiple-unpinning/) - `frida --codeshare akabe1/frida-multiple-unpinning -f YOUR_BINARY`
* [aesinfo](https://codeshare.frida.re/@dzonerzy/aesinfo/) - `frida --codeshare dzonerzy/aesinfo -f YOUR_BINARY`
* [fridantiroot](https://codeshare.frida.re/@dzonerzy/fridantiroot/) - `frida --codeshare dzonerzy/fridantiroot -f YOUR_BINARY`
* [anti-frida-bypass](https://codeshare.frida.re/@enovella/anti-frida-bypass/) - `frida --codeshare enovella/anti-frida-bypass -f YOUR_BINARY`
* [xamarin-antiroot](https://codeshare.frida.re/@Gand3lf/xamarin-antiroot/) - `frida --codeshare Gand3lf/xamarin-antiroot -f YOUR_BINARY`
* [Intercept Android APK Crypto Operations](https://codeshare.frida.re/@fadeevab/intercept-android-apk-crypto-operations/) - `frida --codeshare fadeevab/intercept-android-apk-crypto-operations -f YOUR_BINARY`
* [Android Location Spoofing](https://codeshare.frida.re/@dzervas/android-location-spoofing/) - `frida --codeshare dzervas/android-location-spoofing -f YOUR_BINARY`
* [java-crypto-viewer](https://codeshare.frida.re/@Serhatcck/java-crypto-viewer/) - `frida --codeshare Serhatcck/java-crypto-viewer -f YOUR_BINARY`

### 运行时移动安全 (Runtime Mobile Security)

> 运行时移动安全 (Runtime Mobile Security, RMS) 📱🔥 —— 是一个强大的 Web 界面，可帮助你在运行时操纵安卓和 iOS 应用。

* [RMS - Github](https://github.com/m0bilesecurity/RMS-Runtime-Mobile-Security)

**要求**：

* `adb`
* `frida`：目标设备上的 server 已启动并运行

如果最喜欢的浏览器出现问题，请使用 Google Chrome（已完全支持）。

* 安装 RMS

    ```powershell
    npm install -g rms-runtime-mobile-security
    ```

* 确保 `frida-server` 在目标设备上运行。
* 启动 RMS：`rms`
* 在浏览器中打开 `http://127.0.0.1:5491/`
* 关联（Attach）到应用，使用 `adb shell pm list package | grep NAME` 查找名称。

### Genymotion

Genymotion 是一款专为开发者设计的强大安卓模拟器，为应用测试提供快速可靠的虚拟设备。它具有 GPS、电池和网络模拟功能，可进行全面的测试和开发。

* [Genymotion](https://www.genymotion.com/)
* [Genymotion Desktop](https://www.genymotion.com/product-desktop/)
* [Genymotion Device Image](https://www.genymotion.com/product-device-image/)
* [Genymotion SaaS](https://www.genymotion.com/product-cloud/)

### Android SDK 模拟器 (Android SDK emulator)

不带 Google Play 商店的安卓虚拟设备 (AVD)。

* 下载 API 25 构建版本的文件

    ```powershell
    sdkmanager "system-images;android-25;google_apis;x86_64"
    ```

* 基于之前下载的内容创建设备

    ```powershell
    avdmanager create avd x86_64_api_25 -k "system-images;android-25;google_apis;x86_64"
    ```

* 运行模拟器

    ```powershell
    emulator @x86_64_api_25

    emulator -list-avds
    emulator -avd <非生产环境_avd名称> -writable-system -no-snapshot
    emulator -avd Pixel_XL_API_31 -writable-system -http-proxy 127.0.0.1:8080
    ```

* 安装 APK

    ```powershell
    adb install ./challenge.apk
    ```

* 启动应用

    ```powershell
    adb shell monkey -p com.scottyab.rootbeer.sample 1
    ```

### 移动安全框架动态分析 (MobSF Dynamic)

:warning: 如果你使用 MobSF Docker 容器或在虚拟机中设置 MobSF，则动态分析将无法工作。

**要求**：

* Genymotion (支持 x86_64 架构的 Android 4.1 - 11.0 版本，最高 API 30)
    * Android 5.0 - 11.0 —— 使用 Frida 且无需任何配置设置即可开箱即用。
    * Android 4.1 - 4.4 —— 使用 Xposed 框架，需要 MobSFy。
* Genymotion Cloud (云端)
    * [Amazon Marketplace - TCP 5555](https://aws.amazon.com/marketplace/seller-profile?id=933724b4-d35f-4266-905e-e52e4792bc45)
    * [Azure Marketplace - TCP 5555](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/genymobile.genymotion-cloud)
* Android Studio 模拟器 (仅支持最高 API 28 的 Android 镜像)
    * 不带 Google Play 商店的 AVD

MobSF 的动态分析提供以下功能：

* Web API 查看器
* Frida API 监控

### Appium

Appium 是一个开源项目和相关软件生态系统，旨在通过各种应用平台促进 UI 自动化，包括移动端（iOS、Android、Tizen）、浏览器（Chrome、Firefox、Safari）、桌面端（macOS、Windows）、电视（Roku、tvOS、Android TV、Samsung）等！

* 安装 Appium：`npm install -g appium`
* 安装并验证 `uiautomator2` 驱动程序

    ```ps1
    export JAVA_HOME=/usr/lib/jvm/default-java
    export ANDROID_HOME=/home/user/Android/Sdk/
    wget https://github.com/google/bundletool/releases/download/1.17.1/bundletool-all-1.17.1.jar
    sudo mv bundletool-all-1.17.1.jar /usr/local/bin
    appium driver install uiautomator2
    appium driver doctor uiautomator2
    ```

* 在默认主机 (0.0.0.0) 和端口 (4723) 上启动 server：`appium server`
* 安装 Appium Python 客户端：`pip install Appium-Python-Client`
* 使用 [appium/appium-inspector](https://github.com/appium/appium-inspector)，配置如下 capability：

    ```json
    {
    "platformName": "Android",
    "appium:automationName": "UiAutomator2"
    }
    ```

示例：

* [quickstarts/py/test.py](https://github.com/appium/appium/blob/master/packages/appium/sample-code/quickstarts/py/test.py)
* [quickstarts/js/test.js](https://github.com/appium/appium/blob/master/packages/appium/sample-code/quickstarts/js/test.js)
* [quickstarts/js/test.rb](https://github.com/appium/appium/blob/master/packages/appium/sample-code/quickstarts/rb/test.rb)

### Flutter

重新打包 Flutter 安卓应用以允许 Burp Suite 代理拦截。

* [ptswarm/reFlutter](https://github.com/ptswarm/reFlutter) - Flutter 逆向工程框架

    ```ps1
    pip3 install reflutter
    reflutter application.apk
    ```

* 使用 [patrickfav/uber-apk-signer](https://github.com/patrickfav/uber-apk-signer/releases/tag/v1.2.1) 对 apk 进行签名

    ```ps1
    java -jar ./uber-apk-signer-1.3.0.jar --apks release.apk
    java -jar ./uber-apk-signer.jar --allowResign -a release.RE.apk
    ```

另一种方法是使用带 `zygisk-reflutter` 的已 Root 安卓设备。

* [yohanes/zygisk-reflutter](https://github.com/yohanes/zygisk-reflutter) - 基于 Zygisk 的 reFlutter (已 Root 的安卓设备，已安装 Magisk 并启用了 Zygisk)

    ```ps1
    adb push  zygiskreflutter_1.0.zip /sdcard/
    adb shell su -c magisk --install-module /sdcard/zygiskreflutter_1.0.zip
    adb reboot
    ```

## SSL 固定 (Pinning) 绕过

APK 中的 SSL 证书固定 (Pinning) 涉及将服务器的公钥或证书直接嵌入到应用中。这确保了应用仅信任特定的证书，通过拒绝任何与固定证书不匹配的证书（即使它们在其他方面有效）来防止中间人攻击。

:warning: Android 9.0 正在更改网络安全配置的默认设置，以阻止所有明文流量。

* [shroudedcode/apk-mitm](https://github.com/shroudedcode/apk-mitm) - 一个自动为 HTTPS 检查准备安卓 APK 文件的 CLI 应用。

    ```powershell
    $ npx apk-mitm application.apk
    npx: 139 installé(s) en 12.206s
    ╭ apk-mitm v0.6.1
    ├ apktool v2.4.1
    ╰ uber-apk-signer v1.1.0
    Using temporary directory:
    /tmp/87d3a4921ddf86cde634205480f89e90
    ✔ 正在对 APK 文件进行解码
    ✔ 正在修改应用清单
    ✔ 正在修改网络安全配置
    ✔ 正在禁用证书固定
    ✔ 正在对修补后的 APK 文件进行编码
    ✔ 正在对修补后的 APK 文件进行签名
    完成！修补后的文件：./application.apk
    ```

* [51j0/Android-CertKiller](https://github.com/51j0/Android-CertKiller) - 一个用于绕过安卓 SSL/证书固定的自动化脚本。

    ```powershell
    python main.py -w #(向导模式)
    python main.py -p 'root/Desktop/base.apk' #(手动模式)
    ```

* [frida/frida](https://github.com/frida/frida) - 通用 SSL 固定 (Pinning) 绕过

    ```javascript
    $ adb devices
    $ adb root
    $ adb shell
    $ phone:/# ./frida-server

    // https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/
    $ frida -U --codeshare pcipolloni/universal-android-ssl-pinning-bypass-with-frida -f com.example.pinned

    $ frida -U -f org.package.name -l universal-ssl-check-bypass.js --no-pause
    Java.perform(function() {                
        var array_list = Java.use("java.util.ArrayList");
        var ApiClient = Java.use('com.android.org.conscrypt.TrustManagerImpl');
        ApiClient.checkTrustedRecursive.implementation = function(a1,a2,a3,a4,a5,a6) {
            var k = array_list.$new(); 
            return k;
        }
    },0);
    ```

* [m0bilesecurity/RMS-Runtime-Mobile-Security](https://github.com/m0bilesecurity/RMS-Runtime-Mobile-Security) - 证书固定 (Pinning) 绕过脚本 (all + okhttpv3)
* [federicodotta/Brida](https://github.com/federicodotta/Brida) - Burp Suite 与 Frida 之间的新桥梁。

## Root 检测绕过 (Root Detection Bypass)

常见的 Root 检测技术：

* Su 二进制文件：`su`/`busybox`
* 已知的 Root 文件/路径：`Superuser.apk`
* Root 管理应用：`Magisk`, `SuperSU`
* RW (读写) 路径：`/system`, `/data` 目录
* 系统属性

常见的绕过方式：

* [fridantiroot](https://codeshare.frida.re/@dzonerzy/fridantiroot/) - `frida --codeshare dzonerzy/fridantiroot -f YOUR_BINARY`
* [xamarin-antiroot](https://codeshare.frida.re/@Gand3lf/xamarin-antiroot/) - `frida --codeshare Gand3lf/xamarin-antiroot -f YOUR_BINARY`
* [multiple-root-detection-bypass/](https://codeshare.frida.re/@KishorBal/multiple-root-detection-bypass/) - `frida --codeshare KishorBal/multiple-root-detection-bypass -f YOUR_BINARY`

## 安卓调试桥 (Android Debug Bridge, ADB)

安卓调试桥 (ADB) 是一种多功能命令行工具，可实现计算机与安卓设备之间的通信。它促进了安装应用、调试、访问设备 shell 和传输文件等任务，是开发者和高级用户在安卓开发和故障排除中必不可少的工具。

### USB 调试

* 打开 **设置** 应用。
* 选择 **系统**。
* 滚动到底部并选择 **关于手机**。
* 滚动到底部，连续点击 **版本号 (Build number)** 7 次。
* 返回上一个屏幕，在底部附近找到 **开发者选项**。
* 向下滚动并启用 **USB 调试**。

```ps1
./platform-tools/adb connect IP:PORT
./platform-tools/adb shell
```

### 无线调试 (Wireless Debugging)

* 打开 **设置** 应用。
* 选择 **系统**。
* 滚动到底部并选择 **关于手机**。
* 滚动到底部，连续点击 **版本号 (Build number)** 7 次。
* 返回上一个屏幕，在底部附近找到 **开发者选项**。
* 向下滚动并启用 **Wifi 调试**。
* 点击 **Wifi 调试** 进入设置。

还有一个步骤，你需要使用代码对设备进行配对。

```ps1
./platform-tools/adb pair IP:PORT CODE
./platform-tools/adb connect IP:PORT
./platform-tools/adb shell
```

| 命令 (Command)                | 描述 (Description)                             |
|------------------------------|------------------------------------------------|
| `adb devices`                | 列出设备                                       |
| `adb connect <IP>:<PORT>`    | 连接到远程设备                                 |
| `adb install app.apk`        | 安装应用                                       |
| `adb uninstall app.apk`      | 卸载应用                                       |
| `adb root`                   | 以 root 身份重启 adbd                         |
| `adb shell pm list packages` | 列出软件包                                     |
| `adb shell pm list packages -3` | 显示第三方软件包                              |
| `adb shell pm list packages -f` | 显示软件包及关联文件                          |
| `adb shell pm clear com.test.abc` | 删除与软件包关联的所有数据                    |
| `adb pull <remote> <local>`  | 下载文件                                       |
| `adb push <local> <remote>`  | 上传文件                                       |
| `adb shell screenrecord /sdcard/demo.mp4`| 录制屏幕视频                                |
| `adb shell am start -n com.test.abc` | 启动一个 Activity                              |
| `adb shell am startservice` | 启动一个服务                                   |
| `adb shell am broadcast`    | 发送一个广播                                   |
| `adb logcat *:D`             | 显示 Debug 级别日志                            |
| `adb logcat -c`              | 清除整个日志                                   |

## 安卓虚拟设备 (Android Virtual Device, AVD)

安卓虚拟设备 (AVD) 是模拟物理安卓设备的模拟器配置。它允许开发者在模拟环境中，通过特定的硬件配置、屏幕尺寸和安卓版本来测试和运行安卓应用，从而在无需实际设备的情况下促进应用测试。

```ps1
emulator -avd Pixel_8_API_34 -writable-system
```

| 命令 (Command)                | 描述 (Description)                             |
|------------------------------|------------------------------------------------|
| `-tcpdump /path/dumpfile.cap`| 将所有流量抓取到文件中 |
| `-dns-server X.X.X.X`        | 设置 DNS 服务器 |
| `-http-proxy X.X.X.X:8080`   | 设置 HTTP 代理 |
| `-port 5556`                 | 设置 ADB TCP 端口号 |

## 解锁引导加载程序 (Unlock Bootloader)

**要求**：

* 启用 `设置` > `开发者选项` > `OEM 解锁`
* 启用 `设置` > `开发者选项` > `USB 调试`

解锁引导加载程序 (Bootloader) 将清除用户数据分区。在某些设备上，这些方法需要一个密钥才能成功解锁 Bootloader。

* 方法 1

    ```ps1
    adb reboot bootloader
    fastboot oem unlock
    ```

* 方法 2

    ```ps1
    adb reboot bootloader
    fastboot flashing unlock
    ```

* 基于芯片的方法
    * 对于高通设备，你可以使用 EDL (紧急下载模式)
    * 对于联发科 (MediaTek) 设备，使用 BROM (Boot ROM) 模式
    * 对于展锐 (Unisoc) 设备，使用 Research Download 模式。

## 参考资料 (References)

* [A beginners guide to using Frida to bypass root detection. - DianaOpanga - November 27, 2023](https://medium.com/@dianaopanga/a-beginners-guide-to-using-frida-to-bypass-root-detection-16af76b989ac)
* [Android App Reverse Engineering 101 - @maddiestone](https://www.ragingrock.com/AndroidAppRE/)
* [Android app vulnerability classes - Google Play Protect](https://static.googleusercontent.com/media/www.google.com/fr//about/appsecurity/play-rewards/Android_app_vulnerability_classes.pdf)
* [Appium documentation](https://appium.io/docs/en/latest/)
* [Configuring Android Emulator with Burp Suite - Jarrod @Jrod_R87 - January 8, 2025](https://owlhacku.com/configuring-android-emulator-with-burp-suite/)
* [Configuring Burp Suite with Android Emulators - Aashish Tamang - June 6, 2022](https://blog.yarsalabs.com/setting-up-burp-for-android-application-testing/)
* [Configuring Burp Suite With Android Nougat - ropnop - January 18, 2018](https://blog.ropnop.com/configuring-burp-suite-with-android-nougat)
* [Configuring Frida with BurpSuite and Genymotion to bypass Android SSL Pinning - arben - September 4, 2020](https://spenkk.github.io/bugbounty/Configuring-Frida-with-Burp-and-GenyMotion-to-bypass-SSL-Pinning/)
* [How to root an Android device for analysis and vulnerability assessment - Joe Lovett - August 23, 2024](https://www.pentestpartners.com/security-blog/how-to-root-an-android-device-for-analysis-and-vulnerability-assessment/)
* [Intercepting OkHttp at Runtime With Frida - A Practical Guide - Szymon Drosdzol - January 22, 2026](https://blog.doyensec.com/2026/01/22/frida-instrumentation.html)
* [Introduction to Android Pentesting - Jarrod - July 8, 2024](https://owlhacku.com/introduction-to-android-pentesting/)
* [Mobile Systems and Smartphone Security - @reyammer](https://mobisec.reyammer.io)
* [Rooting an Android Emulator for Mobile Security Testing - 8ksecresearch - April 17, 2025](https://8ksec.io/rooting-an-android-emulator-for-mobile-security-testing/)
