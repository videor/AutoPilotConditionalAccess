# Tutorial: Easy configuration using Conditional Access APIs and PowerShell cmdlets

Planning your Conditional Access deployment is critical to achieving your organization's access strategy for apps and resources.

Conditional Access policies are if-then statements: If an assignment is met, then apply these access controls.

When configuring Conditional Access policies, conditions are called assignments. Conditional Access policies allow you to enforce access controls on your organization’s apps based on certain assignments.

For more information, see [Components of CA policy](https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/concept-conditional-access-policies).

You can use the conditional access APIs to manage policies at scale. For example, you can:

- Get a snapshhot of all conditional access policies within your organization and take backups.
- Update a conditional access policy to add new groups as you rollout your policies within your organization.
- Update the list of applications you are targeting within the conditional access policies as new line of business application are added in your organization.
- Delete a policy that is no longer needed.

In this tutorial, you'll learn how to:

:heavy_check_mark: Authenticate with the right roles  <br /> 
:heavy_check_mark: Authenticate with the right permissions  <br /> 
:heavy_check_mark: Create a conditional access policy using an API template  <br /> 
:heavy_check_mark: List all conditional access policies.  <br /> 
:heavy_check_mark: Get a specific conditional access policy.  <br /> 
:heavy_check_mark: Update a conditional access policy  <br /> 
:heavy_check_mark: Delete a conditional access policy  <br /> 

### The contents of this repository are unsupported and may or not be current. Replies to questions about unsupported material have the lowest priority.

# Why unsupported?
Unsupported samples and documentation are provided for our fans and partners for training, and feedback only.


# Step 1: Authenticate with the right roles

If you are using delegated permissions flow, sign in using an account with one of these roles:

For read and write to conditional access policies:

- Global administrator
- [Conditional Access administrator](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-assign-admin-roles#conditional-access-administrator-permissions) ***Recommended***
- Security administrator

For reading conditional access policies:

- [Security Reader permissions](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-assign-admin-roles#security-reader-permissions)
- [Global Reader permissions](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-assign-admin-roles#global-reader-permissions)

# Step 2: Authenticate with the right permissions

One of the following permissions is required to call the read API. To learn more, including how to choose permissions, see [Permissions](/graph/permissions-reference).

|Permission type                        | Permissions (all three roles below are required for read and write operations)                    |
|:--------------------------------------|:---------------------------------------------------------------|
|Delegated (work or school account)     | Policy.Read.All |
|Delegated (personal Microsoft account) | Not supported. |
|Application                            | Policy.Read.All |

All of the below three permissions are required to call the read and write API. To learn more, including how to choose permissions, see [Permissions](/graph/permissions-reference).

|Permission type                        | Permissions (all three roles below are required for read and write operations)                    |
|:--------------------------------------|:---------------------------------------------------------------|
|Delegated (work or school account)     | Policy.Read.All, Policy.ReadWrite.ConditionalAccess and Application.Read.All |
|Delegated (personal Microsoft account) | Not supported. |
|Application                            | Policy.Read.All, Policy.ReadWrite.ConditionalAccess and Application.Read.All |

# Step 3: Connect to Azure AD PowerShell module

Connect to the General Availability release of Azure Active Directory V2 PowerShell Module. A minimum version of [2.0.2.106](https://www.powershellgallery.com/packages/AzureAD/2.0.2.106) is required. 

To install version '2.0.2.106', run Install-Module and add the -Force parameter, this command will install version '2.0.2.106' side-by-side with previous version.

```
Install-Module -Name AzureAD -Force
```
Once completed, connect to Azure AD.

```
Connect-AzureAD
```

To get help with Conditional Access policy cmdlets use the following Get-help command.

```
Get-help AzureADMSConditionalAccessPolicy
```

# Step 3: Create a Conditional Access Policy 

The steps to create a Sign-in risk-based Conditional Access policy [create sign-in risk policy](https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/howto-conditional-access-policy-risk) within Azure Portal is documented. Let us use this document as a reference to create a policy called "CA002: Require MFA for medium + sign-in risk" using the APIs.

To get help with New Conditional Access policy creation cmdlets, use the following *Get-help* command.

```
Get-help New-AzureADMSConditionalAccessPolicy
```

To create a conditional access policy, use the following *New-AzureADMSConditionalAccessPolicy commands*.

```
New-AzureADMSConditionalAccessPolicy -DisplayName "MFA policy" -State "Enabled" -Conditions $conditions -GrantControls $controls 
```

## Create a request

To create the *New-AzureADMSConditionalAccessPolicy* request

## Create the request body

The following common definitions are used to build a request body:

|Name  |Required  |Type  |Description  |
|---------|---------|---------|---------|
|DisplayName     |   true      |   String      |  Policy name       |
|State     |  true       |String         |   Policy state      |
|Conditions     |   true      | [Condition Set](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccessconditionset?view=graph-rest-beta)        |  Represents the type of conditions that govern when the policy applies       |
|GrantControls     |  true       |  [Grant Controls Set](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccessgrantcontrols?view=graph-rest-beta)       |    Represents grant controls that must be fulfilled to pass the policy     |

## Example request body

The following template is used to create a conditional access policy with display name "CA002: Require MFA for medium + sign-in risk".

```
$conditions = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessConditionSet
$conditions.Applications = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessApplicationCondition
$conditions.Applications.IncludeApplications = "All"
$conditions.Users = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessUserCondition
$conditions.Users.IncludeGroups = "6c96716b-b32b-40b8-9009-49748bb6fcd5"
$conditions.Users.ExcludeGroups = "f753047e-de31-4c74-a6fb-c38589047723"
$conditions.SignInRiskLevels = @('high', 'medium')
$controls = New-Object -TypeName Microsoft.Open.MSGraph.Model.ConditionalAccessGrantControls
$controls._Operator = "OR"
$controls.BuiltInControls = "mfa"
```

To create the policy using the above template for conditions and control use the below command 

```
New-AzureADMSConditionalAccessPolicy -DisplayName "CA0002: Require MFA for medium + sign-in risk" -State "enabledForReportingButNotEnforced" -Conditions $conditions -GrantControls $controls
```

## Responses

### Example response

A *successful* response from the previous example request body shows an *id* has been assigned and the *state* is *enabledForReportingButNotEnforced*:

```
Id              : 46a45d1a-2ac8-43d2-b087-78f587f417db
DisplayName     : CA0002: Require MFA for medium + sign-in risk
State           : enabledForReportingButNotEnforced
Conditions      : class ConditionalAccessConditionSet {
                    Applications: class ConditionalAccessApplicationCondition {
                    IncludeApplications: System.Collections.Generic.List`1[System.String]
                    ExcludeApplications: System.Collections.Generic.List`1[System.String]
                    IncludeUserActions: System.Collections.Generic.List`1[System.String]
                    IncludeProtectionLevels:
                  }

                    Users: class ConditionalAccessUserCondition {
                    IncludeUsers: System.Collections.Generic.List`1[System.String]
                    ExcludeUsers: System.Collections.Generic.List`1[System.String]
                    IncludeGroups: System.Collections.Generic.List`1[System.String]
                    ExcludeGroups: System.Collections.Generic.List`1[System.String]
                    IncludeRoles: System.Collections.Generic.List`1[System.String]
                    ExcludeRoles: System.Collections.Generic.List`1[System.String]
                  }

                    Platforms:
                    Locations:
                    SignInRiskLevels: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessRiskLevel]
                    ClientAppTypes: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessClientApp]
                  }

