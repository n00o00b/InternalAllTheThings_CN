# AWS - 服务 - Lambda 和 API Gateway

## 列出 Lambda 函数 (List Lambda Functions)

```ps1
aws lambda list-functions
```

### 调用 Lambda 函数 (Invoke a Lambda Function)

```ps1
aws lambda invoke --function-name name response.json --region region 
```

## 提取函数代码 (Extract Function's Code)

```powershell
aws lambda list-functions --profile uploadcreds
aws lambda get-function --function-name "LAMBDA-NAME-HERE-FROM-PREVIOUS-QUERY" --query 'Code.Location' --profile uploadcreds
wget -O lambda-function.zip url-from-previous-query --profile uploadcreds
```

## 列出 API Gateway (List API Gateway)

```ps1
aws apigateway get-rest-apis
aws apigateway get-rest-api --rest-api-id ID
```

## 列出有关端点的信息 (Listing Information About Endpoints)

```ps1
aws apigateway get-resources --rest-api-id ID
aws apigateway get-resource --rest-api-id ID --resource-id ID
aws apigateway get-method --rest-api-id ApiID --resource-id ID --http-method method
```

## 列出 API 密钥 (Listing API Keys)

```ps1
aws apigateway get-api-keys --include-values
```

## 获取有关特定 API 密钥的信息 (Getting Information About A Specific Api Key)

```ps1
aws apigateway get-api-key --api-key KEY
```

## 参考资料 (References)

* [Getting shell and data access in AWS by chaining vulnerabilities - Appsecco - Riyaz Walikar - Aug 29, 2019](https://blog.appsecco.com/getting-shell-and-data-access-in-aws-by-chaining-vulnerabilities-7630fa57c7ed)
