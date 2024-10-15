To ensure diagnostic settings are enabled for **Queue**, **Table**, and **File** services, alongside **Blob**, you need to explicitly define logs and metrics for each of these services. Here's how you can update the `azurerm_monitor_diagnostic_setting` resource to include the logs and metrics for **Queue**, **Table**, and **File** services.

### Full Diagnostic Settings for Storage Account (Blob, Queue, Table, File)

```hcl
resource "azurerm_monitor_diagnostic_setting" "storage_diagnostic" {
  name               = format("%s-diagnostics", azurerm_storage_account.storageaccount.name)
  target_resource_id = azurerm_storage_account.storageaccount.id

  log_analytics_workspace_id = var.log_analytics_workspace_id

  # Blob Service Log Categories
  log {
    category = "StorageRead"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  log {
    category = "StorageWrite"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  log {
    category = "StorageDelete"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  # Queue Service Log Categories
  log {
    category = "QueueOperation"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  # Table Service Log Categories
  log {
    category = "TableOperation"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  # File Service Log Categories
  log {
    category = "FileOperation"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  # Metrics Categories for all services (Blob, Queue, Table, File)
  metrics {
    category = "Transaction"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  metrics {
    category = "Capacity"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  metrics {
    category = "Availability"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  metrics {
    category = "Ingress"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  metrics {
    category = "Egress"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  metrics {
    category = "Latency"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }
}
```

### Breakdown by Service:

- **Blob Service**:
  - **Logs**: `StorageRead`, `StorageWrite`, `StorageDelete`
  - **Metrics**: `Transaction`, `Capacity`, `Availability`, `Ingress`, `Egress`, `Latency`

- **Queue Service**:
  - **Logs**: `QueueOperation`
  - **Metrics**: Shared with Blob (`Transaction`, `Capacity`, `Availability`, `Ingress`, `Egress`, `Latency`)

- **Table Service**:
  - **Logs**: `TableOperation`
  - **Metrics**: Shared with Blob (`Transaction`, `Capacity`, `Availability`, `Ingress`, `Egress`, `Latency`)

- **File Service**:
  - **Logs**: `FileOperation`
  - **Metrics**: Shared with Blob (`Transaction`, `Capacity`, `Availability`, `Ingress`, `Egress`, `Latency`)

### Explanation:
- **Log Categories**:
  - For **Queue**, **Table**, and **File** services, the relevant logs are `QueueOperation`, `TableOperation`, and `FileOperation`. These logs track read, write, and delete operations for each service.
  
- **Metric Categories**:
  - Common metric categories like `Transaction`, `Capacity`, `Availability`, `Ingress`, `Egress`, and `Latency` are shared across Blob, Queue, Table, and File services. This ensures that performance metrics for all services are captured.

### What this setup does:
- This covers diagnostic settings for **all four services** (Blob, Queue, Table, File) for both **logs** and **metrics**.
- These logs and metrics are sent to your Log Analytics workspace specified by `var.log_analytics_workspace_id`.
  
By adding these log categories and metric categories, the diagnostic settings will now fully cover the **Queue**, **Table**, and **File** services in addition to **Blob**, just like you see in the Azure portal.
