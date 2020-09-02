# Tutorial:	Alert on Conditional Acess Policy changes

Intent: As an IT admin, I want to be able to easily setup alerts on conditional access policy changes.

You can use the conditional access APIs to manage alerts on policy changes. For example, you can:

As a IT admin, add, update or delete a conditional access policy using conditional APIs or UI and be able to set up alerts that fire when CA policies are changed. 

This automation can be very useful for: 
- Organizations that manages large numbers of conditional access policies. OR
- Identity partners that manages policies for customers. 

This tutorial shows how to build a [logic app](https://docs.microsoft.com/en-us/azure/logic-apps/) that automates policy alerts. Specifically, this logic app monitors the audit logs for policy changes and triggers alert on Teams channel.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add parameters specific to your organization within logic app.  <br /> 
:heavy_check_mark: Add the webhook trigger from audit logs to run a custom workflow task.<br /> 
:heavy_check_mark: Get client secret from key vault using managed identity.<br /> 
:heavy_check_mark: Get audit logs for CRUD operation on conditional access policies.<br /> 
:heavy_check_mark: Add a condition that checks if any CRUD operations were returned.<br /> 
:heavy_check_mark: Fire an alert on Team channel if a policy alert condition is satisfied. <br /> 

When you're done, your logic app looks like this workflow at a high level:

![High-level logic app overview](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-alert-automation/images/Alert0-new.PNG)

# Pre-requisites

If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you start.

### The contents of this repository are unsupported and may or not be current. Replies to questions about unsupported material have the lowest priority.

# Why unsupported?
Unsupported samples and documentation are provided for our fans and partners for training, and feedback only.

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
   
      [Video Link that takes you through the deployment process for Alert on Conditional Access policies](https://www.screencast.com/t/Fw4gKvf5)

1. In the portal, on the **Custom deployment** page, enter or select these values:

   | Property | Value | Description |
   |----------|-------|-------------|
   | **Subscription** | <*Azure-subscription-name*> | The name for the Azure subscription to use |
   | **Resource group** | <*Azure-resource-group-name*> | The name for a new or existing Azure resource group. This example uses `AutoPilotConditionalAccess`. |
   | **Location** |  <*Azure-region-for-all-resources*> | The Azure region to use for all resources, if different from the default value. This example uses the default value, `[resourceGroup().location]`, which is the resource group location. |
   | **Logic App Name** | <*logic-app-name*> | The name to use for your logic app. This example uses `301-conditionalaccess-policy-alert-automation`. |
   ||||

   Here is how the page looks:

   ![Provide information for quickstart template](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-alert-automation/images/Alert-1.PNG)

1. When you're done, select **Review + Create**. Finally select **Create**.

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

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/MI-edit.png).

# Step 3: Update parameters

1. In the left-hand navigation pane, select Logic App designer > Parameters > Replace the default value with Key Vault URI, Client ID and Tenant ID.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LA-parameters-edit.png)


# Step 4: Add the Recurrence trigger

