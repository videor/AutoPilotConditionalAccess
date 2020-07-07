# Tutorial:	Emergency Account management within conditional acess policies

Intent: As an IT admin, I want to be able to automate management of emergency account within conditional access policies.

You can use the conditional access APIs to automate management of emergency accounts within conditional access policies. For example, you can:

As a IT admin, be able to automatically exclude emergency accounts from all new conditional access policies. Auto-remediate when a conditional access policy has been updated accidently or maliciously to remove emergency account. Finally, get an alert on missing emergency account and remediation taken for a conditional access policy. 

This automation can be very useful for: 
- Organizations that manages large numbers of conditional access policies. OR
- Identity partners that manages policies for customers. 

This tutorial shows how to build a [logic app](https://docs.microsoft.com/en-us/azure/logic-apps/) that automates emergency account management. Specifically, this logic app retrieves all conditional access policies as a scheduled task. Checks if the policy has necessary exclusions, and if it doesn't, updates the conditional access policy and triggers an alert on Team channel.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add parameters specific to your organization within logic app.  <br /> 
:heavy_check_mark: Add the recurrence trigger to run a schedule workflow task.<br /> 
:heavy_check_mark: Get client secret from key vault using managed identity.<br /> 
:heavy_check_mark: Get all conditional access policies. <br /> 
:heavy_check_mark: Add a condition to check for missing emergency account. <br /> 
:heavy_check_mark: Add an exclusion for emergency account if it is missing within the conditional access policy. <br /> 
:heavy_check_mark: Add a condition that checks if the policy was updated successfully. <br /> 
:heavy_check_mark: Fire an alert on Team channel confirming the remediation taken and status. <br /> 

When you're done, your logic app looks like this workflow at a high level:

![High-level finished logic app overview](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-emergency-account-automation/images/EmergencyAccount0.PNG)

# Pre-requisites

If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you start.

# Step 1: Deploy this logic app to your organization

Follow the option that you want to use for deploying the quickstart template:

| Option | Description |
|--------|-------------|
| [Azure portal](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-portal#deploy-template) | If your Azure environment meets the prerequisites, and you're familiar with using ARM templates, these steps help you sign in directly to Azure and open the quickstart template in the Azure portal. For more information, see [Deploy resources with ARM templates and Azure portal](../azure-resource-manager/templates/deploy-portal.md). |
| [Azure CLI](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-cli#deploy-template) | The Azure command-line interface (Azure CLI) is a set of commands for creating and managing Azure resources. To run these commands, you need Azure CLI version 2.6 or later. To check your CLI version, type `az --version`. For more information, see these topics: <p><p>- [What is Azure CLI](https://docs.microsoft.com/cli/azure/what-is-azure-cli?view=azure-cli-latest) <br>- [Get started with Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) |
| [Azure PowerShell](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-powershell#deploy-template) | Azure PowerShell provides a set of cmdlets that use the Azure Resource Manager model for managing your Azure resources. For more information, see these topics: <p><p>- [Azure PowerShell Overview](https://docs.microsoft.com/powershell/azure/azurerm/overview) <br>- [Introducing the Azure PowerShell Az module](https://docs.microsoft.com/powershell/azure/new-azureps-module-az) <br>- [Get started with Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps) |
|||

<a name="deploy-azure-portal"></a>

#### [Azure Portal](#tab/azure-portal)

1. Select the following image to sign in with your Azure account and open the logic app in the Azure portal:

   [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-policy-alert-automation%2Fazuredeploy.json)

1. In the portal, on the **Custom deployment** page, enter or select these values:

   | Property | Value | Description |
   |----------|-------|-------------|
   | **Subscription** | <*Azure-subscription-name*> | The name for the Azure subscription to use |
   | **Resource group** | <*Azure-resource-group-name*> | The name for a new or existing Azure resource group. This example uses `AutoPilotConditionalAccess`. |
   | **Location** |  <*Azure-region-for-all-resources*> | The Azure region to use for all resources, if different from the default value. This example uses the default value, `[resourceGroup().location]`, which is the resource group location. |
   | **Logic App Name** | <*logic-app-name*> | The name to use for your logic app. This example uses `301-conditionalaccess-policy-alert-automation`. |
   ||||

   Here is how the page looks with the values used in this example:

   ![Provide information for quickstart template](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/Deploy-LA-edit.png)

1. When you're done, select **Purchase**.

#### [CLI](#tab/azure-cli)

```azurecli-interactive
read -p "Enter a project name name to use for generating resource names:" projectName &&
read -p "Enter the location, such as 'westus':" location &&
templateUri="https://raw.githubusercontent.com/videor/AutoPilotConditionalAccess/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-alert-automation/azuredeploy.json" &&
resourceGroupName="${projectName}rg" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

For more information, see these topics:

* [Azure CLI: az deployment group](https://docs.microsoft.com/cli/azure/deployment/group)
* [Deploy resources with ARM templates and Azure CLI](../azure-resource-manager/templates/deploy-cli.md)

#### [PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name to use for generating resource names"
$location = Read-Host -Prompt "Enter the location, such as 'westus'"
$templateUri = "https://raw.githubusercontent.com/videor/AutoPilotConditionalAccess/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-alert-automation/azuredeploy.json"

$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```

For more information, see these topics:

* [Azure PowerShell: New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup)
* [Azure PowerShell: New-AzResourceGroupDeployment](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroupdeployment)
* [Deploy resources with ARM templates and Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)


# Step 2: Authenticate your logic app to Azure AD with the right permissions

This logic app uses managed identity for getting secrets from key vault in order to call conditional access APIs. Please look at [Authenticate your logic app to Azure AD with the right permissions](https://github.com/videor/AutoPilotConditionalAccess/tree/master/AutoPilotConditionalAccess/azure-quickstart-templates/docs) on how to create key vault and connect to managed identity. To learn more on how to use managed identities within Logic App please look at [**Logic Apps and Managed Identities**](https://docs.microsoft.com/en-us/azure/logic-apps/create-managed-service-identity) .

1. In the left-hand navigation pane, select Identity > User Assigned > Select Add.

2. Select the User-assigned managed identity from the context pane that appears on the right, select Add.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprintMI-edit.png).

# Step 3: Update parameters

1. In the left-hand navigation pane, select Logic App designer > Parameters > Replace the default value with Key Vault URI, Client ID and Tenant ID.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint-parameters-edit.png)


# Step 4: Add the Recurrence trigger

1. On the Logic App Designer, click `schedule automation of emergency account management within conditional access` box. This example uses a recurrence trigger.

   ![Change the Recurrence trigger's interval and frequency](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-emergency-account-automation/images/EmergencyAccount1-edit.png)

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Interval** | Yes | 1 | The number of intervals to wait between checks |
   | **Frequency** | Yes | Minute | The unit of time to use for the recurrence |
   |||||
      
 This trigger fires every `1 minute`. The **schedule** trigger box shows the recurrence schedule. For more information, see [Schedule tasks and workflows](https://docs.microsoft.com/en-us/azure/connectors/connectors-native-recurrence) and [Workflow actions and triggers](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#recurrence-trigger).
 
# Step 5: Get client secret from key vault using managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Method, URI, Queries, Authentication type, Managed Identity and Audience.

   ![Select "GET client secret from key vault using managed identity" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-emergency-account-automation/images/EmergencyAccount2-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Method** | `GET` | Method to call |
      | **URI** | `AutoPilotConditionalAccessKeyVaultClientCredentials` | The key vault parameter URI configured in step 3  |
      | **Queries** | `2016-10-01` | api-version |
      | **Authentication type** | `Managed Identity` | Authentication type is managed identity |
      | **Managed Identity** | `AutoPilotCAUAI1` | User assigned managed identity connected in step 2 |
      | **Audience** | `https://vault.azure.net` | Key vault |
      ||||
    
1. Response from Key vault is parsed.

# Step 6: Get all conditional access policies.

1. On the Logic App Designer, in the HTTP connection box, click `Get all conditional access policies`. This example uses HTTP connector.

1. Specify the Method, URI, Headers, Body, Authentication type, Tenant, Audience, Client ID, Credential Type and Secret.

   ![Select "GET all conditional access policies" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-emergency-account-automation/images/EmergencyAccount3-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Method** | `GET` | Method to call |
      | **URI** | `URL variable` | Conditional Access APIs v1.0 endpoint |
      | **Headers** | `application/json` | Content-Type |
      | **Authentication type** | `Active Directory OAuth` | Authentication type for App-only flow |
      | **Tenant** | `TenantID` | Tennat ID configured in step 3 |
      | **Audience** | `https://graph.microsoft.com` | MS Graph |
      | **Client ID** | `Client ID` | Client ID configured in step 3 |
      | **Credential Type** | `Secret` | Client Secret  |
      | **Secret** | `value` | Secret value retrieved from key vault |
      ||||
      
# Step 7: Add a condition that checks if any policy has missing exclusion for emergency account.

1. On the Logic App Designer, in the Condition box, verify response. This example uses response from earlier HTTP connector:

1. Specify the HTTP response to evaluate, expression and verify it contains `EmergencyAccountsGroupObjectID` in the condition.
    
      ![Select "Condition to check if policy has exclusion for emergency account"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-emergency-account-automation/images/EmergencyAccount4-edit.png)
    
      | Property | Value | Description |
      |----------|-------|-------------|
      | **Exclude Groups** | `Exclude groups` | The exclude group attribute to evaluate within policy JSON |
      | **Expression** | `Contains` | The expression to evaluate |
      | **Condition** | `EmergencyAccountsGroupObjectID` | response to verify in the condition |
      ||||

# Step 8: Add an exclusion for emergency account if it is missing within the conditional access policy.

1. On the Logic App Designer, in the HTTP connection box, click `Exclude emergency accounts within conditional access policies`. This example uses HTTP connector.

1. Specify the Method, URI, Headers, Body, Authentication type, Tenant, Audience, Client ID, Credential Type and Secret.

   ![Select "Exclude emergency accounts within conditional access policies" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-emergency-account-automation/images/EmergencyAccount5-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Method** | `PATCH` | Method to call |
      | **URI** | `URL variable` | Conditional Access APIs v1.0 endpoint |
      | **Headers** | `application/json` | Content-Type |
      | **Body** | `json body with exclude groups condition` | Updated JSON to patch the conditional access policy  |
      | **Authentication type** | `Active Directory OAuth` | Authentication type for App-only flow |
      | **Tenant** | `TenantID` | Tennat ID configured in step 3 |
      | **Audience** | `https://graph.microsoft.com` | MS Graph |
      | **Client ID** | `Client ID` | Client ID configured in step 3 |
      | **Credential Type** | `Secret` | Client Secret  |
      | **Secret** | `value` | Secret value retrieved from key vault |
      ||||
      
# Step 9: Add a check to find if the conditional access policy update was successful. If true, fire an alert on Team channel. 

1. On the Logic App Designer, in the conditions connection box, click `check if the conditional access policy was updated successfully`. This example uses logic app condition evaluation:

   ![Select "check to find if the conditional access policy update was successful" condition](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-emergency-account-automation/images/EmergencyAccount6-edit.png)

1. In the condition, provide the criteria for checking the condition.

1. Specify the HTTP response to evaluate, expression and verify it is equal to `204` response in the condition.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Status code** | `Status code` | The status code from previous step to evaluate |
      | **Expression** | `is equal to` | The expression to evaluate |
      | **Condition** | `204` | response to verify in the condition |
      ||||   
  
1. On the Logic App Designer, in the Teams connection box, click `Post a message to Team channel notifying remediation for emergency account`. This example uses Teams connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel and message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post alert |
      | **Channel** | `Emergency Accounts` | The Teams channel to post alert |
      | **message** | `message` | Post the adaptive card with an alert message |
      ||||

1. On the Logic App Designer, in the Teams connection box, click `Post a message to Team channel that remediation failed but scheduled for retry`. This example uses Teams connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel and message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post alert |
      | **Channel** | `Emergency Accounts` | The Teams channel to post alert |
      | **message** | `message` | Post the adaptive card with an alert message |
      ||||

