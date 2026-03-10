# Azure 服务 - 部署模板 (Deployment Template)

* 列出部署 (deployments)

    ```powershell
    PS Az> Get-AzResourceGroup
    PS Az> Get-AzResourceGroupDeployment -ResourceGroupName SAP
    ```

* 导出部署模板 (Deployment Template)

    ```ps1
    PS Az> Save-AzResourceGroupDeploymentTemplate -ResourceGroupName <资源组名称> -DeploymentName <部署名称>
    
    # 搜索硬编码密码 (hardcoded password)
    cat <部署名称>.json 
    cat <.json 文件路径> | Select-String password
    ```

## 参考资料 (References)

* [Training - Attacking and Defending Azure Lab - Altered Security](https://www.alteredsecurity.com/azureadlab)
