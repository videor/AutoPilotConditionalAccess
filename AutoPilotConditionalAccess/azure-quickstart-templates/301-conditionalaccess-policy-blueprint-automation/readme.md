# Tutorial: One-click deploy policy blueprints using Conditional Access APIs

Intent: As an IT admin, I want to be able to easily deploy conditional access policies to large number of branch offices and subsidiaries.

You can use the conditional access APIs to manage policy blueprints that you can make available to your branch offices and subsidiaries. For example, you can:

As a central IT admin, add a new policy blueprint using conditional access schema to a shared onedrive. All subsidieries and branch offices who have been given access to this onedrive will receive notification to create a new conditional access policy based on the policy blueprint. Admins in branch offices and subsidiaries can approve or reject this blueprint. If approved, the policy blueprint will be deployed to consented branch offices and subsidiaries! 

This automation can be very useful for organizations that manage large numbers of Azure AD tenants representing branch offices and subsidiaries. Identity partners that need to provide policy blueprints to a number of their customers regularly. Or even in cases of mergers and acquisitions. 

This tutorial shows how to build a [logic app](../logic-apps/logic-apps-overview.md) that automates policy blueprint deployment to branch offices and subsidiaries based on an approval-based workflow. Specifically, this logic app monitors a shared onedrive account for new or updated policy blueprints submissions, sends requests for approval, and deploys the policy blueprint to approved branch offices and subsidiaries.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add a trigger that monitors OneDrive folder for policy blueprint requests.<br /> 
:heavy_check_mark: Add an action that sends a message to Teams security channel for approving or rejecting these requests.<br /> 
:heavy_check_mark: Add a condition that checks the approval response.<br /> 
:heavy_check_mark: Add an action that deploys the policy blueprint request.<br /> 
:heavy_check_mark: Add a condition that checks whether the policy blueprint was deployed successfully.<br /> 
:heavy_check_mark: Add an action that sends email confirming deployment of policy blueprint.<br /> 


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

   ![Provide information for quickstart template](./media/quickstart-create-deploy-azure-resource-manager-template/create-logic-app-template-portal.png)

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




