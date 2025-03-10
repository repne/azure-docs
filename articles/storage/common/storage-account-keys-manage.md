---
title: Manage account access keys
titleSuffix: Azure Storage
description: Learn how to view, manage, and rotate your storage account access keys.
services: storage
author: tamram

ms.service: storage
ms.topic: how-to
ms.date: 06/29/2021
ms.author: tamram 
ms.custom: devx-track-azurepowershell
---

# Manage storage account access keys

When you create a storage account, Azure generates two 512-bit storage account access keys. These keys can be used to authorize access to data in your storage account via Shared Key authorization.

Microsoft recommends that you use Azure Key Vault to manage your access keys, and that you regularly rotate and regenerate your keys. Using Azure Key Vault makes it easy to rotate your keys without interruption to your applications. You can also manually rotate your keys.

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

## View account access keys

You can view and copy your account access keys with the Azure portal, PowerShell, or Azure CLI. The Azure portal also provides a connection string for your storage account that you can copy.

### [Portal](#tab/azure-portal)

To view and copy your storage account access keys or connection string from the Azure portal:

1. Navigate to your storage account in the [Azure portal](https://portal.azure.com).

2. Under **Settings**, select **Access keys**. Your account access keys appear, as well as the complete connection string for each key.

3. Locate the **Key** value under **key1**, and click the **Copy** button to copy the account key.

4. Alternately, you can copy the entire connection string. Find the **Connection string** value under **key1**, and click the **Copy** button to copy the connection string.

    :::image type="content" source="media/storage-account-keys-manage/portal-connection-string.png" alt-text="Screenshot showing how to view access keys in the Azure portal":::

### [PowerShell](#tab/azure-powershell)

To retrieve your account access keys with PowerShell, call the [Get-AzStorageAccountKey](/powershell/module/az.Storage/Get-azStorageAccountKey) command.

The following example retrieves the first key. To retrieve the second key, use `Value[1]` instead of `Value[0]`. Remember to replace the placeholder values in brackets with your own values.

```powershell
$storageAccountKey = `
    (Get-AzStorageAccountKey `
    -ResourceGroupName <resource-group> `
    -Name <storage-account>).Value[0]
```

### [Azure CLI](#tab/azure-cli)

To list your account access keys with Azure CLI, call the [az storage account keys list](/cli/azure/storage/account/keys#az_storage_account_keys_list) command, as shown in the following example. Remember to replace the placeholder values in brackets with your own values.

```azurecli-interactive
az storage account keys list \
  --resource-group <resource-group> \
  --account-name <storage-account>
```

---

You can use either of the two keys to access Azure Storage, but in general it's a good practice to use the first key, and reserve the use of the second key for when you are rotating keys.

To view or read an account's access keys, the user must either be a Service Administrator, or must be assigned an Azure role that includes the **Microsoft.Storage/storageAccounts/listkeys/action**. Some Azure built-in roles that include this action are the **Owner**, **Contributor**, and **Storage Account Key Operator Service Role** roles. For more information about the Service Administrator role, see [Classic subscription administrator roles, Azure roles, and Azure AD roles](../../role-based-access-control/rbac-and-directory-admin-roles.md). For detailed information about built-in roles for Azure Storage, see the **Storage** section in [Azure built-in roles for Azure RBAC](../../role-based-access-control/built-in-roles.md#storage).

## Use Azure Key Vault to manage your access keys

Microsoft recommends using Azure Key Vault to manage and rotate your access keys. Your application can securely access your keys in Key Vault, so that you can avoid storing them with your application code. For more information about using Key Vault for key management, see the following articles:

- [Manage storage account keys with Azure Key Vault and PowerShell](../../key-vault/secrets/overview-storage-keys-powershell.md)
- [Manage storage account keys with Azure Key Vault and the Azure CLI](../../key-vault/secrets/overview-storage-keys.md)

## Manually rotate access keys

Microsoft recommends that you rotate your access keys periodically to help keep your storage account secure. If possible, use Azure Key Vault to manage your access keys. If you are not using Key Vault, you will need to rotate your keys manually.

Two access keys are assigned so that you can rotate your keys. Having two keys ensures that your application maintains access to Azure Storage throughout the process.

> [!WARNING]
> Regenerating your access keys can affect any applications or Azure services that are dependent on the storage account key. Any clients that use the account key to access the storage account must be updated to use the new key, including media services, cloud, desktop and mobile applications, and graphical user interface applications for Azure Storage, such as [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/).

If you plan to manually rotate access keys, Microsoft recommends that you set a key expiration policy, and then use queries in Azure Monitor to determine when it's time to rotate an access key.

### Create a key expiration policy

#### [Portal](#tab/azure-portal)

The ability to set a key expiration policy by using the Azure portal is not yet available. You can use either PowerShell or Azure CLI.

#### [PowerShell](#tab/azure-powershell)

To create a key expiration policy, use the [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) command and set the `-KeyExpirationPeriodInDay` parameter to the number of days an access key can be active before it should be rotated.

```powershell
$account = Set-AzStorageAccount -ResourceGroupName <resource-group> -Name `
    <storage-account-name>  -KeyExpirationPeriodInDay <period-in-days> 
```

> [!TIP]
> You can also set the key expiration policy as you create a storage account by setting the `-KeyExpirationPeriodInDay` parameter of the [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) command.

To verify that the policy has been applied, use the `KeyPolicy` property of the [PSStorageAccount](/dotnet/api/microsoft.azure.commands.management.storage.models.psstorageaccount) returned to the `$account` variable in the previous command.

```powershell
$account.KeyPolicy
```

The key expiration period appears in the console output.

> [!div class="mx-imgBorder"]
> ![access key expiration period](./media/storage-account-keys-manage/key-policy-powershell.png)

You might want to rotate existing keys if they've been active for longer than the expiration period. To find out when a key was created, use the `KeyCreationTime` property.

```powershell
$account.KeyCreationTime
```

The access key creation time for both access keys appears in the console output.

> [!div class="mx-imgBorder"]
> ![access key creation times](./media/storage-account-keys-manage/key-creation-time-powershell.png)

#### [Azure CLI](#tab/azure-cli)

To create a key expiration policy on existing storage accounts, use the [az storage account update](/cli/azure/storage/account#az_storage_account_update) command and set the `--key-exp-days` parameter to the number of days an access key can be active before it should be rotated.

```azurecli-interactive
az storage account update \
  -n <storage-account-name> \
  -g <resource-group> --key-exp-days <period-in-days>
```

> [!TIP]
> You can also set the key expiration policy as you create a storage account by setting the `-KeyExpirationPeriodInDay` parameter of the [az storage account create](/cli/azure/storage/account#az_storage_account_create) command.

To verify that the policy has been applied, call the [az storage account show](/cli/azure/storage/account#az_storage_account_show) command, and use the string `{KeyPolicy:keyPolicy}` for the `-query` parameter.

```azurecli-interactive
az storage account show \
  -n <storage-account-name> \
  -g <resource-group-name> \
  --query "{KeyPolicy:keyPolicy}"
```

The key expiration period appears in the console output.

```json
{
  "KeyPolicy": {
    "keyExpirationPeriodInDays": 5
  }
}
```

You might want to rotate existing keys if they've been active for longer than the expiration period. To find out when a key was created, use the  [az storage account show](/cli/azure/storage/account#az_storage_account_show) command, and then use the string `keyCreationTime` for the -query parameter.

```azurecli-interactive
az storage account show \
  -n <storage-account-name> \
  -g <resource-group-name> \
  --query "keyCreationTime"
```

---

### Query for policy violations

If you create a diagnostic setting that [sends logs to Azure Log Analytics](../blobs/monitor-blob-storage.md#send-logs-to-azure-log-analytics) workspace, then you can use an Azure Monitor log query to determine whether a key has expired.

To determine if a key has expired, enter the following query in the **Log search** bar.

```kusto
StorageBlobLogs | where KeyExpiryStatus startsWith "Policy Violated". 
```

You can also create a query that helps you determine if a query is nearing expiration. The following query provides this information.

```kusto
resources 
| where type =~ 'microsoft.storage/storageAccounts'
| extend days = datetime_diff('day', now(), todatetime(parse_json(properties).keyCreationTime))
| extend KeyExpiryStatus = iff(days > 180, "Policy Violated", "")
| project name, days, KeyExpiryStatus  
```

### Rotate access keys

#### [Portal](#tab/azure-portal)

To rotate your storage account access keys in the Azure portal:

1. Update the connection strings in your application code to reference the secondary access key for the storage account.

2. Navigate to your storage account in the [Azure portal](https://portal.azure.com).

3. Under **Settings**, select **Access keys**.

4. To regenerate the primary access key for your storage account, select the **Regenerate** button next to the primary access key.

5. Update the connection strings in your code to reference the new primary access key.

6. Regenerate the secondary access key in the same manner.

#### [PowerShell](#tab/azure-powershell)

To rotate your storage account access keys with PowerShell:

1. Update the connection strings in your application code to reference the secondary access key for the storage account.

2. Call the [New-AzStorageAccountKey](/powershell/module/az.storage/new-azstorageaccountkey) command to regenerate the primary access key, as shown in the following example:

    ```powershell
    New-AzStorageAccountKey -ResourceGroupName <resource-group> `
      -Name <storage-account> `
      -KeyName key1
    ```

3. Update the connection strings in your code to reference the new primary access key.

4. Regenerate the secondary access key in the same manner. To regenerate the secondary key, use `key2` as the key name instead of `key1`.

#### [Azure CLI](#tab/azure-cli)

To rotate your storage account access keys with Azure CLI:

1. Update the connection strings in your application code to reference the secondary access key for the storage account.

2. Call the [az storage account keys renew](/cli/azure/storage/account/keys#az_storage_account_keys_renew) command to regenerate the primary access key, as shown in the following example:

    ```azurecli-interactive
    az storage account keys renew \
      --resource-group <resource-group> \
      --account-name <storage-account> \
      --key primary
    ```

1. Update the connection strings in your code to reference the new primary access key.

2. Regenerate the secondary access key in the same manner. To regenerate the secondary key, use `secondary` as the key name instead of `primary`.

---

> [!NOTE]
> Microsoft recommends using only one of the keys in all of your applications at the same time. If you use Key 1 in some places and Key 2 in others, you will not be able to rotate your keys without some application losing access.

To rotate an account's access keys, the user must either be a Service Administrator, or must be assigned an Azure role that includes the **Microsoft.Storage/storageAccounts/regeneratekey/action**. Some Azure built-in roles that include this action are the **Owner**, **Contributor**, and **Storage Account Key Operator Service Role** roles. For more information about the Service Administrator role, see [Classic subscription administrator roles, Azure roles, and Azure AD roles](../../role-based-access-control/rbac-and-directory-admin-roles.md). For detailed information about Azure built-in roles for Azure Storage, see the **Storage** section in [Azure built-in roles for Azure RBAC](../../role-based-access-control/built-in-roles.md#storage).

## Next steps

- [Azure storage account overview](storage-account-overview.md)
- [Create a storage account](storage-account-create.md)
