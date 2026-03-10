# Azure 服务 - 存储 Blob (Storage Blob)

* Blobs - `*.blob.core.windows.net`
* 文件服务 (File Services) - `*.file.core.windows.net`
* 数据表 (Data Tables) - `*.table.core.windows.net`
* 队列 (Queues) - `*.queue.core.windows.net`

## 枚举 Blob (Enumerate blobs)

```powershell
PS > . C:\Tools\MicroBurst\Misc\InvokeEnumerateAzureBlobs.ps1
PS > Invoke-EnumerateAzureBlobs -Base <短域名> -OutputFile azureblobs.txt
Found Storage Account -  redacted.blob.core.windows.net
```

## 列出并下载 Blob (List and download blobs)

访问 `https://<存储名称>.blob.core.windows.net/<存储容器>?restype=container&comp=list` 会提供一个包含 Azure Blob 完整列表的 JSON 文件。

```xml
<EnumerationResults ContainerName="https://<存储名称>.blob.core.windows.net/<存储容器>">
    <Blobs>
        <Blob>
            <Name>index.html</Name>
            <Url>https://<存储名称>.blob.core.windows.net/<存储容器>/index.html</Url>
            <Properties>
            <Last-Modified>Fri, 20 Oct 2023 20:08:20 GMT</Last-Modified>
            <Etag>0x8DBD1A84E6455C0</Etag>
            <Content-Length>782359</Content-Length>
            <Content-Type>text/html</Content-Type>
            <Content-Encoding/>
            <Content-Language/>
            <Content-MD5>JSe+sM+pXGAEFInxDgv4CA==</Content-MD5>
            <Cache-Control/>
            <BlobType>BlockBlob</BlobType>
            <LeaseStatus>unlocked</LeaseStatus>
            </Properties>
        </Blob>
```

浏览已删除的文件。

```ps1
$ curl -s -H "x-ms-version: 2019-12-12" 'https://<存储名称>.blob.core.windows.net/<存储容器>?restype=container&comp=list&include=versions' | xmllint --format - | grep Name

<EnumerationResults ServiceEndpoint="https://<存储名称>.blob.core.windows.net/" ContainerName="<存储容器>">
      <Name>index.html</Name>
      <Name>scripts-transfer.zip</Name>
```

```powershell
PS Az> Get-AzResource
PS Az> Get-AzStorageAccount -name <名称> -ResourceGroupName <名称>
PS Az> Get-AzStorageContainer -Context (Get-AzStorageAccount -name <名称> -ResourceGroupName <名称>).context
PS Az> Get-AzStorageBlobContent -Container <名称> -Context (Get-AzStorageAccount -name <NAME> -ResourceGroupName <NAME>).context -Blob
```

检索具有公共访问权限的公开容器 (Retrieve exposed containers with public access)

```ps1
PS Az> (Get-AzStorageAccount | Get-AzStorageContainer).cloudBlobContainer | select Uri,@{n='PublicAccess';e={$_.Properties.PublicAccess}}
```

## SAS URL

* 使用 [存储浏览器 (Storage Explorer)](https://azure.microsoft.com/en-us/features/storage-explorer/)。
* 在左侧菜单中点击 **打开连接对话框 (Open Connect Dialog)**。
* 选择 **Blob 容器 (Blob container)**。
* 在 **选择身份验证方法 (Select Authentication Method)** 页面：
    * 选择 **共享访问签名 (Shared access signature / SAS)** 并点击“下一步”。
    * 将 URL 拷贝到 **Blob 容器 SAS URL (Blob container SAS URL)** 字段中。

:warning: 你也可以使用 `subscription` (用户名/密码) 来访问存储资源，如 Blob 和文件。

## 参考资料 (References)

* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
