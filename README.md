To integrate the Azure SQL module into your existing Terraform configuration for the Azure Landing Zone, you’ll need to define several parameters specific to the Azure SQL components. Based on your current setup and the `variables.tf` file provided for the Azure SQL module, here’s a summary of the parameters you'll need to pass in:

### Parameters for the Azure SQL Module

1. **`eonId`**: The EON ID for the landing zone.
   ```hcl
   eonId = var.eonId
   ```

2. **`location`**: The Azure region where the resources will be deployed.
   ```hcl
   location = var.location
   ```

3. **`vnetName`**: The name of the virtual network where the SQL Server will be deployed.
   ```hcl
   vnetName = module.subnets.vnet_name
   ```

4. **`vnetResourceGroupName`**: The resource group containing the virtual network.
   ```hcl
   vnetResourceGroupName = module.subnets.system_rg_name
   ```

5. **`pepSubnet`**: The name of the subnet used for private endpoints.
   ```hcl
   pepSubnet = module.subnets.subnets_info["private"].subnet_name
   ```

6. **`pepSubnetId`**: The ID of the private endpoint subnet.
   ```hcl
   pepSubnetId = module.subnets.subnets_info["private"].subnet_id
   ```

7. **`resourceGroupName`**: The resource group name for the Azure SQL Server.
   ```hcl
   resourceGroupName = module.landing_zone.mod_out_app_resource_group_names
   ```

8. **`keyvaultName`**: The name of the Key Vault used for SQL Server secrets.
   ```hcl
   keyvaultName = module.akv.keyvault_name
   ```

9. **`serverBaseName`**: The base name for the SQL Server.
   ```hcl
   serverBaseName = "${var.location}-${var.env_name}-${var.lz_name}"
   ```

10. **`serverName`**: The full name of the SQL Server resource.
    ```hcl
    serverName = "${var.serverBaseName}-sqlserver"
    ```

11. **`lzSPNName`**: The name of the LZ Service Principal.
    ```hcl
    lzSPNName = var.lzSPNName
    ```

12. **`lzSPNObjectId`**: The object ID of the LZ Service Principal.
    ```hcl
    lzSPNObjectId = var.lzSPNObjectId
    ```

13. **`lzSPNClientId`**: The client ID of the LZ Service Principal.
    ```hcl
    lzSPNClientId = var.lzSPNClientId
    ```

14. **`aadadminUserName`**: The AAD group to be set as the AD administrator for SQL Server.
    ```hcl
    aadadminUserName = var.aadadminUserName
    ```

15. **`aadadminObjectId`**: The object ID of the AAD administrator for SQL Server.
    ```hcl
    aadadminObjectId = var.aadadminObjectId
    ```

16. **`sqlDatabases`**: Map of SQL Databases to create on the SQL Server.
    ```hcl
    sqlDatabases = var.sqlDatabases
    ```

17. **`deployDatabaseInExistingServer`**: Boolean flag indicating if the database should be deployed in an existing SQL Server.
    ```hcl
    deployDatabaseInExistingServer = var.deployDatabaseInExistingServer
    ```

18. **`sqlDatabaseActionGroup`**: Map of action group properties to send alerts.
    ```hcl
    sqlDatabaseActionGroup = var.sqlDatabaseActionGroup
    ```

19. **`sqlDatabaseMetricAlerts`**: Map of SQL Database metrics alert properties.
    ```hcl
    sqlDatabaseMetricAlerts = var.sqlDatabaseMetricAlerts
    ```

20. **`sqlFailoverGroup`**: Map of SQL Server failover properties.
    ```hcl
    sqlFailoverGroup = var.sqlFailoverGroup
    ```

21. **`logAnalyticsWorkspace`**: Properties for the LZ-scoped log analytics workspace.
    ```hcl
    logAnalyticsWorkspace = var.logAnalyticsWorkspace
    ```

22. **`diagnostics`**: Map of diagnostics settings for each database.
    ```hcl
    diagnostics = var.diagnostics
    ```

23. **`connectionPolicy`**: The connection policy to SQL Server.
    ```hcl
    connectionPolicy = var.connectionPolicy
    ```

24. **`enableAdOnlyAuth`**: Boolean flag indicating if AD-only authentication to SQL Server is enabled.
    ```hcl
    enableAdOnlyAuth = var.enableAdOnlyAuth
    ```

