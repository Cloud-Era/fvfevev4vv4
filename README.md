The code I provided earlier partially covers the diagnostic settings for the Storage Account, but it can be extended to ensure that it includes **all the diagnostic categories** (blob, queue, table, and file services) and **all log and metric categories** as you see in the portal.

Here's how you can update the `azurerm_monitor_diagnostic_setting` resource to cover all the log and metric categories for **blob**, **queue**, **table**, and **file** services, including audit logs and capacity metrics.

### Updated Diagnostic Setting Code for Storage Account:

```hcl
resource "azurerm_monitor_diagnostic_setting" "storage_diagnostic" {
  name               = format("%s-diagnostics", azurerm_storage_account.storageaccount.name)
  target_resource_id = azurerm_storage_account.storageaccount.id

  log_analytics_workspace_id = var.log_analytics_workspace_id

  # Log categories for Blob Service
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

  log {
    category = "AuditLogs"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  log {
    category = "AllLogs"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  # Metric categories for Blob, Queue, Table, File
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
    category = "Latency"
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
    category = "Ingress"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  # Metrics for specific services
  metrics {
    category = "BlobCapacity"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  metrics {
    category = "QueueCapacity"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  metrics {
    category = "TableCapacity"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }

  metrics {
    category = "FileCapacity"
    enabled  = true
    retention_policy {
      enabled = false
      days    = 0
    }
  }
}
```

### Explanation of the Changes:

- **Log Categories**:
  - Added `AuditLogs` and `AllLogs`, which correspond to the groupings you see in the portal under diagnostic settings.
  - Each of these logs (e.g., `StorageRead`, `StorageWrite`, `StorageDelete`) corresponds to the specific actions you mentioned, ensuring all are enabled.

- **Metric Categories**:
  - The metrics categories include `Transaction`, `Capacity`, `Availability`, `Latency`, `Ingress`, and `Egress`, which are standard metrics for monitoring Azure Storage services.
  - Also added specific capacity metrics for blob, queue, table, and file services (`BlobCapacity`, `QueueCapacity`, `TableCapacity`, and `FileCapacity`), which match what you mentioned seeing in the Azure portal.

### How It Maps to Portal:

- **Blob Service**:
  - Logs: `StorageRead`, `StorageWrite`, `StorageDelete`, `AuditLogs`, and `AllLogs`.
  - Metrics: `Transaction`, `Capacity`, `BlobCapacity`, etc.

- **Queue, Table, File Services**:
  - The relevant logs and metrics for each of these services are covered by categories like `Transaction` and `Capacity` for Queue, Table, and File.

### Apply the Changes:
1. Ensure the variable `log_analytics_workspace_id` is defined and provided in your `terraform.tfvars`.
2. Run `terraform plan` to verify the changes.
3. Run `terraform apply` to implement the diagnostic settings for the storage account.

This update will ensure that **all logs and metrics** for blob, queue, table, and file services are covered and sent to your Log Analytics workspace.
