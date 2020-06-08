# Azure Resource Manager QuickStart Template to Manage emergency access accounts in Azure AD

It is important that you prevent being accidentally locked out of your Azure Active Directory (Azure AD) organization because you can't sign in or activate another user's account as an administrator. You can mitigate the impact of accidental lack of administrative access by creating two or more emergency access accounts in your organization.

Emergency access accounts are highly privileged, and they are not assigned to specific individuals. Emergency access accounts are limited to emergency or "break glass"' scenarios where normal administrative accounts can't be used. We recommend that you maintain a goal of restricting emergency account use to only the times when it is absolutely necessary.

See the [**Create Emergency Accounts**](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-emergency-access) for how to create, configure and manage emergency access accounts. 

One of the guidance from this [article](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-emergency-access) is to exclude at least one account from Conditional Access policies. So during an emergency, you do not want a policy to potentially block your access to fix an issue. At least one emergency access account should be excluded from all Conditional Access policies. 

This article provides script for managing automated exclusion of emergency access accounts in Azure AD Conditional Access. Please follow the guidance within this [article](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-emergency-access) to create your emergency account.