1. On the Logic App Designer, click `recurrence` box. This example uses a recurrence trigger.

   ![Change the Recurrence trigger's interval and frequency](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup1-edit.png)

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Interval** | Yes | 1 | The number of intervals to wait between checks |
   | **Frequency** | Yes | Minute | The unit of time to use for the recurrence |
   |||||
      
 This trigger fires every 1 minute, starting at time provided in `start time`. The **recurrence** box shows the recurrence schedule. For more information, see [Schedule tasks and workflows](https://docs.microsoft.com/en-us/azure/connectors/connectors-native-recurrence) and [Workflow actions and triggers](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#recurrence-trigger).
 
Sometimes, you might want to run operations on data in your workflow, and then use the results in later actions. To save these results so that you can easily reuse or reference them, you can create variables to store those results after processing them. You can create variables only at the top level in your logic app.

By default, the previous **recurrence** action returns the time in seconds when the workflow has started. By converting and storing this value as a range within the audit logs endpoint filter, you make the value easier to reuse later without converting again. Similarly, storing old and new values of conditional access policy JSON, we can reuse these values in different parts of workflow. 

2. On the Logic App Designer, click `Audit logs endpoint for conditional access policies` box. Provide the details for your variable as described here:

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Name** | Yes | URL | The name for your variable. This example uses "URL". |
   | **Type** | Yes | String | The data type for your variable |
   | **Value** | No| MS graph audit logs endpoint. | The initial value for your variable |
   ||||
   
3. On the Logic App Designer, click `Old policy JSON before change is made to conditional access policy` box. Provide the details for your variable as described here:

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Name** | Yes | OldJSON | The name for your variable. This example uses "OldJSON". |
   | **Type** | Yes | String | The data type for your variable |
   | **Value** | No| empty | The initial value for your variable |
   ||||
   
3. On the Logic App Designer, click `New policy JSON after change is made to conditional access policy` box. Provide the details for your variable as described here: 

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Name** | Yes | NewJSON | The name for your variable. This example uses "NewJSON". |
   | **Type** | Yes | String | The data type for your variable |
   | **Value** | No| empty | The initial value for your variable |
   ||||

# Step 5: Get client secret from key vault using managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Method, URI, Queries, Authentication type, Managed Identity and Audience.

   ![Select "GET client secret from key vault using managed identity" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup2-edit.png)

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

# Step 6: Get audit logs for CRUD operation on conditional access policies.

1. On the Logic App Designer, in the HTTP connection box, click `Get audit logs for CRUD operation on conditional access policies`. This example uses HTTP connector.

1. Specify the Method, URI, Headers, Body, Authentication type, Tenant, Audience, Client ID, Credential Type and Secret.

   ![Select "GET audit logs for CRUD operation on conditional access policies" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup3-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Method** | `GET` | Method to call |
      | **URI** | `URL variable` | Audit Logs API v1.0 endpoint |
      | **Headers** | `application/json` | Content-Type |
      | **Authentication type** | `Active Directory OAuth` | Authentication type for App-only flow |
      | **Tenant** | `TenantID` | Tennat ID configured in step 3 |
      | **Audience** | `https://graph.microsoft.com` | MS Graph |
      | **Client ID** | `Client ID` | Client ID configured in step 3 |
      | **Credential Type** | `Secret` | Client Secret  |
      | **Secret** | `value` | Secret value retrieved from key vault |
      ||||
      
# Step 7: Add a condition that checks if any CRUD operations were returned.

1. On the Logic App Designer, in the Condition box, verify response. This example uses response from earlier HTTP connector:

1. Specify the HTTP response to evaluate, expression and verify the greater than or equal to `1` response in the condition.
    
      ![Select "Condition to check the response from get audit logs HTTP connector"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup4-edit.png)
    
      | Property | Value | Description |
      |----------|-------|-------------|
      | **Length** | `length of array` | The length of array response to evaluate |
      | **Expression** | `is greater than or equal to` | The expression to evaluate |
      | **Condition** | `1` | response to verify in the condition |
      ||||

# Step 8: Add a switch statement that checks whether the conditional access policy was added, updated or deleted.

1. On the Logic App Designer, in the HTTP connection box, click `Switch`. This example uses logic app switch statement.

1. Specify the Switch On and Case to evaluate. Specify `Add conditional access policy` for case 1, `Update conditional access policy` for case 2 and `Delete conditional access policy` for case 3.

   ![Select "Check if deployment is successful" condition](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup5-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Switch On** | `Activity display name` | Check the activity display name retrieved from audit logs |
      | **Case 1 equals** | `Add conditional access policy` | Case 1 to check Add operations on conditional access policies |
      | **Case 2 equals** | `Update conditional access policy` | Case 2 to check Update operations on conditional access policies|
      | **Case 3 equals** | `Delete conditional access policy` | Case 3 to check Delete operations on conditional access policies|
      ||||
      
# Step 9: Add a check to find if the newly created conditional access policy has block controls. If it does, fire an alert on Team channel<br /> 

1. On the Logic App Designer, in the Teams connection box, click `check if the conditional access policy has been created with block controls`. This example uses Teams connector:

   ![Select "fire an alert on Team channel if a new block policy is created" trigger for Teams](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-alert-automation/images/Alert9-edit.png)

1. In the condition, provide the criteria for checking the condition.

1. Specify the HTTP response to evaluate, expression and verify it contains `block` response in the condition.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Built in control** | `built in control` | The built in control value within new policy JSON to evaluate |
      | **Expression** | `contains` | The expression to evaluate |
      | **Condition** | `block` | response to verify in the condition |
      ||||   
  
1. On the Logic App Designer, in the Teams connection box, click `Post alert to Team channel on new block policy creation`. This example uses Teams connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel and message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post alert |
      | **Channel** | `General` | The Teams channel to post alert |
      | **message** | `message` | Post the adaptive card with an alert message |
      ||||


# Step 10: Add a check to find if a conditional access policy has been updated from grant access to block controls. If it does, fire an alert on Team channel<br /> 

1. On the Logic App Designer, in the Onedrive connection box, click `check if the conditional access policy has been updated from Grant to block access`. This example uses Team connector:

   ![Select "fire an alert on Team channel when a policy is updated from grant to block" trigger for Teams](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-alert-automation/images/Alert10-alert.png)

1. In the condition, provide the criteria for checking the condition.

1. Specify the HTTP response to evaluate, expression and verify the old policy JSON did not contain `block` control and the updated new policy JSON contains `block` control.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Built in control** | `built in control` | The built in control value within new policy JSON to evaluate |
      | **Expression** | `does not contain` | The expression to evaluate |
      | **Condition** | `block` | response to verify in the condition |
      | **Built in control** | `built in control` | The built in control value within new policy JSON to evaluate |
      | **Expression** | `contains` | The expression to evaluate |
      | **Condition** | `block` | response to verify in the condition |
      ||||   
  
1. On the Logic App Designer, in the Teams connection box, click `Post alert to Team channel on potentially disruptive update`. This example uses Teams connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel and message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post alert |
      | **Channel** | `General` | The Teams channel to post alert |
      | **message** | `message` | Post the adaptive card with an alert message |
      ||||
