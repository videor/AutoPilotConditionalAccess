# Tutorial: Easy configuration using Conditional Access API Templates

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

To fetch all conditional access policies, use the following *GET* operation

```http
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
```

The GET URI has `{v1.0}`, `{identity}`, `{conditionalAccess}` and `{policies}` within the endpoint. As all the required parameters are given in the URI, there is no need for a separate request body.


#### Responses

The successful response for the 'GET' operation is shown below:

|Name  | Description  |
|---------|---------|
|200 OK    | OK        |

##### Example response

Once the 'GET' request is submitted, a 200 (successful) response is returned. All conditional access policies (upto the current maximum limit of 194 policies) are returned in the response. 

```json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#identity/conditionalAccess/policies",
    "value": [
     {
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#identity/conditionalAccess/policies/$entity",
    "id": "34ab2fe6-51ec-4442-9558-723f480ee29f",
    "displayName": "CA002: Require MFA for medium + sign-in risk",
    "createdDateTime": "2020-07-08T11:34:24.6674365Z",
    "modifiedDateTime": null,
    "state": "enabledForReportingButNotEnforced",
    "sessionControls": null,
    "conditions": {
        "signInRiskLevels": [
            "high",
            "medium"
        ],
        "clientAppTypes": [],
        "platforms": null,
        "locations": null,
        "devices": null,
        "applications": {
            "includeApplications": [
                "All"
            ],
            "excludeApplications": [],
            "includeUserActions": []
        },
        "users": {
            "includeUsers": [],
            "excludeUsers": [],
            "includeGroups": [
                "6c96716b-b32b-40b8-9009-49748bb6fcd5"
            ],
            "excludeGroups": [
                "f753047e-de31-4c74-a6fb-c38589047723"
            ],
            "includeRoles": [],
            "excludeRoles": []
        }
    },
    "grantControls": {
        "operator": "OR",
        "builtInControls": [
            "mfa"
        ],
        "customAuthenticationFactors": [],
        "termsOfUse": []
       }
    }
  ]
}
```

# Step 5: Get a specific conditional access policy

To get a specific conditional access policy, use the following *GET* operation

```http
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/{policyid}
```

The GET URI has `{policyid}` parameter. In this example, `{policyid}` is "34ab2fe6-51ec-4442-9558-723f480ee29f".  As all the required parameters are given in the URI, there is no need for a separate request body.

```http
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/34ab2fe6-51ec-4442-9558-723f480ee29f
```

#### Responses

The successful response for the 'GET' operation is shown below:

|Name  | Description  |
|---------|---------|
|200 OK    | OK        |

##### Example response

Once the 'GET' request is submitted, a 200 (successful) response is returned. 

```json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#identity/conditionalAccess/policies/$entity",
    "id": "34ab2fe6-51ec-4442-9558-723f480ee29f",
    "displayName": "CA002: Require MFA for medium + sign-in risk",
    "createdDateTime": "2020-07-08T11:34:24.6674365Z",
    "modifiedDateTime": null,
    "state": "enabledForReportingButNotEnforced",
    "sessionControls": null,
    "conditions": {
        "signInRiskLevels": [
            "high",
            "medium"
        ],
        "clientAppTypes": [],
        "platforms": null,
        "locations": null,
        "devices": null,
        "applications": {
            "includeApplications": [
                "All"
            ],
            "excludeApplications": [],
            "includeUserActions": []
        },
        "users": {
            "includeUsers": [],
            "excludeUsers": [],
            "includeGroups": [
                "6c96716b-b32b-40b8-9009-49748bb6fcd5"
            ],
            "excludeGroups": [
                "f753047e-de31-4c74-a6fb-c38589047723"
            ],
            "includeRoles": [],
            "excludeRoles": []
        }
    },
    "grantControls": {
        "operator": "OR",
        "builtInControls": [
            "mfa"
        ],
        "customAuthenticationFactors": [],
        "termsOfUse": []
    }
}
```

# Step 6: Update a Conditional Access Policy 

Let us now update the conditional access policy we retrieved in the previous step.

To update a conditional access policy, use the following *PATCH* operation.

```http
PATCH https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/{policyid}
```
The PATCH URI has `{policyid}` parameter. In this example, `{policyid}` is "34ab2fe6-51ec-4442-9558-723f480ee29f".  

```http
PATCH https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/34ab2fe6-51ec-4442-9558-723f480ee29f
```

## Create a request

To create the *PATCH* request

The following headers are required:

| Request header   | Description |
|------------------|-----------------|
| *Content-Type:*  | Required. Set to `application/json`. |
| *Authorization:* | Required. Set to a valid `Bearer` [access token](https://docs.microsoft.com/rest/api/azure/#authorization-code-grant-interactive-clients). |

For more information about how to create the request, see [Components of API request/response](https://docs.microsoft.com/en-us/rest/api/azure/#components-of-a-rest-api-requestresponse).

## Create the request body

The following common definitions are used to build a request body:

|Name  |Required  |Type  |Description  |
|---------|---------|---------|---------|
|displayName     |   true      |   String      |  Policy name       |
|state     |  true       |String         |   Policy state      |
|conditions     |   true      | [Condition Set](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccessconditionset?view=graph-rest-beta)        |  Represents the type of conditions that govern when the policy applies       |
|grantControls     |  true       |  [Grant Controls Set](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccessgrantcontrols?view=graph-rest-beta)       |    Represents grant controls that must be fulfilled to pass the policy     |

## Example request body

The following example body is used to update the conditional access policy with an additional group id {"ba8e7ded-8b0f-4836-ba06-8ff1ecc5c8ba"}.

```json
{ 
    "conditions": { 
        "users": { 
            "includeGroups": [ 
                "6c96716b-b32b-40b8-9009-49748bb6fcd5", "ba8e7ded-8b0f-4836-ba06-8ff1ecc5c8ba"
            ]
        } 
    }
} 
```

## Responses

A successful response for the operation to create a conditional access policy:

|Name  |Description  |
|---------|---------|
|204 No Content     |   Updated      |

For more information about REST API responses, see [Process the response message](https://docs.microsoft.com/en-us/rest/api/azure/#process-the-response-message).

### Example response

A *204 No Content* response from the previous example request body shows the *groupid* has been added to *includeGroups* property:

```json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#identity/conditionalAccess/policies/$entity",
    "id": "34ab2fe6-51ec-4442-9558-723f480ee29f",
    "displayName": "CA002: Require MFA for medium + sign-in risk",
    "createdDateTime": "2020-07-08T11:34:24.6674365Z",
    "modifiedDateTime": "2020-07-08T12:22:42.3500388Z",
    "state": "enabledForReportingButNotEnforced",
    "sessionControls": null,
    "conditions": {
        "signInRiskLevels": [
            "high",
            "medium"
        ],
        "clientAppTypes": [
            "browser",
            "mobileAppsAndDesktopClients"
        ],
        "platforms": null,
        "locations": null,
        "devices": null,
        "applications": {
            "includeApplications": [
                "All"
            ],
            "excludeApplications": [],
            "includeUserActions": []
        },
        "users": {
            "includeUsers": [],
            "excludeUsers": [],
            "includeGroups": [
                "6c96716b-b32b-40b8-9009-49748bb6fcd5",
                "ba8e7ded-8b0f-4836-ba06-8ff1ecc5c8ba"
            ],
            "excludeGroups": [
                "f753047e-de31-4c74-a6fb-c38589047723"
            ],
            "includeRoles": [],
            "excludeRoles": []
        }
    },
    "grantControls": {
        "operator": "OR",
        "builtInControls": [
            "mfa"
        ],
        "customAuthenticationFactors": [],
        "termsOfUse": []
    }
}
```

# Step 7: Delete a specific conditional access policy

To delete a specific conditional access policy, use the following *DELETE* operation

```http
DELETE https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/{policyid}
```

The DELETE URI has `{policyid}` parameter. In this example, `{policyid}` is "34ab2fe6-51ec-4442-9558-723f480ee29f".  As all the required parameters are given in the URI, there is no need for a separate request body.

```http
DELETE https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/34ab2fe6-51ec-4442-9558-723f480ee29f
```

#### Responses

The successful response for the 'DELETE' operation is shown below:

|Name  | Description  |
|---------|---------|
|204 No Content    | Deleted        |

##### Example response

Once the 'DELETE' request is submitted, a 204 (successful) response is returned. 


## Next steps

Try practising the above steps with some other quick-start api templates here:

- [Template 1](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/101-conditionalaccess-apis-tutorial/Quickstart%20api%20template%201%20-%20MFA%20registration%20restrictions.json)
- [Template 2](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/101-conditionalaccess-apis-tutorial/Quickstart%20api%20template%202%20-%20Require%20mfa%20for%20privileged%20accounts.json)
- [Template 3](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/101-conditionalaccess-apis-tutorial/Quickstart%20api%20template%203%20-%20Require%20mfa%20guest%20access.json)
- [Template 4](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/101-conditionalaccess-apis-tutorial/Quickstart%20api%20template%204%20-Require%20mam%20policies%20for%20android%20and%20ios.json)
- [Template 5](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/101-conditionalaccess-apis-tutorial/Quickstart%20api%20template%205%20-%20Require%20trusted%20device%20for%20app%20access.json)

Next, [try creating a backup of your conditional access policies in Onedrive](https://github.com/videor/AutoPilotConditionalAccess/tree/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation).

For more information on the Conditional Access APIs, see the following documents:

- [Conditional Access API](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta)
- [Named Location API](https://docs.microsoft.com/en-us/graph/api/resources/namedlocation?view=graph-rest-beta)


