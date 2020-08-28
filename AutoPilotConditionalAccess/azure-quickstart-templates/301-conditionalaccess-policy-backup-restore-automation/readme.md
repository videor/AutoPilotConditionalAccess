# Tutorial:	Backup and Restore Conditional Access policies with approval workflow

Intent: As an IT admin, I want to be able to easily copy-paste conditional access policies from pre-production to production environment.

You can use the conditional access APIs to easily deploy conditional access policies from one environment to another. For example, you can:

As a IT admin, be able to automate a regular snapshot of all conditional access policies within pre-production environment. Select a policy to copy from pre-production. Finally, paste the conditional access policy in production environment. 

This automation can be very useful for: 
- Organizations that manages large numbers of conditional access policies. OR
- Identity partners that manages policies for customers. 

This tutorial shows how to build a [logic app](https://docs.microsoft.com/en-us/azure/logic-apps/) that allows copy-paste of conditional access policies. Specifically, this logic app takes a snapshot of all conditional access policies as a scheduled task within Onedrive. Another logic app monitors any policies being pasted in the production folder of Ondrive. If a new policy is detected, an approval workflow is triggered on Team channel. On approval, the policy is pasted to production environment.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add parameters specific to your organization within logic app.  <br /> 
:heavy_check_mark: Add the recurrence trigger to run a schedule workflow task.<br /> 
:heavy_check_mark: Get client secret from key vault using managed identity.<br /> 
:heavy_check_mark: Get all conditional access policies. <br /> 
:heavy_check_mark: Clears any previous snapshot from Onedrive. <br /> 
:heavy_check_mark: Creates a snapshot of all conditional access policy within pre-production environment. <br /> 
:heavy_check_mark: Add a trigger that monitors PROD OneDrive folder for paste requests.<br /> 
:heavy_check_mark: Add an action that sends a message to Team channel for approving or rejecting these requests.<br /> 
:heavy_check_mark: Add a condition that checks the approval response.<br /> 
:heavy_check_mark: Get client secret from key vault using managed identity.<br /> 
:heavy_check_mark: Add an action that paste the policy in PROD environment.<br /> 
:heavy_check_mark: Add a condition that checks whether the policy was pasted successfully.<br /> 
:heavy_check_mark: Add an action that sends a message to Team channel confirming the outcome of policy deployment.<br /> 

When you're done, your logic app looks like this workflow at a high level:

![High-level finished logic app overview](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Copy0.PNG)

# Pre-requisites

If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you start.

You will also need knowledge of key concepts within Azure logic apps, Onedrive and Teams. 

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

:heavy_check_mark: Logic App 1 for Export of Conditional Access Policies (Copy)

   [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-policy-copy-paste-automation%2Fazuredeploy.json)
   
   [Video Link that takes you through the deployment process for copy/export of conditional access policies](https://www.screencast.com/t/OXNJj2xWaiCM)
   
 :heavy_check_mark: Logic App 2 for Import of Conditional Access Policies (Paste)
  
   [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-policy-copy-paste-automation%2F301-conditionalaccess-policy-paste-automation%2Fazuredeploy.json)

  [Video Link that takes you through the deployment process for paste/import of conditional access policies](https://www.screencast.com/t/ahW29WCqy)
  
1. In the portal, on the **Custom deployment** page, enter or select these values:

   | Property | Value | Description |
   |----------|-------|-------------|
   | **Subscription** | <*Azure-subscription-name*> | The name for the Azure subscription to use |
   | **Resource group** | <*Azure-resource-group-name*> | The name for a new or existing Azure resource group. This example uses `AutoPilotConditionalAccess`. |
   | **Location** |  <*Azure-region-for-all-resources*> | The Azure region to use for all resources, if different from the default value. This example uses the default value, `[resourceGroup().location]`, which is the resource group location. |
   | **Logic App Name** | <*logic-app-name*> | The name to use for your logic app. This example uses `301-conditionalaccess-policy-copy-paste-automation`. |
   ||||

   Here is how the page looks with the values used in this example:

   ![Provide information for quickstart template](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/Deploy-LA-edit.png)

1. When you're done, select **Purchase**.

#### [CLI](#tab/azure-cli)

```azurecli-interactive
read -p "Enter a project name name to use for generating resource names:" projectName &&
read -p "Enter the location, such as 'westus':" location &&
templateUri="https://raw.githubusercontent.com/videor/AutoPilotConditionalAccess/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/azuredeploy.json" &&
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
$templateUri = "https://raw.githubusercontent.com/videor/AutoPilotConditionalAccess/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/azuredeploy.json"

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

1. On the Logic App Designer, click `schedule snapshot of pre-production conditional access policies` box. This example uses a recurrence trigger.

   ![Change the Recurrence trigger's interval and frequency](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Copy1-edit.png)

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Interval** | Yes | 1 | The number of intervals to wait between checks |
   | **Frequency** | Yes | Day | The unit of time to use for the recurrence |
   |||||
      
 This trigger fires every `day`. The **schedule** trigger box shows the recurrence schedule. For more information, see [Schedule tasks and workflows](https://docs.microsoft.com/en-us/azure/connectors/connectors-native-recurrence) and [Workflow actions and triggers](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#recurrence-trigger).
 
# Step 5: Get client secret from key vault using managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Method, URI, Queries, Authentication type, Managed Identity and Audience.

   ![Select "GET client secret from key vault using managed identity" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Copy2-edit.png)

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

   ![Select "GET all conditional access policies" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Copy3-edit.png)

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
      
# Step 7: List files in Onedrive folder.

1. On the Logic App Designer, in the Onedrive connection box, click `List files in folder`. This example uses OneDrive connector:

   ![Select "List files in Onedrive folder" connector for Onedrive](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Copy4-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the connector, provide the criteria for listing all files.

1. Specify the folder for listing files.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Folder** | `/ConditionalAccess/PPE` | The Onedrive folder to list files |
      ||||
 
# Step 8: Clear previous snapshot of PPE environment within Onedrive folder.

1. On the Logic App Designer, in the Onedrive connection box, click `Delete file`. This example uses OneDrive connector:

   ![Select "Delete files in Onedrive folder" connector for Onedrive](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Copy5-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the connector, provide the criteria for deleting files.

1. Specify the file id for deleting files.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **File** | `Id` | The Onedrive file id retrieved from list files operation |
      ||||

# Step 9: Create a new snapshot of PPE environment within Onedrive folder.

1. On the Logic App Designer, in the Onedrive connection box, click `Create file`. This example uses OneDrive connector:

   ![Select "Create files in Onedrive folder" connector for Onedrive](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Copy6-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the connector, provide the criteria for creating files.

1. Specify the folder path, file name and file content for creating files.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Folder Path** | `/ConditionalAccess/PPE` | The Onedrive folder to take snapshot of PPE |
      | **File Name** | `[id] display name.json` | The file name to use for each policy |
      | **File Content** | `Hour` | The unit of time to use for the recurrence |
      ||||    

# Step 10: Deploy logic app to paste the policy in PROD.

1. Select the following image to sign in with your Azure account and deploy the logic app for paste action in the Azure portal. This is only necessary if you have not already deployed the paste logic app as per the instruction in Step 1:

 [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-policy-copy-paste-automation%2F301-conditionalaccess-policy-paste-automation%2Fazuredeploy.json)

When you're done, your logic app looks like this workflow at a high level:

![High-level finished logic app overview](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste0.PNG)

# Step 11: Add a trigger that monitors PROD OneDrive folder for paste requests

1. On the Logic App Designer, in the Onedrive connection box, click `When a file is created`. This example uses OneDrive trigger:

   ![Select "When a new file is pasted" trigger for Onedrive](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste1-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the trigger, provide the criteria for checking all new files.

1. Specify folder, include sub folder, infer content type and how often do you want to check for items


      | Property | Value | Description |
      |----------|-------|-------------|
      | **Folder** | `/ConditionalAccess/PROD` | The Onedrive folder to monitor |
      | **Include sub folder** | `No` | Should the sub folders be included |
      | **Infer content type** | `Yes` | Infer the content type of the file |
      | **How often do you want to check for items** | `1 Minute` | The unit of time to use for monitoring the folder |
      ||||
      
1. Following the trigger, the response is parsed.

# Step 12: Add an action that sends a message to Teams channel for approving or rejecting these requests.

1. On the Logic App Designer, in the Teams connection box, click `Post an adaptive card to team channel and wait for response`. This example uses Teams connector:

   ![Select "Post an adaptive card to Teams channel and wait for a response" connector for Teams](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste2-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel, update card and update message for posting to Teams. Message is shortened for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post approval workflow |
      | **Message** | `Adaptive card message` | The message to send in workflow |
      | **Channel** | `General` | The Teams channel to post approval workflow |
      | **Update card** | `Yes` | Update the adaptive card to show a member of Teams channel has taken an action |
      | **update message** | `Processing requested blueprint deployment` | Update the adaptive card to show a message once an approval action is taken |
      ||||
      
# Step 13: Add a condition that checks the approval response.

1. On the Logic App Designer, select the Condition box `Check for approval of copy-paste action from PPE to PROD`. This example uses response from earlier Teams connector:

1. Specify the Team card response, expression and verify `approve` response in the condition.
    
      ![Select "Condition to check the response from adaptive card that was posted earlier to Teams channel"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste3-edit.png)
    
      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team Card** | `data.action` | The Team card response to evaluate |
      | **Expression** | `is equal to` | The expression to evaluate |
      | **Condition** | `Approve` | response to verify in the condition |
      ||||

# Step 14: Get client secret from key vault using managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Method, URI, Queries, Authentication type, Managed Identity and Audience.

   ![Select "GET client secret from key vault using managed identity" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste4-edit.png)

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

1. The pasted policy is checked to ensure it is set to report-only mode and read-only attributes (`id`, `createdDateTime` and `modifiedDateTime`) are removed.
  
  ![Select "Remove read-only attributes from pasted policy" Data compose action](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste5-edit.png)

# Step 15: Paste the conditional access policy in PROD environment.

1. On the Logic App Designer, in the HTTP connection box, click `Paste conditional access policy`. This example uses HTTP connector.

1. Specify the Method, URI, Headers, Body, Authentication type, Tenant, Audience, Client ID, Credential Type and Secret.

   ![Select "Paste conditional access policy" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste6-edit.png)

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

# Step 16: Add a check to find if the paste operation was successful. If true, fire an alert on Team channel. 

1. On the Logic App Designer, in the conditions connection box, click `check if the conditional access policy was pasted successfully`. This example uses logic app condition evaluation:

   ![Select "check to find if the conditional access policy was pasted successfully" condition](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste7-edit.png)

1. In the condition, provide the criteria for checking the condition.

1. Specify the HTTP response to evaluate, expression and verify it is equal to `204` response in the condition.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Status code** | `Status code` | The status code from previous step to evaluate |
      | **Expression** | `is equal to` | The expression to evaluate |
      | **Condition** | `204` | response to verify in the condition |
      ||||   
  
1. On the Logic App Designer, in the Teams connection box, click `Post to team channel that the copy-paste action was completed successfully`. This example uses Teams connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel and message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post alert |
      | **Channel** | `General` | The Teams channel to post alert |
      | **message** | `message` | Post the adaptive card with an alert message |
      ||||

1. On the Logic App Designer, in the Onedrive connection box, click `Delete file after successful copy-paste action`. This example uses OneDrive connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the connector, provide the criteria for deleting the file.

1. Specify the file id within Ondedrive.


      | Property | Value | Description |
      |----------|-------|-------------|
      | **File** | `File identifier` | The Onedrive file id to delete |
      ||||

# Step 17: If the paste operation was un-successful. Fire an alert on Team channel. 

1. On the Logic App Designer, in the conditions connection box, click `if invalid application id`. This example uses logic app condition evaluation:

   ![Select "check to find if the conditional access policy paste failed" condition](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-copy-paste-automation/images/Paste8-edit.png)

1. In the condition, provide the criteria for checking the condition.

1. Specify the HTTP response to evaluate, expression and verify it is equal to `1034: Policy contains invalid applications` response in the condition.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Error Message** | `error message` | The error message from previous step to evaluate |
      | **Expression** | `contains` | The expression to evaluate |
      | **Condition** | `1034: Policy contains invalid applications` | response to verify in the condition |
      ||||   
  
1. On the Logic App Designer, in the Teams connection box, click `Post message in team channel that policy contains un-supported application ids`. This example uses Teams connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel and message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post alert |
      | **Channel** | `General` | The Teams channel to post alert |
      | **message** | `message` | Post the adaptive card with an alert message |
      ||||



# Forward Looking

Try the following challenge:

:heavy_check_mark: Edit this logic app to send a custom message on Teams channel when the approval workflow selection is Reject copy-paste action.  <br /> 
:heavy_check_mark: Edit this logic app to delete the policy file in PROD Onedrive folder when the approval workflow selection is Reject copy-paste action. <br /> 

Finally, Try cloning the logic and build workflows to also support line of business application ids. If you would like to request a logic app to do this, please send a request on Twitter @Vi_Deora.




