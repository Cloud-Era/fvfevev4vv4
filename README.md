To add **central diagnostic settings** to your **Azure Virtual Network** within the existing `terraform-azure-virtual-network` module, you will need to include the **diagnostic setting resource** for the virtual network and configure it to point to the **central Log Analytics Workspace**.

Here’s how you can achieve this:

### Step 1: Update the `terraform-azure-virtual-network` Module
You can add the `azurerm_monitor_diagnostic_setting` resource to enable diagnostic logging for your virtual network and point it to the **central Log Analytics Workspace**.

In your `terraform-azure-virtual-network/main.tf`, modify it as follows:

```hcl
resource "azurerm_virtual_network" "Vnet" {
  name                = var.res_vnet_name
  location            = var.res_location
  resource_group_name = var.res_vnet_rg_name
  address_space       = var.res_vnet_address_prefix
  tags                = var.res_tags
}

# Add Central Diagnostic Setting for Virtual Network
resource "azurerm_monitor_diagnostic_setting" "vnet_diagnostic_central" {
  name                       = "${azurerm_virtual_network.Vnet.name}-diagnostic-central"
  target_resource_id         = azurerm_virtual_network.Vnet.id
  log_analytics_workspace_id = var.central_law_workspace_id  # Variable for Central LAW Workspace

  logs {
    category = "NetworkSecurityGroupEvent"
    enabled  = true

    retention_policy {
      enabled = true
      days    = 30  # Set retention policy as per your requirements
    }
  }

  metrics {
    category = "AllMetrics"
    enabled  = true

    retention_policy {
      enabled = true
      days    = 30
    }
  }
}
```

### Step 2: Declare the Central LAW Variable
In the module's `variables.tf` file, add a new input variable for the **central Log Analytics Workspace ID**:

```hcl
variable "central_law_workspace_id" {
  description = "The ID of the Centralized Log Analytics Workspace for diagnostics"
  type        = string
}
```

### Step 3: Update the Landing Zone Configuration
Now that you’ve updated the `terraform-azure-virtual-network` module, you need to pass the `central_law_workspace_id` from the **landing zone module** (`terrafora-azure-component-landing-zone`).

In your **landing zone module**, update the **Virtual Network** module call as follows:

```hcl
module "virtual_network" {
  source                 = "github.com/cloud-era/terraform-azure-virtual-network?ref=init"
  res_vnet_name          = "routable-${var.location}-vnet-01"
  res_location           = var.location
  res_vnet_rg_name       = "eons(${var.eonid})-${var.location}-${var.lz_name}-system-rg"
  res_vnet_address_prefix = var.vnet_address_prefix
  res_dns_servers        = null
  res_tags               = merge(var.tags, var.automation_tags)
  
  # Pass the central LAW workspace ID to the virtual network module
  central_law_workspace_id = var.central_law_workspace_id
}
```

### Step 4: Ensure Central LAW Variable is Defined in the Landing Zone
In the **landing zone module’s `variables.tf`** file, make sure that `central_law_workspace_id` is declared as a variable:

```hcl
variable "central_law_workspace_id" {
  description = "The ID of the Centralized Log Analytics Workspace for diagnostics"
  type        = string
}
```

### Step 5: Provide the `central_law_workspace_id` Input
Finally, when calling the **landing zone module** in your main configuration, pass the `central_law_workspace_id` like this:

```hcl
module "landing_zone" {
  source                 = "github.com/cloud-era/terrafora-azure-component-landing-zone?ref=init"

  # Other parameters
  eonid                  = var.eonid
  location               = var.location
  lz_name                = var.lz_name
  short_name             = var.short_name
  short_env              = var.short_env
  env                    = var.env_name
  vnet_address_prefix     = var.vnet_address_prefix
  tags                   = var.tags

  # Pass Central Log Analytics Workspace ID
  central_law_workspace_id = var.central_law_workspace_id
}
```

### Result:
- **Diagnostic logs** from the Virtual Network will now be sent to the **central Log Analytics Workspace**.
- You can configure the logs and metrics categories based on your needs, such as `NetworkSecurityGroupEvent`, `AllMetrics`, or others.

This approach ensures that your virtual network has both functionality and centralized diagnostics.
