
# Types of permissions

Accessing Microsoft Graph endpoints requires that the application and / or user making the request has the appropriate permissions assigned.  These permissions can be one of two types: delegated permissions or application permissions. Delegated permissions, sometimes called “on behalf of” permissions, require a user context to also be supplied when making the request.  

To learn more on permissions, app registration and using app-only token please look at [**Create App Registration**]( https://docs.microsoft.com/en-us/graph/auth-register-app-v2) and [**Get access without a user/app-only**](https://docs.microsoft.com/en-us/graph/auth-v2-service)

# Application (app-only or “without a user”)

Some apps call Microsoft Graph with their own identity and not on behalf of a user. In many cases, these are background services or daemons that run on a server without the presence of a signed-in user. An example of such an app might be an email archival service that wakes up and runs overnight. In some cases, apps that have a signed-in user present may also need to call Microsoft Graph under their own identity. For example, an app may need to use functionality that requires more elevated privileges in an organization than those carried by the signed-in user.

Apps that call Microsoft Graph with their own identity use the OAuth 2.0 client credentials grant flow to get access tokens from Azure AD. This article describes the basic steps to configure a service and use the OAuth client credentials grant flow to get an access token.

When we call Conditional Access APIs for break glass account management or other workflows to be executed as background services we would be using app-only permissions..


# Step 1: Register an application with the Microsoft identity platform


This page shows you how to add and register an application using the App registrations experience in the Azure portal so that your app can be integrated with the Microsoft identity platform and call Microsoft Graph.

Register a new application using the Azure portal

1. Sign in to the Azure portal using either a work or school account or a personal Microsoft account.

2. If your account gives you access to more than one tenant, select your account in the top right corner, and set your portal session to the Azure AD tenant that you want.

3. In the left-hand navigation pane, select the Azure Active Directory service, and then select App registrations > New registration.

4. When the Register an application page appears, enter your application's registration information. (a) Name - Enter a meaningful application name that will be displayed to users of the app. (b) Supported account types - Select Accounts in this organizational directory only.	

5. When finished, select Register.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/AppRegistration1.PNG)

With the OAuth 2.0 client credentials grant flow, your app authenticates directly at the Microsoft identity platform /token endpoint using the Application ID assigned by Azure AD and the Application Secret that you create using the portal.

# Step 2: Configure permissions for Microsoft Graph and secret

For apps that call Microsoft Graph under their own identity, Microsoft Graph exposes application permissions (Microsoft Graph can also expose delegated permissions for apps that call Microsoft Graph on behalf of a user). You pre-configure the application permissions your app needs when you register your app. Application permissions always require administrator consent. An administrator can either consent to these permissions using the Azure portal when your app is installed in their organization, or you can provide a sign-up experience in your app through which administrators can consent to the permissions you configured. Once administrator consent is recorded by Azure AD, your app can request tokens without having to request consent again. For more detailed information about the permissions available with Microsoft Graph, see the Permissions reference

To configure application permissions for your app in the Azure app registrations portal: under an application's API permissions page, choose Add a permission, select Microsoft Graph, and then choose the permissions your app requires under Application permissions.

The following screenshot shows the Application Permissions required for Microsoft Graph Conditional Access Automation.

Generate The Application Secret for your app in the app registration portal. 

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/AppRegistration7.PNG)

# Step 3: Deploy the Azure Quickstart Template


# Step 4: Create a Managed Identity for your Automation

Managed identities for Azure resources is a feature of Azure Active Directory. Please look at [**managed Identity**]( https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) for more information.

As a identity admin, you are probably looking for the simplest and most secure method to access Conditional Access APIs in your automation.

The managed identities for Azure can help you with this requirement because managed identities:

1. Eliminates the need for credentials.
2. Rotate credentials automatically.
3. Reduces your involvement in managing identities to a minimum.

All Azure resources that support managed identities can obtain tokens to exchange data without having credentials in the code. The process consists of the following steps:

Enable – Create the Managed Identity for the resource.
Grant access – Allow access to resources with Azure RBAC.
Access - Perform the allowed actions.
Disable – Delete the Managed Identity.

Managed identity types

There are two types of managed identities: System-assigned managed identity and User-assigned managed identity

For stand-alone automations, you can enable system-assigned managed identities. System-assigned managed identities provide the most convenient support from the identity management perspective. With just one click, you can enable the automated life cycle management of an identity for your resource.

Once you have deployed the automation template from this repository. 

1. In the left-hand navigation pane, select the Logic App service, and then select Identity > System Assigned > Switch status to "On".

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-2.PNG)

2. When finished, select Yes.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-3.PNG)

3. You should see the screen below with your registered managed identity. 

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/LogicApp-301-CA-breakglass-automation-4.PNG)




