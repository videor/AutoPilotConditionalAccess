# Tutorial: One-click deploy policy blueprints using Conditional Access APIs

Intent: As an IT admin, I want to be able to easily deploy conditional access policies to large number of branch offices and subsidiaries.

You can use the conditional access APIs to manage policy blueprints that you can make available to your branch offices and subsidiaries. For example, you can:

As a central IT admin, add a new policy blueprint using conditional access schema to a shared onedrive. All subsidieries and branch offices who have been given access to this onedrive will receive notification to create a new conditional access policy based on the policy blueprint. Admins in branch offices and subsidiaries can approve or reject this blueprint. If approved, the policy blueprint will be deployed.

This automation can be very useful for: 
- Organizations that manages large numbers of Azure AD tenants representing branch offices and subsidiaries.
- Identity partners that manages policy blueprints for a large number of customers. OR
- Mergers and acquisitions. 

This tutorial shows how to build a [logic app](../logic-apps/logic-apps-overview.md) that automates policy blueprint deployment to branch offices and subsidiaries based on an approval-based workflow. Specifically, this logic app monitors a shared onedrive account for new policy blueprints submissions, sends requests for approval, and deploys the policy blueprint to approved branch offices and subsidiaries.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add parameters specific to your organization within logic app.  <br /> 
:heavy_check_mark: Add a trigger that monitors OneDrive folder for policy blueprint requests.<br /> 
:heavy_check_mark: Add an action that sends a message to Team channel for approving or rejecting these requests.<br /> 
:heavy_check_mark: Add a condition that checks the approval response.<br /> 
:heavy_check_mark: Get client secret from key vault using managed identity.<br /> 
:heavy_check_mark: Add an action that deploys the policy blueprint request.<br /> 
:heavy_check_mark: Add a condition that checks whether the policy blueprint was deployed successfully.<br /> 
:heavy_check_mark: Add an action that sends a message to Team channel confirming the outcome of policy blueprint deployment.<br /> 


When you're done, your logic app looks like this workflow at a high level:

![High-level finished logic app overview](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/High-Level-View-1.PNG)

# Pre-requisites

If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you start.

# Step 1: Deploy this logic app to your organization

Follow the option that you want to use for deploying the quickstart template:

