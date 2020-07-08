# Tutorial:	Business Continuity with Contingency Policies and Conditional Access

Intent: As an IT admin, I want to be able to easily fail-over to contingency policies during service issues.

You can use the conditional access APIs to manage failover to contingency policies. For example, you can:

As a IT admin, trigger fail-over and post an approval workflow to contingency plan Team channel. Members of contingency plan can approve or reject the fail-over. If approved, the contingency policy is enabled and relevant operational policies are disabled.

This automation can be very useful for: 
- Organizations that manages large numbers of conditional access policies. OR
- Identity partners that manages policies for customers. 

This tutorial shows how to build a [logic app](https://docs.microsoft.com/en-us/azure/logic-apps/) that automates fail-over to contingency policies. Specifically, this logic app is triggired with a specific disruption tag. This sends an alert on Teams channel with list of contingency policies specific to that disruption. Admins, can individually select and approve fail-over.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add parameters specific to your organization within logic app.  <br /> 
:heavy_check_mark: Add the HTTP trigger with disruption tag to run the workflow task.<br /> 
:heavy_check_mark: Get client secret from key vault using managed identity.<br /> 
:heavy_check_mark: Get all conditional access policies.<br /> 
:heavy_check_mark: Add a condition that checks the policy for disruption tag and enable in emergency tag.<br /> 
:heavy_check_mark: Fire an approval workflow on Team channel if a policy includes disruption tag and enable in emergency tag.<br /> 
:heavy_check_mark: Add a condition that checks the approval response.<br /> 
:heavy_check_mark: Add an action that on approval, enables contingency policies. <br /> 
:heavy_check_mark: Add a condition that checks whether the contingency policies were enabled successfully. <br /> 
:heavy_check_mark: Add an action that sends a message to Team channel confirming the outcome of enabling contingency policies. <br /> 
:heavy_check_mark: Fire an approval workflow on Team channel for disabling operational policy with disruption tag.<br /> 
:heavy_check_mark: Add a condition that checks the approval response.<br /> 
:heavy_check_mark: Add an action that on approval, disables relevant operational policies. <br /> 
:heavy_check_mark: Add a condition that checks whether the operational policies were disabled successfully. <br /> 
:heavy_check_mark: Add an action that sends a message to Team channel confirming the outcome of disabling operational policies. <br /> 

When you're done, your logic app looks like this workflow at a high level:

![High-level finished logic app overview](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency0.PNG)

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


# Step 4: Add the HTTP trigger

1. On the Logic App Designer, click `When a HTTP request is received` box. This example uses a HTTP trigger.

   ![Change the HTTP trigger](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency1-edit.png)

   | Property | Value | Description |
   |----------|-------|-------------|
   | **Request Body JSON Schema** | disruption tag | The json schema to capture disruption tag |
   | **Method** | POST | The HTTP method to call this logic app |
   |||||
      
This trigger fires when a `HTTP` request is received at the HTTP POST URL endpoint of the logic app. 
 
Sometimes, you might want to run operations on data in your workflow, and then use the results in later actions. To save these results so that you can easily reuse or reference them, you can create variables to store those results after processing them. You can create variables only at the top level in your logic app.

By default, the previous **HTTP** action returns the disruption tag when the workflow is started. By storing this value as a variable, you make the value easier to reuse later. 

2. On the Logic App Designer, click `Disruption Tag` box. Provide the details for your variable as described here:

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Name** | Yes | Disruption tag | The name for your variable. This example uses "Disruption tag". |
   | **Type** | Yes | String | The data type for your variable |
   | **Value** | Yes| Disruption tag retrieved from previous HTTP request. | The initial value for your variable |
   ||||
   

# Step 5: Get client secret from key vault using managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Method, URI, Queries, Authentication type, Managed Identity and Audience.

   ![Select "GET client secret from key vault using managed identity" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency2-edit.png)

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

   ![Select "GET all conditional access policies" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency3-edit.png)

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
      
# Step 7: Add a condition that checks if any policy has disruption tag and enable in emergency.

1. On the Logic App Designer, in the Condition box, verify response. This example uses response from earlier HTTP connector:

1. Specify the HTTP response to evaluate, expression and verify it contains `Disruption tag`  and `Enable tag` in the condition.
    
      ![Select "Condition to check if policy has disruption tag and Enable tag"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency4-edit.png)
    
      | Property | Value | Description |
      |----------|-------|-------------|
      | **Display name** | `Display name` | The display name attribute to evaluate within policy JSON |
      | **Expression** | `Contains` | The expression to evaluate |
      | **Condition** | `Disruption tag` | response to verify in the condition |
      | **Display name** | `Display name` | The display name attribute to evaluate within policy JSON |
      | **Expression** | `Contains` | The expression to evaluate |
      | **Condition** | `Enable tag` | response to verify in the condition |
      ||||

# Step 8: Add an action that sends a message to Teams channel for approving or rejecting enabling contingency policies request.

1. On the Logic App Designer, in the Teams connection box, click `Post an Adaptive card to Teams channel and wait for a response`. This example uses Teams connector:

     ![Specify Team, message, channel, update card and update message for posting to Teams](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency5-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, message, channel, update card and update message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post approval workflow |
      | **Message** | `message to send` | The message to post to Team channel |
      | **Channel** | `Contingency Plan` | The Teams channel to post approval workflow |
      | **Update card** | `Yes` | Update the adaptive card to show a member of Teams channel has taken an action |
      | **Update message** | `Processing requested action` | Update the adaptive card to show a message once an approval action is taken |
      ||||
      
# Step 9: Add a condition that checks the approval response.

1. On the Logic App Designer, in the Condition box, verify response. This example uses response from earlier Teams connector:

1. Specify the Team card response, expression and verify `approve` response in the condition.
    
      ![Select "Condition to check the response from adaptive card that was posted earlier to Teams channel"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency6-edit.png)
    
      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team Card** | `data.action` | The Team card response to evaluate |
      | **Expression** | `is equal to` | The expression to evaluate |
      | **Condition** | `Approve` | response to verify in the condition |
      ||||
      
# Step 10: Add an action on approval, to enable contingency policies.

1. On the Logic App Designer, in the HTTP connection box, click `Enable contingency policies within conditional access`. This example uses HTTP connector.

1. Specify the Method, URI, Headers, Body, Authentication type, Tenant, Audience, Client ID, Credential Type and Secret.

   ![Select "Enable contingency policies within conditional access" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency7-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Method** | `PATCH` | Method to call |
      | **URI** | `https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/{id}` | Conditional Access API v1.0 endpoint |
      | **Headers** | `application/json` | Content-Type |
      | **Body** | `state: enabled` | Enabling the contingency policies |
      | **Authentication type** | `Active Directory OAuth` | Authentication type for App-only flow |
      | **Tenant** | `TenantID` | Tennat ID configured in step 3 |
      | **Audience** | `https://graph.microsoft.com` | MS Graph |
      | **Client ID** | `Client ID` | Client ID configured in step 3 |
      | **Credential Type** | `Secret` | Client Secret  |
      | **Secret** | `value` | Secret value retrieved from key vault |
      ||||      

# Step 11: Add a check to find if the conditional access policy update was successful. If true, fire an alert on Team channel. 

1. On the Logic App Designer, in the conditions connection box, click `check if the conditional access policy was enabled successfully`. This example uses logic app condition evaluation:

   ![Select "check to find if the conditional access policy was enabled successfully" condition](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-contingency-policies-automation/images/contingency8-edit.png)

1. In the condition, provide the criteria for checking the condition.

1. Specify the HTTP response to evaluate, expression and verify it is equal to `204` response in the condition.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Status code** | `Status code` | The status code from previous step to evaluate |
      | **Expression** | `is equal to` | The expression to evaluate |
      | **Condition** | `204` | response to verify in the condition |
      ||||   
  
1. On the Logic App Designer, in the Teams connection box, click `Post to team channel that contingency policy was enabled successfully`. This example uses Teams connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel and message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post alert |
      | **Channel** | `Contingency plan` | The Teams channel to post alert |
      | **message** | `message` | Post the adaptive card with an alert message |
      ||||

1. On the Logic App Designer, in the Teams connection box, click `Post to team channel that enabling contingency policy failed`. This example uses Teams connector:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. In the connector box, provide the criteria for posting an adaptive card to Teams channel.

1. Specify the Team, channel and message for posting to Teams. The message is shorterned for readability.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Team** | `ConditionalAccess` | The Team to post alert |
      | **Channel** | `Contingency plan` | The Teams channel to post alert |
      | **message** | `message` | Post the adaptive card with an alert message |
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

