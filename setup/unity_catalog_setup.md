# Unity Catalog Storage Setup (No Keys, No Mounts)

This is the governed access chain used instead of deprecated DBFS mounts:

```
Azure Access Connector (managed identity)
        → Storage Credential
        → External Location
        → Unity Catalog Volume
```

## 1. Azure side — Access Connector

1. In the Azure portal, create an **Access Connector for Azure Databricks** (it provides a managed identity).
2. On the ADLS Gen2 storage account, grant the connector's managed identity the **Storage Blob Data Contributor** role.

## 2. Databricks side — Storage Credential

Catalog Explorer → External Data → Credentials → Create credential, pointing to the Access Connector's resource ID. (Or via SQL:)

```sql
-- Example (replace with your connector resource ID)
CREATE STORAGE CREDENTIAL adls_credential
  WITH (AZURE_MANAGED_IDENTITY
    (ACCESS_CONNECTOR_ID = '/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.Databricks/accessConnectors/<connector-name>'));
```

## 3. External Location

```sql
CREATE EXTERNAL LOCATION landing_location
  URL 'abfss://<container>@<storage-account>.dfs.core.windows.net/<path>'
  WITH (STORAGE CREDENTIAL adls_credential);
```

## 4. Unity Catalog Volume

```sql
CREATE EXTERNAL VOLUME dlt_demo_ws1.default.landing_vol
  LOCATION 'abfss://<container>@<storage-account>.dfs.core.windows.net/<path>/landing';
```

Result: a clean, governed, auditable path — `/Volumes/dlt_demo_ws1/default/landing_vol/` — with zero secrets in any notebook or pipeline code.

> Note: placeholders (`<sub-id>`, `<storage-account>`, etc.) are intentional — replace with your own resource names. TODO: verify these commands match the exact steps used (UI vs SQL) and adjust.