25. **`failoverRole`**: The role of the SQL Server if part of a failover group.
    ```hcl
    failoverRole = var.failoverRole
    ```

26. **`primaryResourceGroupName`**: The resource group of the primary Server landing zone.
    ```hcl
    primaryResourceGroupName = var.primaryResourceGroupName
    ```

27. **`primaryServerName`**: The full name of the primary SQL Server (if configuring failover).
    ```hcl
    primaryServerName = var.primaryServerName
    ```

28. **`primaryKeyvaultName`**: The name of the primary Azure Key Vault containing the TDE key.
    ```hcl
    primaryKeyvaultName = var.primaryKeyvaultName
    ```

29. **`primarySqlServerTdeKey`**: The primary SQL Server TDE key to be used by server encryption.
    ```hcl
    primarySqlServerTdeKey = var.primarySqlServerTdeKey
    ```

30. **`secondaryResourceGroupName`**: The resource group of the secondary Server landing zone.
    ```hcl
    secondaryResourceGroupName = var.secondaryResourceGroupName
    ```

31. **`secondaryServerName`**: The full name of the secondary SQL Server (if configuring failover).
    ```hcl
    secondaryServerName = var.secondaryServerName
    ```

32. **`enableDiagnostics`**: Boolean flag indicating if DB-level diagnostics logging is enabled.
    ```hcl
    enableDiagnostics = var.enableDiagnostics
    ```

33. **`triggerDBFurnishings`**: Trigger value to update DB furnishings.
    ```hcl
    triggerDBFurnishings = var.triggerDBFurnishings
    ```

34. **`secondaryDatabaseIdsForDiagnostics`**: Map of database IDs for which diagnostics logging is enabled.
    ```hcl
    secondaryDatabaseIdsForDiagnostics = var.secondaryDatabaseIdsForDiagnostics
    ```

You can integrate these parameters into your main configuration file as follows:

```hcl
module "sql" {
  source = "github.com/cloud-era/terraform-azure-component-sql?ref=init"

  eonId                         = var.eonId
  location                      = var.location
  vnetName                      = module.subnets.vnet_name
  vnetResourceGroupName         = module.subnets.system_rg_name
  pepSubnet                     = module.subnets.subnets_info["private"].subnet_name
  pepSubnetId                   = module.subnets.subnets_info["private"].subnet_id
  resourceGroupName             = module.landing_zone.mod_out_app_resource_group_names
  keyvaultName                  = module.akv.keyvault_name
  serverBaseName                = "${var.location}-${var.env_name}-${var.lz_name}"
  serverName                    = "${var.serverBaseName}-sqlserver"
  lzSPNName                     = var.lzSPNName
  lzSPNObjectId                 = var.lzSPNObjectId
  lzSPNClientId                 = var.lzSPNClientId
  aadadminUserName              = var.aadadminUserName
  aadadminObjectId              = var.aadadminObjectId
  sqlDatabases                  = var.sqlDatabases
  deployDatabaseInExistingServer = var.deployDatabaseInExistingServer
  sqlDatabaseActionGroup        = var.sqlDatabaseActionGroup
  sqlDatabaseMetricAlerts       = var.sqlDatabaseMetricAlerts
  sqlFailoverGroup              = var.sqlFailoverGroup
  logAnalyticsWorkspace        = var.logAnalyticsWorkspace
  diagnostics                   = var.diagnostics
  connectionPolicy              = var.connectionPolicy
  enableAdOnlyAuth              = var.enableAdOnlyAuth
  failoverRole                  = var.failoverRole
  primaryResourceGroupName      = var.primaryResourceGroupName
  primaryServerName             = var.primaryServerName
  primaryKeyvaultName           = var.primaryKeyvaultName
  primarySqlServerTdeKey        = var.primarySqlServerTdeKey
  secondaryResourceGroupName    = var.secondaryResourceGroupName
  secondaryServerName           = var.secondaryServerName
  enableDiagnostics             = var.enableDiagnostics
  triggerDBFurnishings          = var.triggerDBFurnishings
  secondaryDatabaseIdsForDiagnostics = var.secondaryDatabaseIdsForDiagnostics
}
```

Make sure to adjust the parameters according to your specific requirements and any additional values that might be required by your Azure SQL module.