GrantControls   : class ConditionalAccessGrantControls {
                    _Operator: OR
                    BuiltInControls: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessGrantControl]
                    CustomAuthenticationFactors: System.Collections.Generic.List`1[System.String]
                    TermsOfUse: System.Collections.Generic.List`1[System.String]
                  }

SessionControls :
```

# Step 4: List all conditional access policies

To fetch all conditional access policies, use the following *Get-AzureADMSConditionalAccessPolicy* command

```
Get-AzureADMSConditionalAccessPolicy  
```

The GET command has all the required parameters in the command.


#### Responses

##### Example response

Once the 'GET' request is submitted, all conditional access policies (upto the current maximum limit of 194 policies) are returned in the response. 

```
Id              : 46a45d1a-2ac8-43d2-b087-78f587f417db
DisplayName     : CA0002: Require MFA for medium + sign-in risk
State           : enabledForReportingButNotEnforced
Conditions      : class ConditionalAccessConditionSet {
                    Applications: class ConditionalAccessApplicationCondition {
                    IncludeApplications: System.Collections.Generic.List`1[System.String]
                    ExcludeApplications: System.Collections.Generic.List`1[System.String]
                    IncludeUserActions: System.Collections.Generic.List`1[System.String]
                    IncludeProtectionLevels:
                  }

                    Users: class ConditionalAccessUserCondition {
                    IncludeUsers: System.Collections.Generic.List`1[System.String]
                    ExcludeUsers: System.Collections.Generic.List`1[System.String]
                    IncludeGroups: System.Collections.Generic.List`1[System.String]
                    ExcludeGroups: System.Collections.Generic.List`1[System.String]
                    IncludeRoles: System.Collections.Generic.List`1[System.String]
                    ExcludeRoles: System.Collections.Generic.List`1[System.String]
                  }

                    Platforms:
                    Locations:
                    SignInRiskLevels: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessRiskLevel]
                    ClientAppTypes: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessClientApp]
                  }

GrantControls   : class ConditionalAccessGrantControls {
                    _Operator: OR
                    BuiltInControls: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessGrantControl]
                    CustomAuthenticationFactors: System.Collections.Generic.List`1[System.String]
                    TermsOfUse: System.Collections.Generic.List`1[System.String]
                  }

SessionControls :
```

# Step 5: Get a specific conditional access policy

To fetch all conditional access policies, use the following *Get-AzureADMSConditionalAccessPolicy* command

```
Get-AzureADMSConditionalAccessPolicy -PolicyId 46a45d1a-2ac8-43d2-b087-78f587f417db
```

The GET command has all the required parameters in the command.


#### Responses

##### Example response

Once the 'GET' request is submitted, the specified conditional access policy is returned in the response. 

```
Id              : 46a45d1a-2ac8-43d2-b087-78f587f417db
DisplayName     : CA0002: Require MFA for medium + sign-in risk
State           : enabledForReportingButNotEnforced
Conditions      : class ConditionalAccessConditionSet {
                    Applications: class ConditionalAccessApplicationCondition {
                    IncludeApplications: System.Collections.Generic.List`1[System.String]
                    ExcludeApplications: System.Collections.Generic.List`1[System.String]
                    IncludeUserActions: System.Collections.Generic.List`1[System.String]
                    IncludeProtectionLevels:
                  }

                    Users: class ConditionalAccessUserCondition {
                    IncludeUsers: System.Collections.Generic.List`1[System.String]
                    ExcludeUsers: System.Collections.Generic.List`1[System.String]
                    IncludeGroups: System.Collections.Generic.List`1[System.String]
                    ExcludeGroups: System.Collections.Generic.List`1[System.String]
                    IncludeRoles: System.Collections.Generic.List`1[System.String]
                    ExcludeRoles: System.Collections.Generic.List`1[System.String]
                  }

                    Platforms:
                    Locations:
                    SignInRiskLevels: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessRiskLevel]
                    ClientAppTypes: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessClientApp]
                  }

