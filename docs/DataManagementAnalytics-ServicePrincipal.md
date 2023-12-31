# Data Management Landing Zone - Setting up Service Principal

A service principal with *Contributor* and *User Access Administrator* rights needs to be generated for authentication and authorization from GitHub or Azure DevOps to your Azure subscription. This is required to deploy resources to your environment.

## Create Service Principal

First, go to the Azure Portal to find the ID of your subscription. Then start the Cloud Shell or Azure CLI, login to Azure, set the Azure context and execute the following commands to generate the required credentials:

**Azure CLI:**

```sh
# Replace {service-principal-name} and {subscription-id} with your
# Azure subscription id and any name for your service principal.
az ad sp create-for-rbac \
  --name "{service-principal-name}" \
  --role "Contributor" \
  --scopes "/subscriptions/{subscription-id}" \
  --sdk-auth
```

This will generate the following JSON output:

```json
{
  "clientId": "<GUID>",
  "clientSecret": "<GUID>",
  "subscriptionId": "<GUID>",
  "tenantId": "<GUID>",
  (...)
}
```

> **Note:** Take note of the output. It will be required for the next steps.

**Azure PowerShell:**

```powershell
# Ensure you are in the right subscription
Set-AzContext -Subscription "{SubscriptionId or SubscriptionName}"

# Create service principal
$sp = New-AzADServicePrincipal -DisplayName {name}
$sp.PasswordCredentials.SecretText
```
The returned object contains the generated password. Make sure that you store this value somewhere secure to authenticate with the service principal.

> **Note:** Beginning with Az PowerShell module version 7.x, `New-AzADServicePrincipal` no longer assigns the Contributor role to the service principal by default. Follow steps below to grant it the `Contributor` role.

## Adding additional role assigments

For automation purposes, one more role assignments is required for the service principal.
Additional required role assignments include:

| Role Name | Description | Scope |
|:----------|:------------|:------|
| [User Access Administrator](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#user-access-administrator) | Required to assign the managed identity of Purview to the Azure Key Vault. | <div style="width: 31ch">(Resource Scope) `/subscriptions/{datamanagementzone-subscriptionId}`</div> |

To add these role assignments, you can use the [Azure Portal](https://portal.azure.com/) or run the following commands using Azure CLI/Azure Powershell:

**Azure CLI - Add role assignments:**

```sh
# Get Service Principal Object ID
az ad sp list --display-name "{servicePrincipalName}" --query "[].{objectId:objectId}" --output tsv

# Add role assignment
# Resource Scope level assignment
az role assignment create \
  --assignee "{servicePrincipalObjectId}" \
  --role "{roleName}" \
  --scope "{scope}"

# Resource group scope level assignment (below command is required for the Data Landing Zone)
az role assignment create \
  --assignee "{servicePrincipalObjectId}" \
  --role "{roleName}" \
  --resource-group "{resourceGroupName}"
```

**Azure Powershell - Add role assignments:**

```powershell
# Get Service Principal Object ID
$spObjectId = (Get-AzADServicePrincipal -DisplayName "{servicePrincipalName}").id

# Add role assignment
# For Resource Scope level assignment
New-AzRoleAssignment `
  -ObjectId $spObjectId `
  -RoleDefinitionName "{roleName}" `
  -Scope "{scope}"

# For Resource group scope level assignment (below command is required for the Data Landing Zone)
New-AzRoleAssignment `
  -ObjectId $spObjectId `
  -RoleDefinitionName "{roleName}" `
  -ResourceGroupName "{resourceGroupName}"

# For Child-Resource Scope level assignment (below command is required for the Data Landing Zone)
New-AzRoleAssignment `
  -ObjectId $spObjectId `
  -RoleDefinitionName "{roleName}" `
  -ResourceName "{resourceName}" `
  -ResourceType "{resourceType (e.g. 'Microsoft.Network/virtualNetworks/subnets')}" `
  -ParentResource "{parentResource (e.g. 'virtualNetworks/{virtualNetworkName}')" `
  -ResourceGroupName "{resourceGroupName}
```

>[Previous](/docs/DataManagementAnalytics-CreateRepository.md)
>[Next (Option (a) GitHub Actions)](/docs/DataManagementAnalytics-GitHubActionsDeployment.md)
>[Next (Option (b) Azure DevOps)](/docs/DataManagementAnalytics-AzureDevOpsDeployment.md)
