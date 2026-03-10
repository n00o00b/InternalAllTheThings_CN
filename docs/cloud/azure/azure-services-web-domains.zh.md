# Azure 服务 - DNS 后缀 (DNS Suffix)

## DNS 表 (DNS table)

许多 Azure 服务会生成带有后缀（如 `.cloudapp.azure.com`、`.windows.net`）的自定义端点。下表列出了常见服务及其关联的 DNS 后缀。

当这些服务被代理或防火墙规则列入白名单 (Whitelisted) 时，它们也可以被用于域名置前 (Domain fronting) 或与外部 C2 服务器进行通信。

| 服务 | 域名 |
| --- | --- |
| 分析服务后缀 (Analysis Services) | .asazure.windows.net |
| API 管理后缀 (API Management) | .azure-api.net |
| 应用服务后缀 (App Services) | .azurewebsites.net |
| 自动化后缀 (Automation) | .azure-automation.net |
| 批处理后缀 (Batch) | .batch.azure.com |
| Blob 端点后缀 (Blob Endpoint) | .blob.core.windows.net |
| CDN 后缀 (CDN) | .azureedge.net |
| Data Lake 分析目录后缀 (Data Lake Analytics) | .azuredatalakeanalytics.net |
| Data Lake 存储后缀 (Data Lake Store) | .azuredatalakestore.net |
| DocumentDB/CosmosDB 后缀 | .documents.azure.com |
| 事件透传后缀 (Event Hubs) | .servicesbus.windows.net |
| 文件端点后缀 (File Endpoint) | .file.core.windows.net |
| FrontDoor 后缀 | .azurefd.net |
| IoT 中心后缀 (IoT Hub) | .azure-devices.net |
| 密钥保管库后缀 (Key Vault) | .vault.azure.net |
| 逻辑应用后缀 (Logic App) | .azurewebsites.net |
| 队列端点后缀 (Queue Endpoint) | .queue.core.windows.net |
| Redis 缓存后缀 (Redis Cache) | .redis.cache.windows.net |
| 服务总线后缀 (Service Bus) | .servicesbus.windows.net  |
| Service Fabric 后缀 | .cloudapp.azure.com |
| SQL 数据库后缀 (SQL Database) | .database.windows.net |
| 存储端点后缀 (Storage Endpoint) | .core.windows.net |
| 表端点后缀 (Table Endpoint) | .table.core.windows.net |
| 流量管理器后缀 (Traffic Manager) | .trafficmanager.net |
| Web 应用程序网关后缀 (Web Application Gateway) | .cloudapp.azure.com |

## 参考资料 (References)

* [Azure services URLs and IP addresses for firewall or proxy whitelisting - Daniel Neumann - 20. December 2016](https://www.danielstechblog.io/azure-services-urls-and-ip-addresses-for-firewall-or-proxy-whitelisting/)
