# Step 1: Deploy the Azure Quickstart Template

# Step 2: Connect User-Assigned Managed Identity to your Logic App

Please look at [**Logic Apps and Managed Identities**](https://docs.microsoft.com/en-us/azure/logic-apps/create-managed-service-identity) to learn more on how to use managed identities within Logic App.

Once you have deployed the automation template from this repository. Connect the user-Assigned Managed Identity to your Logic App by doing the following:

1. In the left-hand navigation pane, select the Logic App service, and then select Identity > User Assigned > Select Add.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-5.PNG)

2. Select the User-assigned managed identity from the context pane that appears on the right, select Add.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-6.PNG)

3. You should see the screen below with your registered user-assigned managed identity connected to your logic app. 

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-7.PNG).

# Step 3: Update parameters value and click Save. That is it!

Once you have connected your user-Assigned Managed Identity to your Logic App add the parameters within your logic app:

1. In the left-hand navigation pane, select the Logic App you deployed, and then select Logic App designer > Parameters > Replace the default value with yours for (a) Key Vault URI (b) Client ID of your app registration (c) The Group Object ID you will like to automate for exclusion and (d) Tenant ID.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-breakglass-automation/images/LogicApp-301-CA-breakglass-automation-8.PNG)

2. Now open the Logic App step for Get Client Secret from Key Vault using Managed Identity and select the Managed Identity you linked to the logic app from the drop down.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-breakglass-automation/images/LogicApp-301-CA-breakglass-automation-9.PNG)

# Success!

You now don't have to worry about manually managing emergency access accounts when you create or update you conditional access policies. The Logic App will wake up and run regularly using app-only permissions completely in Azure Cloud and bring all policies to compliance so you don't get locked out! (Tip: Look at all the Green ticks, they show that the process ran successfully. :-))

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-breakglass-automation/images/LogicApp-301-CA-breakglass-automation-10.PNG)