| Option | Description |
|--------|-------------|
| [Azure portal](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-portal#deploy-template) | If your Azure environment meets the prerequisites, and you're familiar with using ARM templates, these steps help you sign in directly to Azure and open the quickstart template in the Azure portal. For more information, see [Deploy resources with ARM templates and Azure portal](../azure-resource-manager/templates/deploy-portal.md). |
| [Azure CLI](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-cli#deploy-template) | The Azure command-line interface (Azure CLI) is a set of commands for creating and managing Azure resources. To run these commands, you need Azure CLI version 2.6 or later. To check your CLI version, type `az --version`. For more information, see these topics: <p><p>- [What is Azure CLI](https://docs.microsoft.com/cli/azure/what-is-azure-cli?view=azure-cli-latest) <br>- [Get started with Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) |
| [Azure PowerShell](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-powershell#deploy-template) | Azure PowerShell provides a set of cmdlets that use the Azure Resource Manager model for managing your Azure resources. For more information, see these topics: <p><p>- [Azure PowerShell Overview](https://docs.microsoft.com/powershell/azure/azurerm/overview) <br>- [Introducing the Azure PowerShell Az module](https://docs.microsoft.com/powershell/azure/new-azureps-module-az) <br>- [Get started with Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps) |
| [Azure Resource Management REST API](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=rest-api#deploy-template) | Azure provides Representational State Transfer (REST) APIs, which are service endpoints that support HTTP operations (methods) that you use to create, retrieve, update, or delete access to service resources. For more information, see [Get started with Azure REST API](https://docs.microsoft.com/rest/api/azure/). |
|||

<a name="deploy-azure-portal"></a>

#### [Portal](#tab/azure-portal)

1. Select the following image to sign in with your Azure account and open the quickstart template in the Azure portal:

   [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-policy-blueprint-automation%2Fazuredeploy.json)

1. In the portal, on the **Create a logic app using a template** page, enter or select these values:

   | Property | Value | Description |
   |----------|-------|-------------|
   | **Subscription** | <*Azure-subscription-name*> | The name for the Azure subscription to use |
   | **Resource group** | <*Azure-resource-group-name*> | The name for a new or existing Azure resource group. This example uses `Check-Azure-Status-RG`. |
   | **Region** | <*Azure-region*> | The Azure datacenter region to use your logic app. This example uses `West US`. |
   | **Logic App Name** | <*logic-app-name*> | The name to use for your logic app. This example uses `Check-Azure-Status-LA`. |
   | **Test Uri** | <*test-URI*> | The URI for the service to call based on a specific schedule. This example uses `https://status.azure.com/en-us/status/`, which is the Azure status page. |
   | **Location** |  <*Azure-region-for-all-resources*> | The Azure region to use for all resources, if different from the default value. This example uses the default value, `[resourceGroup().location]`, which is the resource group location. |
   ||||

   Here is how the page looks with the values used in this example:

   ![Provide information for quickstart template](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/Deployment.PNG)

1. When you're done, select **Review + create**.

1. Continue with the steps in [Review deployed resources](#review-deployed-resources).

#### [CLI](#tab/azure-cli)

```azurecli-interactive
read -p "Enter a project name name to use for generating resource names:" projectName &&
read -p "Enter the location, such as 'westus':" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-logic-app-create/azuredeploy.json" &&
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
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-logic-app-create/azuredeploy.json"

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

This logic app uses user assigned managed identity for getting secrets from KeyVault in order to call conditional access APIs. Please look at [Authenticate your logic app to Azure AD with the right permissions](https://github.com/videor/AutoPilotConditionalAccess/tree/master/AutoPilotConditionalAccess/azure-quickstart-templates/docs) on how to create managed identity. 

To learn more on how to use managed identities within Logic App please look at [**Logic Apps and Managed Identities**](https://docs.microsoft.com/en-us/azure/logic-apps/create-managed-service-identity) .

Once you have deployed the automation template from this repository. Connect the user-Assigned Managed Identity to your Logic App by doing the following:

1. In the left-hand navigation pane, select the Logic App service, and then select Identity > User Assigned > Select Add.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-5.PNG)

2. Select the User-assigned managed identity from the context pane that appears on the right, select Add.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-6.PNG)

3. You should see the screen below with your registered user-assigned managed identity connected to your logic app. 

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-7.PNG).

# Step 3: Update parameters value and click Save. 

Once you have connected your user-Assigned Managed Identity to your Logic App add the parameters within your logic app:

1. In the left-hand navigation pane, select the Logic App you deployed, and then select Logic App designer > Parameters > Replace the default value with yours for (a) Key Vault URI (b) Client ID of your app registration (c) The Group Object ID you will like to automate for exclusion and (d) Tenant ID.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-breakglass-automation/images/LogicApp-301-CA-breakglass-automation-8.PNG)

2. Now open the Logic App step for Get Client Secret from Key Vault using Managed Identity and select the Managed Identity you linked to the logic app from the drop down.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-breakglass-automation/images/LogicApp-301-CA-breakglass-automation-9.PNG)

# Step 4: Add a trigger that monitors OneDrive folder for policy blueprint requests

1. On the Logic App Designer, in the Onedrive connection box, click `Add new`. This example uses OneDrive trigger:

   ![Select "When a new file arrives" trigger for Onedrive](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint1-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the trigger, provide the criteria for checking all new files.

   1. Specify the folder, interval, and frequency for checking files.

      ![Specify folder, interval, and frequency for checking mails](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint2-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Folder** | `/ConditionalAccess/Blueprints` | The Onedrive folder to monitor |
      | **Interval** | `1` | The number of intervals to wait between checks |
      | **Frequency** | `Hour` | The unit of time to use for the recurrence |
      ||||
      
1. Following the trigger, the response is parsed.

# Step 5: Add an action that sends a message to Teams channel for approving or rejecting these requests.

1. On the Logic App Designer, in the Teams connection box, click `Add new`. This example uses Teams connector:

   ![Select "Post an adaptive card to Teams channel and wait for a response" connector for Teams](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint3-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

   1. Specify the Team, channel, update card and update message for posting to Teams.

      ![Specify Team, channel, update card and update message for posting to Teams](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint4-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post approval workflow |
      | **Channel** | `Blueprints` | The Teams channel to post approval workflow |
      | **Update card** | `Yes` | Update the adaptive card to show a member of Teams channel has taken an action |
      | **update message** | `Processing requested blueprint deployment` | Update the adaptive card to show a message once an approval action is taken |
      ||||
      
# Step 6: Add a condition that checks the approval response.

1. On the Logic App Designer, in the Condition box, verify response. This example uses response from earlier Teams connector:

1. Specify the Team card response, expression and verify `approve` response in the condition.
    
      ![Select "Condition to check the response from adaptive card that was posted earlier to Teams channel"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint5-edit.png)
    
      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team Card** | `data.action` | The Team card response to evaluate |
      | **Expression** | `is equal to` | The expression to evaluate |
      | **Condition** | `Approve` | response to verify in the condition |
      ||||

# Step 7: Get client secret from key vault using managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Method, URI, Queries, Authentication type, Managed Identity and Audience.

   ![Select "GET client secret from key vault using managed identity" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint6-edit.png)

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

1. The blueprint is checked to ensure it is enabled in report-only mode and read-only attributes (`id`, `createdDateTime` and `modifiedDateTime`) are removed.

# Step 8: Add an action that deploys the policy blueprint request.

1. On the Logic App Designer, in the HTTP connection box, click `Deploy new conditional access policy blueprint to branch office and subsidiaries`. This example uses HTTP connector.

1. Specify the Method, URI, Headers, Body, Authentication type, Tenant, Audience, Client ID, Credential Type and Secret.

   ![Select "GET client secret from key vault using managed identity" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint7-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Method** | `POST` | Method to call |
      | **URI** | `https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies` | Conditional Access API v1.0 endpoint |
      | **Headers** | `application/json` | Content-Type |
      | **Body** | `Outputs` | Output from blueprint JSON after clensing from previous step |
      | **Authentication type** | `Active Directory OAuth` | Authentication type for App-only flow |
      | **Tenant** | `TenantID` | Tennat ID configured in step 3 |
      | **Audience** | `https://graph.microsoft.com` | MS Graph |
      | **Client ID** | `Client ID` | Client ID configured in step 3 |
      | **Credential Type** | `Secret` | Client Secret  |
      | **Secret** | `value` | Secret value retrieved from key vault |
      ||||

# Step 8: Add a condition that checks whether the policy blueprint was deployed successfully and send message in Team channel.

1. On the Logic App Designer, in the HTTP connection box, click `Check if deployment is successful`. This example uses logic app conditions.

1. Specify the Team, channel and message for posting to Team channel. The message is shorterned for readability.

   ![Select "Check if deployment is successful" condition](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint8-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post the outcome of deployment |
      | **Channel** | `Blueprints` | The Teams channel to post the outcome of deployment |
      | **Message** | `Adaptive card message` | Update the adaptive card to show a message depending on the outcome of deployment |
      ||||
