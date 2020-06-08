# Azure Resource Manager QuickStart Template to Manage emergency access accounts in Azure AD

It is important that you prevent being accidentally locked out of your Azure Active Directory (Azure AD) organization because you can't sign in or activate another user's account as an administrator. You can mitigate the impact of accidental lack of administrative access by creating two or more emergency access accounts in your organization.

Emergency access accounts are highly privileged, and they are not assigned to specific individuals. Emergency access accounts are limited to emergency or "break glass"' scenarios where normal administrative accounts can't be used. We recommend that you maintain a goal of restricting emergency account use to only the times when it is absolutely necessary.

See the [**Create Emergency Accounts**](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-emergency-access) for how to create, configure and manage emergency access accounts. 

One of the guidance from this [article](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-emergency-access) is to exclude at least one account from Conditional Access policies. So during an emergency, you do not want a policy to potentially block your access to fix an issue. At least one emergency access account should be excluded from all Conditional Access policies. 

This article provides script for managing automated exclusion of emergency access accounts in Azure AD Conditional Access. Please follow the guidance within this [article](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-emergency-access) to create your emergency account.

# Why do you need this automation

# New Azure AD Conditional Access Policies deployed in production may not have approprate exclusions applied for emergency accounts.

Conditional Access Policies rolling out through an organization can have benefits and risks similar to code rolling out through the various rings of a service. A new policy could behave just right in your test environment with correct exclusion being applied to emergency access accounts. But how would it behave when deployed to your production environment and missed the exclusion? At the first touch of the pragmatic realities of your organization, it may demonstrate to you all the corner cases that did not occur to you to test!

# Accidental or malicious removal of emergency accounts from Azure AD Conditional Access Policies during policy update.

And if that doesn’t happen in your org — well, some well-meaning individuals with the appropriate privileges may make a change to a Conditional Access policy — which suddenly causes your org not to have the necessary exclusions for emergency access accounts. 

Okay, let’s say that makes sense, and you agree with me — we all can do with an autopilot to correct the course if mistakes happen. 




