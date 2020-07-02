# One-click deploy templates for Conditional Access policies

This is the equivalent of your quick-start templates in your favourite IDE. We have made templates available that you can One-click Deploy to your organization. These includes top used CA policies and sophisticated policies incorporating other products like MCAS. You can deploy these templates with one click to your organization. Fork this repository, and you can create templates specific to your organization’s needs that you can make available to your branch offices and subsidiaries!

# Why do you need these templates

# Template 1 - MFA Registration Restrictions.

Conditional Access Policy using MFA Registration Restrictions feature is one of the top 5 configured policies globally. This template allows you to control the conditions in which security sensitive MFA or SSPR information can be registered. This helps ensure it’s the right user—not an attacker—registering this security sensitive info. 

Some common restrictions you can include are:

1. Users are on a trusted network.
2. Only users with a low sign-in risk can register security information.
3. Users can only register on a managed device.
4. Users should agree to a terms of use during registration.

Make sure that all users you want to apply this policy to are part of the [combined MFA and SSPR experience](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/combined-mfa-and-password-reset-registration-is-now-generally/ba-p/1257355). This is required because users not within scope of combined registration will use the older security information page and the policy will not be enforced. 

This template applies the trusted network restriction to both new registration and users that are already registered but are now updating their registration information.

Now, if a user is outside of a trusted network and attempts to register MFA for first time or re-register MFA, they will be blocked.


# Accidental or malicious removal of emergency accounts from Azure AD Conditional Access Policies during policy update.

And if that doesn’t happen in your org — well, some well-meaning individuals with the appropriate privileges may make a change to a Conditional Access policy — which suddenly causes your org not to have the necessary exclusions for emergency access accounts. 

Okay, let’s say that makes sense, and you agree with me — we all can do with an autopilot to correct the course if mistakes happen. 

# What do you get from this automation

Autopilot provides fully automated exclusions of emergency access accounts when you create new conditional access policies. 

Autopilot will make fully automated adjustments to your policies in case of accidental or malicious deletion of emergency access accounts, so you don’t get locked out!	

![](images/AutoPilotEmergencyAccounts.PNG)




