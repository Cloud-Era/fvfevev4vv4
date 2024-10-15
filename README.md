# Data block to fetch the Network Interface associated with the Private Endpoint
data "azurerm_network_interface" "kv_pe_nic" {
  name                = "${azurerm_private_endpoint.kv_pe.name}.nic.${substr(azurerm_private_endpoint.kv_pe.private_service_connection[0].private_connection_resource_id, -36, 36)}"  # Dynamically construct NIC name
  resource_group_name = var.resource_group_name
}

# Diagnostic Settings for Key Vault's Private Endpoint Network Interface (NIC)
resource "azurerm_monitor_diagnostic_setting" "kv_pe_nic_diagnostic" {
  name                       = format("%s-nic-diagnostic", azurerm_private_endpoint.kv_pe.name)
  target_resource_id         = data.azurerm_network_interface.kv_pe_nic.id
  log_analytics_workspace_id = var.central_law_workspace_id

  # Only AllMetrics is available for NICs
  metric {
    category = "AllMetrics"
    enabled  = true
  }

  lifecycle {
    ignore_changes = [
      log_analytics_destination_type
    ]
  }
}
