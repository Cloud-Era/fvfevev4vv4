To enable diagnostic settings for blob, queue, table, and file services in your Azure Storage Account, you need to create separate diagnostic settings for each service. The current configuration only enables diagnostics for the storage account itself. Here's how you can modify your code to include diagnostic settings for all services:

1. First, add the following diagnostic settings for the storage account:

```hcl
resource "azurerm_monitor_diagnostic_setting" "storage_account_diagnostic" {
  name               = format("%s-sa-diagnostics", azurerm_storage_account.storageaccount.name)
  target_resource_id = azurerm_storage_account.storageaccount.id
  log_analytics_workspace_id = var.log_analytics_workspace_id

  metric {
    category = "Transaction"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  metric {
    category = "Capacity"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }
}
```

2. Then, add diagnostic settings for blob, queue, table, and file services:

```hcl
resource "azurerm_monitor_diagnostic_setting" "blob_diagnostic" {
  name               = format("%s-blob-diagnostics", azurerm_storage_account.storageaccount.name)
  target_resource_id = "${azurerm_storage_account.storageaccount.id}/blobServices/default"
  log_analytics_workspace_id = var.log_analytics_workspace_id

  log {
    category = "StorageRead"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  log {
    category = "StorageWrite"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  log {
    category = "StorageDelete"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  metric {
    category = "Transaction"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }
}

resource "azurerm_monitor_diagnostic_setting" "queue_diagnostic" {
  name               = format("%s-queue-diagnostics", azurerm_storage_account.storageaccount.name)
  target_resource_id = "${azurerm_storage_account.storageaccount.id}/queueServices/default"
  log_analytics_workspace_id = var.log_analytics_workspace_id

  log {
    category = "StorageRead"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  log {
    category = "StorageWrite"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  log {
    category = "StorageDelete"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  metric {
    category = "Transaction"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }
}

resource "azurerm_monitor_diagnostic_setting" "table_diagnostic" {
  name               = format("%s-table-diagnostics", azurerm_storage_account.storageaccount.name)
  target_resource_id = "${azurerm_storage_account.storageaccount.id}/tableServices/default"
  log_analytics_workspace_id = var.log_analytics_workspace_id

  log {
    category = "StorageRead"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  log {
    category = "StorageWrite"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  log {
    category = "StorageDelete"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  metric {
    category = "Transaction"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }
}

resource "azurerm_monitor_diagnostic_setting" "file_diagnostic" {
  name               = format("%s-file-diagnostics", azurerm_storage_account.storageaccount.name)
  target_resource_id = "${azurerm_storage_account.storageaccount.id}/fileServices/default"
  log_analytics_workspace_id = var.log_analytics_workspace_id

  log {
    category = "StorageRead"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  log {
    category = "StorageWrite"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  log {
    category = "StorageDelete"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }

  metric {
    category = "Transaction"
    enabled  = true
    retention_policy {
      enabled = false
    }
  }
}
```

These configurations will enable diagnostic settings for the storage account itself, as well as for blob, queue, table, and file services. Each service will have its own diagnostic setting, allowing you to collect logs and metrics for read, write, and delete operations, as well as transaction metrics.

Make sure to replace `var.log_analytics_workspace_id` with the actual Log Analytics Workspace ID where you want to send the diagnostic data.

Also, note that the retention policy is set to `enabled = false` for all logs and metrics. If you want to enable retention, you can set `enabled = true` and specify the number of days to retain the data using the `days` parameter.

Citations:
[1] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/11902716/24b993c6-2160-41d3-8935-096be4ea0e36/paste.txt
[2] https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/11902716/9117b432-ef6a-4fe3-bc80-1445dd0166f7/paste.txt
