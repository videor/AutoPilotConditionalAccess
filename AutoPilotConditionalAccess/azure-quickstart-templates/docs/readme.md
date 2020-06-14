
# Types of permissions

Accessing Microsoft Graph endpoints requires that the application and / or user making the request has the appropriate permissions assigned.  These permissions can be one of two types: delegated permissions or application permissions. Delegated permissions, sometimes called “on behalf of” permissions, require a user context to also be supplied when making the request.  

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

![](https://docs.microsoft.com/en-us/graph/images/auth-v2/new-app-registration-expanded.png)

With the OAuth 2.0 client credentials grant flow, your app authenticates directly at the Microsoft identity platform /token endpoint using the Application ID assigned by Azure AD and the Application Secret that you create using the portal.

# Step 2: Configure permissions for Microsoft Graph

For apps that call Microsoft Graph under their own identity, Microsoft Graph exposes application permissions (Microsoft Graph can also expose delegated permissions for apps that call Microsoft Graph on behalf of a user). You pre-configure the application permissions your app needs when you register your app. Application permissions always require administrator consent. An administrator can either consent to these permissions using the Azure portal when your app is installed in their organization, or you can provide a sign-up experience in your app through which administrators can consent to the permissions you configured. Once administrator consent is recorded by Azure AD, your app can request tokens without having to request consent again. For more detailed information about the permissions available with Microsoft Graph, see the Permissions reference

To configure application permissions for your app in the Azure app registrations portal: under an application's API permissions page, choose Add a permission, select Microsoft Graph, and then choose the permissions your app requires under Application permissions.

The following screenshot shows the Select Permissions dialog box for Microsoft Graph application permissions.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/images/AppRegistration0.PNG)

