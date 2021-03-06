# Tutorial:	Business Continuity with Contingency Policies and Conditional Access

Intent: As an IT admin, I want to be able to easily fail-over to contingency policies during service issues.

You can use the conditional access APIs to manage failover to contingency policies. For example, you can:

As a IT admin, trigger fail-over and post an approval workflow to a Team channel. Members of contingency plan can approve or reject the fail-over. If approved, the contingency policy is enabled and relevant operational policies are disabled.


![Manage](/media/Configure5.PNG)
<br /> 

This automation can be very useful for: 
- Organizations that manages large numbers of conditional access policies. OR
- Identity partners that manages policies for customers. 

This tutorial shows how to build a [logic app](https://docs.microsoft.com/en-us/azure/logic-apps/) that automates fail-over to contingency policies. Specifically, this logic app is triggired with a specific disruption tag. This sends an alert on Teams channel with list of contingency policies specific to that disruption. Admins, can individually select and approve fail-over.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add parameters and connections specific to your organization within logic app.  <br /> 

When you're done, you will be able to automate management of contingency policies.
<br /> 
<br /> 


## 1. Ensure you have contingency policies configured for a given distruption and relevant operational policies are tagged for same disruption. Below example shows contingency policies configured for MFA Disruption.

![Contingency](/media/Contingency-Step1.PNG)
<br /> 
<br />  

## 3. When the logic app is triggered, receive approval request to **enable contingency policies** in Teams.

![Check](/media/Contingency-Step2.PNG)
<br /> 
<br /> 

## 4. Receive success notification in Teams

![Check](/media/Contingency-Step3.PNG)
<br /> 
<br /> 

## 5. Receive approval request to **disable tagged operational policies** in Teams

![Check](/media/Contingency-Step4.PNG)
<br /> 
<br /> 

## 6. Check contingency policies are now enabled in Azure Portal

![Check](/media/Contingency-Step5.PNG)
<br /> 
<br /> 


# Pre-requisites

If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you start.

### The contents of this repository are unsupported and may or not be current. Replies to questions about unsupported material have the lowest priority.

# Why unsupported?
Unsupported samples and documentation are provided for our fans and partners for training, and feedback only.

# Step 1: Deploy this logic app to your organization

If your Azure environment meets the prerequisites, and you're familiar with using ARM templates, these steps help you sign in directly to Azure and open the ARM template in the Azure portal. For more information, see [Deploy resources with ARM templates and Azure portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview). 

<a name="deploy-azure-portal"></a>

Select the following image to sign in with your Azure account and open the logic app in the Azure portal:

   [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-contingency-policies-automation%2Fazuredeploy.json)

1. In the portal, on the **Custom deployment** page, enter or select these values:

   | Property | Value | Description |
   |----------|-------|-------------|
   | **Subscription** | <*Azure-subscription-name*> | The name for the Azure subscription to use |
   | **Resource group** | <*Azure-resource-group-name*> | The name for a new or existing Azure resource group. This example uses `AutoPilotConditionalAccess`. |
   | **Location** |  <*Azure-region-for-all-resources*> | The Azure region to use for all resources, if different from the default value. This example uses the default value, `[resourceGroup().location]`, which is the resource group location. |
   | **Logic App Name** | <*logic-app-name*> | The name to use for your logic app. This example uses `301-conditionalaccess-contingency-policies-automation`. |
   ||||

 Here is how the page looks with the values used in this example:

![Provide information for quickstart template](/media/Deploy.png)

2. When you're done, select **Review + Create** and finally **Create**.


# Step 2: Authenticate your logic app to Azure AD with the right permissions

This logic app uses managed identity for getting secrets from key vault in order to call conditional access APIs. Please look at [Authenticate your logic app to Azure AD with the right permissions](https://github.com/videor/AutoPilotConditionalAccess/tree/master/AutoPilotConditionalAccess/azure-quickstart-templates/docs) on how to create key vault and connect to managed identity. To learn more on how to use managed identities within Logic App please look at [**Logic Apps and Managed Identities**](https://docs.microsoft.com/en-us/azure/logic-apps/create-managed-service-identity) .

1. In the left-hand navigation pane, select Identity > User Assigned > Select Add.

2. Select the User-assigned managed identity from the context pane that appears on the right, select Add.

![ManagedIdentity](/media/MI-edit.png)

# Step 3: Update parameters

1. In the left-hand navigation pane, select Logic App designer > Parameters > Replace the default value with Key Vault URI (that stores client secret), Client ID, Enable Tag (based on your naming convention) and Tenant ID.

![Parameters](/media/ContingencyPara.PNG)


# Step 4: Select appropriate managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Managed Identity to use.

![Select "Managed Identity"](/media/MInew.PNG)


# Step 5: Connect to Teams channel for notification.

1. On the Logic App Designer, in the Teams connection box, click `Connections`. This example uses Teams connector:

![Select "Connections"](/media/Teamsnew.PNG)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Teams account.

1. Specify the Team and channel you will like to use for automation.


# Step 6: Update all other connectors within Logic App.

Similar to above, update remaining Teams connectors within the sample Logic App by selecting appropriate Teams account that needs to be used for automation.

# Note

Please ensure you follow the best practise guidelines on managing secrets within Logic apps by using secure inputs and outputs as [documented here](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-securing-a-logic-app).

# Forward Looking

Try the following challenge:

:heavy_check_mark: Edit this logic app to trigger staggered alerts based on sequence number within policy display name. This sequence represents the order in which you must activate the policies. <br /> 

If you would like to request a logic app to do this, please send a request on Twitter @Vi_Deora.
