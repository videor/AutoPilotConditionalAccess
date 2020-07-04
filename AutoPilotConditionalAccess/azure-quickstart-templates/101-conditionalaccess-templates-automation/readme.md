# One-click deploy policy blueprints using Conditional Access APIs

You can use the conditional access APIs to manage policy blueprints that you can make available to your branch offices and subsidiaries. For example, you can:

1. As a central admin, add a new policy blueprint using conditional access API JSON schema to a shared onedrive. All subsidieries and branch offices who have been given access to this onedrive will receive notification to create a new conditional access policy based on the policy blueprint. Admins in branch offices and subsidiaries can approve or reject this blueprint. If admins accept the policy blueprint, the conditional access APIs will be used to deploy the blueprint to consented branch offices and subsidiaries! 

2. Update or delete a conditional access policy blueprint using conditional access APIs and rollout the changes to your branch offices and subsidiaries.

3. Enable or disable the conditional access policy blueprint state using conditional access APIs and rollout the changes to your branch offices and subsidiaries. This may be needed in case of business continuity.

This automation can be very useful for organizations that manage large numbers of Azure AD tenants representing branch offices and subsidiaries. Identity partners that need to provide policy blueprints to a number of their customers regularly. Or even in cases of mergers and acquisitions. 

In this tutorial, you'll learn how to:

- Authenticate your app to Azure AD with the right permissions.
- Create policy blueprints using conditional access schema.
- Add new policy blueprint to all your branch offices and subsidiaries in one click.
- Update and remove a policy blueprint and rollout the changes.
- Update state of a policy blueprint and rollout the changes.

# Pre-requisites

If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you start.