GrantControls   : class ConditionalAccessGrantControls {
                    _Operator: OR
                    BuiltInControls: System.Collections.Generic.List`1[Microsoft.Open.MSGraph.Model.ConditionalAccessGrantControl]
                    CustomAuthenticationFactors: System.Collections.Generic.List`1[System.String]
                    TermsOfUse: System.Collections.Generic.List`1[System.String]
                  }

SessionControls :
```

To view details on a specific object within the response, for example, Users. Use the below example to reference the object.

```
$policy = Get-AzureADMSConditionalAccessPolicy -PolicyId 9de529ed-3051-4643-bb9f-111309fe187d
$policy.Conditions.Users
```

# Step 6: Update a Conditional Access Policy 

To update a conditional access policy, use the following *Set-AzureADMSConditionalAccessPolicy command*.

```
Set-AzureADMSConditionalAccessPolicy -PolicyId 46a45d1a-2ac8-43d2-b087-78f587f417db -DisplayName "CA002: Require MFA for medium + sign-in risk" -State "Enabled" -Conditions $conditions -GrantControls $controls 
```

## Create a request

To create the *Set-AzureADMSConditionalAccessPolicy* request

## Create the request body

The following common definitions are used to build a request body:

|Name  |Required  |Type  |Description  |
|---------|---------|---------|---------|
|DisplayName     |   true      |   String      |  Policy name       |
|State     |  true       |String         |   Policy state      |
|Conditions     |   true      | [Condition Set](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccessconditionset?view=graph-rest-beta)        |  Represents the type of conditions that govern when the policy applies       |
|GrantControls     |  true       |  [Grant Controls Set](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccessgrantcontrols?view=graph-rest-beta)       |    Represents grant controls that must be fulfilled to pass the policy     |

## Example request body

The following command is used to update a conditional access policy by including an additional group.

```
$conditions.Users.IncludeGroups = @('5f6ed05f-c5f4-41d0-96d4-516e2abf825d', 'f5471a71-6fd7-475f-bd09-cda32245e3aa')
```

To update the policy using the above conditions use the below command 

```
Set-AzureADMSConditionalAccessPolicy -PolicyId 46a45d1a-2ac8-43d2-b087-78f587f417db -Conditions $conditions
```

# Step 7: Delete a specific conditional access policy

To delete a specific conditional access policy, use the following *Remove-AzureADMSConditionalAccessPolicy* operation

```
Remove-AzureADMSConditionalAccessPolicy -PolicyId 46a45d1a-2ac8-43d2-b087-78f587f417db
```

The Remove command has all the required parameters in the command.

#### Responses

##### Example response

Once the 'Remove' request is submitted, the specified conditional access policy is deleted. 
