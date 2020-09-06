# Tutorial:	Backup and Restore Conditional Access policies with approval workflow

Intent: As an IT admin, I want to be able to easily backup-restore conditional access policies.

You can use the conditional access APIs to easily backup conditional access policies. For example, you can:

As a IT admin, be able to automate a regular backup/snapshot of all conditional access policies within your environment. Select a policy to Restore. Finally, restore the conditional access policy in your production environment.

![Manage](/media/Configure5.PNG)
<br /> 

This automation can be very useful for: 
- Organizations that manages large numbers of conditional access policies. OR
- Identity partners that manages policies for customers. 

This tutorial shows how to build a [logic app](https://docs.microsoft.com/en-us/azure/logic-apps/) that allows backup-restore of conditional access policies. Specifically, this logic app takes a snapshot of all conditional access policies as a scheduled task within Onedrive. Another logic app monitors any policies being queued in the Restore folder of Ondrive. If a new policy is detected, an approval workflow is triggered on Team channel. On approval, the policy is restored to production environment.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add parameters and connections specific to your organization within logic app.  <br /> 

When you're done, you will be able to backup-restore Conditional Access policies in your production environment:
<br /> 
<br /> 

## 1. Select the policy you want to restore from Backup and copy to Restore Onedrive folder. 

![Copy to OneDrive](/media/BackupRestore-Step1.PNG)
<br /> 
<br /> 

## 2. Approve Restore action in Teams

![Approve configuration](/media/BackupRestore-Step2.PNG)
<br /> 
<br /> 
## 3. Receive notification that Restore action is successfully completed in your production environment

![Confirmation](/media/BackupRestore-Step3.PNG)
<br /> 
<br /> 
## 4. View your restored Conditional Access policy in Azure portal

![View policy](/media/BackupRestore-Step4.PNG)
<br /> 
<br /> 
## 5. Check your user assignments are correct within the policy

![Assign test users](/media/BackupRestore-Step5.PNG)
<br /> 
<br /> 



# Pre-requisites

If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you start.

You will also need knowledge of key concepts within Azure logic apps, Onedrive and Teams. 

### The contents of this repository are unsupported and may or not be current. Replies to questions about unsupported material have the lowest priority.

# Why unsupported?
Unsupported samples and documentation are provided for our fans and partners for training, and feedback only.

# Step 1: Deploy this logic app to your organization

If your Azure environment meets the prerequisites, and you're familiar with using ARM templates, these steps help you sign in directly to Azure and open the ARM template in the Azure portal. For more information, see [Deploy resources with ARM templates and Azure portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview). 

<a name="deploy-azure-portal"></a>

#### [Azure Portal](#tab/azure-portal)

1. Select the following image to sign in with your Azure account and open the logic app in the Azure portal:

:heavy_check_mark: Logic App for Backup

 [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-policy-backup-restore-automation%2Fazuredeploy.json)
 
   [Video Link that takes you through the deployment process for Backup of conditional access policies](https://www.screencast.com/t/c8VXNe7FhANS)
   
  :heavy_check_mark: Logic App for Restore
  
   [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-policy-backup-restore-automation%2F301-conditionalaccess-policy-restore-automation%2Fazuredeploy.json)

  [Video Link that takes you through the deployment process for Restore of conditional access policies](https://www.screencast.com/t/fmUlaf3mW)
  
1. In the portal, on the **Custom deployment** page, enter or select these values:

   | Property | Value | Description |
   |----------|-------|-------------|
   | **Subscription** | <*Azure-subscription-name*> | The name for the Azure subscription to use |
   | **Resource group** | <*Azure-resource-group-name*> | The name for a new or existing Azure resource group. This example uses `AutoPilotConditionalAccess`. |
   | **Location** |  <*Azure-region-for-all-resources*> | The Azure region to use for all resources, if different from the default value. This example uses the default value, `[resourceGroup().location]`, which is the resource group location. |
   | **Logic App Name** | <*logic-app-name*> | The name to use for your logic app. This example uses `301-conditionalaccess-policy-backup-automation` and `301-conditionalaccess-policy-restore-automation`. |
   ||||

 
Here is how the page looks with the values used in this example:

![Provide information for quickstart template](/media/Deploy.png)

1. When you're done, select **Review + Create** and finally **Create**.


# Step 2: Authenticate your logic app to Azure AD with the right permissions

This logic app uses managed identity for getting secrets from key vault in order to call conditional access APIs. Please look at [Authenticate your logic app to Azure AD with the right permissions](https://github.com/videor/AutoPilotConditionalAccess/tree/master/AutoPilotConditionalAccess/azure-quickstart-templates/docs) on how to create key vault and connect to managed identity. To learn more on how to use managed identities within Logic App please look at [**Logic Apps and Managed Identities**](https://docs.microsoft.com/en-us/azure/logic-apps/create-managed-service-identity) .

1. In the left-hand navigation pane, select Identity > User Assigned > Select Add.

2. Select the User-assigned managed identity from the context pane that appears on the right, select Add.

![ManagedIdentity](/media/MI-edit.png)

# Step 3: Update parameters

1. In the left-hand navigation pane, select Logic App designer > Parameters > Replace the default value with Key Vault URI (storing Client Secret), Client ID and Tenant ID.

![Parameters](/media/LA-parameters-edit.png)


# Step 4: Select appropriate managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Managed Identity to use.

![Select "Managed Identity"](/media/MInew.PNG)


# Step 5: Connect to your OneDrive account and select the Backup and Restore folders in respective logic apps you will like to use for automation.

1. On the Logic App Designer, in the OneDrive for Business connection box, click `Connections`. This example uses OneDrive connector for Logic apps:

![Select "Connections"](/media/OneDrivenew.PNG)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your OneDrive account.

1. If connection is successful, select the OneDrive folder you would like to use for automation.


# Step 6: Connect to Teams channel for approving or rejecting Restore requests.

1. On the Logic App Designer, in the Teams connection box, click `Connections`. This example uses Teams connector:

![Select "Connections"](/media/Teamsnew.PNG)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. Specify the Team and channel you will like to use for automation of approval workflow.



# Step 7: Update all other connectors within Logic App.

Similar to above, update remaining OneDrive and Teams connectors within the sample Logic App by selecting appropriate OneDrive and Teams account that needs to be used for automation.

# Note

Please ensure you follow the best practise guidelines on managing secrets within Logic apps by using secure inputs and outputs as [documented here](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-securing-a-logic-app).

# Forward Looking

Try the following challenge:

:heavy_check_mark: Edit this logic app to send a custom message on Teams channel when the approval workflow selection is Reject restore action. <br /> 

If you would like to request a logic app to do this, please send a request on Twitter @Vi_Deora.




