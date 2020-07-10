# AutoPilotConditionalAccess
Autopilot provides advance convenience features that are designed to assist you with the most burdensome parts of managing Conditional Access policies. Autopilot introduces new features and improves existing functionality to make your Conditional Access experience durable and more capable over time.

# Let’s all treat Conditional Access Policies like we treat code!
Hey everyone — My name is Vikas Deora. And I have the privilege of being part of the Azure Active Directory Conditional Access product team in the Microsoft Identity Division. One of the *fantastic* perks of my job is that I get to speak to hundreds of customers each year. That adds up to several thousand customer conversations for our team since the launch of the product. We have had so many light bulb moments because of our customers. We owe our success to these conversations! But one critical insight stands out in my mind as I write this post. And I’d like to pass this insight on to you! And that is 

# Conditional Access Policies should be treated just like code.

# Why?

Conditional Access Policies rolling out through an organization can have benefits and risks similar to code rolling out through the various rings of a service. A new policy could behave just right in your test environment. But how would it behave when deployed to your production environment? At the first touch of the pragmatic realities of your organization, it may demonstrate to you all the corner cases that did not occur to you to test!

And if *that* doesn’t happen in your org — well, some well-meaning individuals with the appropriate privileges may make a change to a Conditional Access policy — which suddenly causes your org not to be compliant with an industry-standard you’re supposed to meet.
Okay, let’s say that makes sense, and you agree with me — policy is not all that different from code. Yet, it is often not treated the way code is handled, right? Conditional Access policies don’t have the same creation, deployment, and management tools that we have developed over decades for software.

# Why?

My thesis is that this primarily because tools haven’t been available that let you do that. When you’re writing code, you can benefit from code templates in your favorite IDE, but when you’re creating a CA policy — you have to start from scratch. Every time!
You can deploy your code to a test environment, and then with a click of a button, roll it out to the next ring, test, and continue this to your production ring. Wouldn’t it be useful if you could do that with CA policies — rather than having to recreate them from scratch in every environment?

With code, you have source control, check-in, check-out, versioning, roll-back, roll-forward. Could your CA policies benefit from the same capabilities? Should you not be able to version your policies and undo any non-compliant changes, control who can edit which policy, and be able to review it before approving its deployment to your org? We are hard at work, changing that. We should no longer think of policy like "ON" or "OFF".

# Instead, the right way to think about the CA policy engine is

1. Configure
2. Deploy
3. Monitor
4. Manage

# And with the Conditional Access APIs, now *you* can too!
With the Conditional Access APIs, you can now nearly eliminate all the risk, uncertainty, and unpredictability currently involved in creating, deploying, and managing your organization’s Conditional Access Policy set. Would you like to learn how?
Over the next few weeks, I’d love to share with you some thoughts, projects, and guidance around how you can now achieve with CA policies (almost) everything you taken for granted with code.

Sound exciting?
Let’s jump in!




