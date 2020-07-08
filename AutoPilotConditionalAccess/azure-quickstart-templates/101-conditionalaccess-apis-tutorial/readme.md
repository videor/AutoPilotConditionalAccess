# Tutorial: How to use Conditional Access APIs 101

:heavy_check_mark: List all conditional access policies.  <br /> 
:heavy_check_mark: Get a specific conditional access policy.  <br /> 
:heavy_check_mark: Create a conditional access policy  <br /> 
:heavy_check_mark: Update a conditional access policy  <br /> 
:heavy_check_mark: Remediate errors within conditional access policy  <br /> 
:heavy_check_mark: Delete conditional access policy  <br /> 

# List all conditional access policies

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
            "id": "2a945ba4-2b5f-4b1c-8f05-db014df71b83",
            "displayName": "CA001 - Enforce MAM Policy for Android and IOS",
            "createdDateTime": "2020-06-28T09:37:53.6366773Z",
            "modifiedDateTime": null,
            "state": "enabledForReportingButNotEnforced",
            "sessionControls": null,
            "conditions": {
                "signInRiskLevels": [],
                "clientAppTypes": [
                    "mobileAppsAndDesktopClients"
                ],
                "locations": null,
                "devices": null,
                "applications": {
                    "includeApplications": [
                        "00000002-0000-0ff1-ce00-000000000000",
                        "00000003-0000-0ff1-ce00-000000000000"
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
                },
                "platforms": {
                    "includePlatforms": [
                        "android",
                        "iOS"
                    ],
                    "excludePlatforms": []
                }
            },
            "grantControls": {
                "operator": "OR",
                "builtInControls": [
                    "approvedApplication",
                    "compliantApplication"
                ],
                "customAuthenticationFactors": [],
                "termsOfUse": []
            }
        }
   ]
}
```

# Get a specific conditional access policy

To fetch all conditional access policies, use the following *GET* operation

```http
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/{policyid}
```

The GET URI has `{policyid}` parameter. In this example, `{policyid}` is "2a945ba4-2b5f-4b1c-8f05-db014df71b83".  As all the required parameters are given in the URI, there is no need for a separate request body.

```http
GET https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies/2a945ba4-2b5f-4b1c-8f05-db014df71b83
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
            "id": "2a945ba4-2b5f-4b1c-8f05-db014df71b83",
            "displayName": "CA001 - Enforce MAM Policy for Android and IOS",
            "createdDateTime": "2020-06-28T09:37:53.6366773Z",
            "modifiedDateTime": null,
            "state": "enabledForReportingButNotEnforced",
            "sessionControls": null,
            "conditions": {
                "signInRiskLevels": [],
                "clientAppTypes": [
                    "mobileAppsAndDesktopClients"
                ],
                "locations": null,
                "devices": null,
                "applications": {
                    "includeApplications": [
                        "00000002-0000-0ff1-ce00-000000000000",
                        "00000003-0000-0ff1-ce00-000000000000"
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
                },
                "platforms": {
                    "includePlatforms": [
                        "android",
                        "iOS"
                    ],
                    "excludePlatforms": []
                }
            },
            "grantControls": {
                "operator": "OR",
                "builtInControls": [
                    "approvedApplication",
                    "compliantApplication"
                ],
                "customAuthenticationFactors": [],
                "termsOfUse": []
            }
}
```

# Create Conditional Access Policies 

The steps to create a Sign-in risk-based Conditional Access policy [create sign-in risk policy](https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/howto-conditional-access-policy-risk) within Azure Portal is documented. Let us use this document as a reference to create a policy called "CA002: Require MFA for medium + sign-in risk" using the APIs.

To create a conditional access policy, use the following *POST* operation.

```http
POST https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
```

## Create a request

To create the *POST* request

The following headers are required:

| Request header   | Description |
|------------------|-----------------|
| *Content-Type:*  | Required. Set to `application/json`. |
| *Authorization:* | Required. Set to a valid `Bearer` [access token](https://docs.microsoft.com/rest/api/azure/#authorization-code-grant-interactive-clients). |

For more information about how to create the request, see [Components of a REST API request/response](/rest/api/azure/#components-of-a-rest-api-requestresponse).

## Create the request body

The following common definitions are used to build a request body:

|Name  |Required  |Type  |Description  |
|---------|---------|---------|---------|
|displayName     |   true      |   String      |  Policy name       |
|state     |  true       |String         |   Policy state      |
|conditions     |   true      | [Condition Set](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccessconditionset?view=graph-rest-beta)        |  Represents the type of conditions that govern when the policy applies       |
|grantControls     |  true       |  [Grant Controls Set](https://docs.microsoft.com/en-us/graph/api/resources/conditionalaccessgrantcontrols?view=graph-rest-beta)       |    Represents grant controls that must be fulfilled to pass the policy     |

Note that vault name and resource group name are provided in the PUT URI. The request body defines the location.

## Example request body

The following example body is used to create a vault in "West US". Specify the location. The SKU is always "Standard".

```json
{ 
    "displayName": "ZT1: Require MFA for medium + sign-in risk", 
    "state": "enabledForReportingButNotEnforced", 
    "conditions": { 
        "signInRiskLevels": [ "high" ,
            "medium" 
        ], 
        "applications": { 
            "includeApplications": [ 
                "All" 
            ] 
        }, 
        "users": { 
            "includeGroups": [ 
                "6c96716b-b32b-40b8-9009-49748bb6fcd5" 
            ], 
            "excludeGroups": [ 
                "f753047e-de31-4c74-a6fb-c38589047723" 
            ] 
        } 
    }, 
    "grantControls": { 
        "operator": "OR", 
        "builtInControls": [ 
            "mfa" 
        ] 
    } 
} 
```

## Responses

There are two successful responses for the operation to create or update a Recovery Services vault:

|Name  |Type  |Description  |
|---------|---------|---------|
|200 OK     |   [Vault](https://docs.microsoft.com/rest/api/recoveryservices/vaults/createorupdate#vault)      | OK        |
|201 Created     | [Vault](https://docs.microsoft.com/rest/api/recoveryservices/vaults/createorupdate#vault)        |   Created      |

For more information about REST API responses, see [Process the response message](/rest/api/azure/#process-the-response-message).

### Example response

A condensed *201 Created* response from the previous example request body shows an *id* has been assigned and the *provisioningState* is *Succeeded*:

```json
{
  "location": "westus",
  "name": "testVault",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "id": "/subscriptions/77777777-b0c6-47a2-b37c-d8e65a629c18/resourceGroups/Default-RecoveryServices-ResourceGroup/providers/Microsoft.RecoveryServices/vaults/testVault",
  "type": "Microsoft.RecoveryServices/vaults",
  "sku": {
    "name": "Standard"
  }
}
```

## Next steps

[Create a backup policy for backing up an Azure VM in this vault](backup-azure-arm-userestapi-createorupdatepolicy.md).

For more information on the Azure REST APIs, see the following documents:

- [Azure Recovery Services provider REST API](/rest/api/recoveryservices/)
- [Get started with Azure REST API](/rest/api/azure/)

