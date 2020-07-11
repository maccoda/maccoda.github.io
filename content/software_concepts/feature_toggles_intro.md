+++
date = 2020-07-11
title = "Why you no longer need feature branches"
description = "An introduction to feature toggles/flags and the benefits of using them over feature branches"
[taxonomies]
tags = ["feature toggles", "continuous delivery", "feature flags"]
+++

The core of what software engineers do is add new and improved features to a service or application. We add a feature the customer requested or address some technical debt. Some of these features can take hours to complete whilst others weeks. All the while you are part of a whole team working on numerous features at once, so the question comes how do merge all these features together once they are ready?

## Feature Branches
One such option is to use feature branches, of the popular workflow [Gitflow], where a new branch is forked off the main branch and all feature development completed on this branch. The branch is then merged back on when the feature is completed. The benefit here is the isolation from the main branch so it is not disturbed with a half complete feature. It also means that as each feature is completed the merge request only needs to focus on the feature being merged and is independent from any other features that are being worked on in parallel.

There are some points of concern here. If you are the lucky or unlucky one, your choice, working on the larger feature then you are constantly needing to merge changes from the main branch as each new smaller feature, bug fix, or any work is committed. This adds a fair bit of churn to this workflow as the developer on this larger feature will be going back and forth between solving merge conflicts and implementing the feature.

This then extends the time it takes for the feature to be completed, leading to the second concern that we end up having long lived feature branches and we start to lose continuous integration. The longer a feature branch lives the more it diverges and the greater the risk of the merge. One way we can definitely improve this is the feature definition process to make these as small as possible but that is separate from the point of this article.

## Enter Feature Toggles

As continuous delivery has become a more prevalnt concept other solutions have developed working on getting these features out as quickly as possible. What this has resulted in is the notion of feature toggles.

Feature toggles at its simplest can be thought of as a switch. It is either off or on. This switch controls whether or not the feature is off or on in our application. Feature toggles have greater functionality than this but let's explore the benefits before we delve into those.

A major benefit with feature toggles is its enabling of main branch development, continuous integration just got a whole lot easier. How do we do this? When we start work on our new feature first thing we do is create a feature toggle and wrap our changes in a conditional on the state of the toggle.

```
if (featureToggleService.isLaunched("NEW_HEADER") {
	addHeader(document)
}
```

By doing this it means that we can implement the feature over several commits but since the feature toggle is off in production our new feature code isn't activate in production. This also means that the changes become smaller meaning those merge conflicts should be less and less.

The biggest benefit is the separation of feature work completion and feature launch. With the feature branch model it wouldn't be unheard of to block the deployment to production or block the merge to the main branch until business, product, or testing team are ready for the launch. This can lead to several problematic circumstances, we have a big bang deployment to production containing a lot of changes so if something does go wrong it is harder to pinpoint the exact issue, or in the case the branch isn't merged the developers have to keep this feature branch up to date well after the feature work has been completed. Feature toggles on the other hand allow us to push our code out to production even when it is incomplete. Once the work is completed we can then hand control over to the interested team who is able to be responsible for launching that feature by simply flicking the switch. Decoupling the code completion to the feature launch entirely.

Another element is the safety lever feature toggles provide. In the feature branch model we need to revert all commits and push it through the pipeline if an issues was found. For the feature toggle it is yet again a simple flick of the switch. Not only is there less time and risk in this "roll back" but it can also be controlled by anyone responsible for the feature and not just the engineering team.

## Other functionalities of feature toggles

Hopefully I have been able to show the benefit of feature toggles however the above is not the only benefits we get.

Feature toggles can typically be dialed up progressively, meaning we are able to test it on a smaller audience to see if all goes well prior to everyone getting the change. An example being we could give the new feature to 10% of the customer base, then after an hour move to 50%, etc. Giving even more confidence when launching these features.

We can also usually override the controls for a feature toggle. That is if the testing team want to test the change for a certain test customer or the UX team want to test how actual customers interact with the change this can be done without affecting all customers.

Depending on which feature toggle service you use there may be even more options available, or you might find your company already has one you can improve.

## Some Feature Toggle Services

Below are a few of the feature toggle services I have heard of:

- [Split.io](https://www.split.io/)
- [Launch Darkly](https://launchdarkly.com/)

There are plenty more just have a search for "feature toggle service", it is also sometimes called "feature flag"

[Gitflow]: https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow