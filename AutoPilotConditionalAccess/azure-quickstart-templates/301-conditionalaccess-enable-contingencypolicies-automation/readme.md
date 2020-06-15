# Azure Resource Manager QuickStart Template to manage Contingency policies in Azure AD

Organizations that rely on a single access control, such as multi-factor authentication (MFA) or a single network location, to secure their IT systems are susceptible to access failures to their apps and resources if that single access control becomes unavailable or misconfigured. For example, a natural disaster can result in the unavailability of large segments of telecommunications infrastructure or corporate networks. Such a disruption could prevent end users and administrators from being able to sign in.

See the [**Create a resilient access control**](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-resilient-controls) for details on how to create a resilient access control management strategy with Azure Active Directory. 

One of the guidance from this [article](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-resilient-controls) is enabling contingency policies in report-only mode when not in use so that administrators can monitor the potential impact of the policies should they need to be turned on. 

This article provides script for automatically enabling contingency policies in Azure AD Conditional Access for a particular disruption and disabling the regular control-based policies. Please follow the guidance within this [article]((https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-resilient-controls) to create and name your contingency policies.

Note: Disabling policies that enforce security on your users, even temporarily, will reduce your security posture while the contingency plan is in place.

# Why do you need this automation

## Organizations can continue to access apps and resources they choose during a disruption by having appropriate contingency plans in place.

Identity is the control plane of users accessing apps and resources. Your identity system controls which users and under which conditions, such as access controls or authentication requirements, users get access to the applications. When one or more authentication or access control requirements aren’t available for users to authenticate due to unforeseen circumstances, organizations can experience one or both of the following issues:

Administrator lockout: Administrators can’t manage the tenant or services.
User lockout: Users can’t access apps or resources.

## Understanding and managing your exposure for a potential disruption helps reduce your risk and is a critical part of your planning process.

A contingency Conditional Access policy is a backup policy that omits Azure MFA, third-party MFA, risk-based or device-based controls. In order to minimize unexpected disruption when a contingency policy is enabled, the policy should remain in report-only mode when not in use. Administrators can monitor the potential impact of their contingency policies using the Conditional Access Insights workbook. When your organization decides to activate your contingency plan, administrators can enable the policy and disable the regular control-based policies.


# What do you get from this automation

Autopilot provides fully automated management of contingency policies so you have one less thing to worry about during disruptions.  

Autopilot will make fully automated adjustments to your policies in case of disruptions, so your users don’t get locked out!	

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-enable-contingencypolicies-automation/images/ResilientPoliciesStatusChange1.PNG)



