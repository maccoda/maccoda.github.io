+++
date = 2020-10-11
title = "Common traps to avoid when using feature toggles"
description = "Feature toggles are a great tool but can easily be misused. In this article I go through some of the poor uses of feature toggles that have made them a bit more painful than useful"
[taxonomies]
tags = ["feature toggle", "feature flag"]
+++

In a [previous blog](/software_concepts/feature_toggles_intro) post I discussed the benefits of feature toggles and how they unlock the ability to have true continuous delivery. As with everything there are certainly ways in which you should not use these or traps to avoid that I wanted to share also.

## Aim for short lived toggles

Creating a feature toggle does not come for free, it add extra branching within the code base and is extra maintenance for all involved. To mitigate this extra cost you should strive to keep feature toggles as short lived as possible. Once the feature has been launched work should be queued up to remove the touch points of this feature toggle.

One benefit to keeping them short lived is that we do not allow the code complexity to creep up. It is not that difficult to imagine having several features being developed in parallel that could have some overlap in the code base. In this case the developer adding in the second feature needs to implement it on both sides of the feature toggle since they cannot be certain that the previous feature is fully launched as that is the nature of the feature toggles, it could be turned off at any moment.

Another simple improvement is that keeping code around that is using the feature toggle well after it has been launched is dead code but dead code that your compiler cannot let you know about. This just bloats your code base and will likely confuse developers not privy to the currently launched state.

A rarer issue is that people may become dependant on the feature toggles to set up the system into a certain state because it exists for so long. Most feature toggle services allow for overrides for specific identifiers which means these can easily be misused and hiding a shortcoming in the system.

## Do not use them as online configuration

Do not do this, please! Whilst these are a very cheap way to manage online configuration they are certainly a very bad way to do it. This is not the purpose of feature toggles as this promotes long lived feature toggles which is discussed above. If you wish to have online configuration with a graphical interface there are many alternatives to do this that are built with this purpose, it is just never good to conflate the purpose of a tool as you don't always know the implications particularly since certain design decisions may have been made based on assumed use cases.

Even more importantly though is now throughout the code base a feature toggle could now be two things, either launching a feature or a configuration manager. As with all things if it is not clear people will misinterpret and make mistakes.

## Keep the checks on the feature toggle to as few as possible

This is similar to just diving into writing code before giving it any thought. You should not just add a feature toggle check on a whim but instead should spend some time thinking about the best location for it. The criteria for the *best location* would be the following:

- Reduce the number of systems that need to perform the check
- Reduce the number of touch points in each system for the feature toggle
- Ensure it gates the entire feature

The third point I will explain further later, so I will elaborate on the first two. For both of these there is a common theme of reducing the scope of influence the feature toggle has.

We want as few systems/services as possible to have to know about a single feature, similarly to how we want the domain to be scoped to the correct context. I am not saying here that you should force it to be in as fewer systems as possible but you always assess before you add a dependency of a feature toggle to a system. A simple check is to ask "Does this system care or need to know about feature X?", if it does not need to know about the feature it certainly does not need to know about the feature toggle.

Once we have determined which systems/services should house the feature, we then want to also limit the number of places we need to perform a check of the feature toggle within them. The less conditional forking we add to the code the simpler it is and in particular the simpler it is to remove the checks once the feature is launched. I once had to remove with tens of places across multiple systems which was a very slow process not only because of the amount of changes required but because when a feature is spread that thin it is very easy for it to become coupled to other elements. When this occurs it can tend not to be just the remove of the if branch but can sometimes require a refactor. To avoid this scenario I do recommend that once you know which system/service is responsible for the feature really consider the design for how you can enable the feature and consider the cost of clean up. If required perform a refactoring prior to make it easier to have fewer touch points. Personally I always strive to have a single touch point where possible.

## Avoid needing more than one toggle per feature

The benefit of feature toggles is that they allow for a controlled launch with percentage based launches. This benefit can easily be forfeited if we are not careful about how you choose to gate your features. What you should strive for is one toggle for one feature to be launched. Let's go through a counter example to explain why.

Imagine we needed to implement a new feature where we needed some changes in system A and system B. We decide in this case that we should one feature toggle in each service because we do not want to share across services (this is our mistake). We then get on and complete the tasks for each of the services. When it comes time to launch we realise that we actually cannot dial them up independently! This is because feature toggles typically use some identifier to ensure that they show the same treatment for the same customer, since we now have two toggles we have no guarantee that the same customer sees the feature in both services. So what happens if we need this to be on across both services (as typically if it is not we will either provide a poor customer experience or get them into a weird state)? The only way to ensure we do not place any customers in such a state is we turn them both on fully at the same time! This may not seem bad and in plenty of cases it isn't but when your customer base is large enough you can be going from 0 to thousands of customers instantly.

What exactly does this mean now? Well we have lost the ability to slowly test our feature out as we slowly crank up the feature. We have lost the ability to experiment on the feature and see if it is valuable or not. Mainly we have lost the main safety net we had.

The moral of this point is that just because you have service boundaries does not equate to feature boundaries. Always consider the launch strategy when choosing the number of toggles and the touch points.

In summary choose wisely how you use your feature toggles as each toggle added adds complexity to the system state, so you will want to ensure they are removed when they can
