---
copyright:
  years: 1994, 2018
lastupdated: "2018-02-19"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Auto scale FAQs

## Does Auto scale support Bare Metal Scale instances?

Auto scale does not support Bare Metal Auto Scale instances at this time.

## Does auto scale work with load balancers?

Yes, auto scale currently works for local load balancers and leverages portion of the load balancer API. For more information, visit [Scale Load Balancers](auto-scale-terms.html#scalelb).

## What are the different auto scale termination policies?

There are 3 auto scale termination policies: newest, oldest, and closest to next charge. These describe how the group chooses what member to take away when scaling down. For more information, visit [Termination Policy](auto-scale-terms.html#termination).

## What are auto scale policies?

A policy holds a set of actions and a set of triggers. Policies typically are comprised of the following elements: name, cooldown, action, and triggers. For more information, visit [Policies](auto-scale-terms.html#policies).

## What are auto scale triggers?

Triggers are conditionals that can be satisfied (only when the group is active). There are three types of triggers: one-time, repeating, and resource-use. For more information, visit [Triggers](auto-scale-terms.html#triggers).

## What is the Maximum VSI Member Count?

It is the most members a group allows to be present. For more information, visit [Maximum VSI Member Count](auto-scale-terms.html#max_virtual_member).

## What is Minimum VSI Member Count?

It is the fewest members a group allows to be present. For more information, visit [Minimum VSI Member Count](auto-scale-terms.html#min_virtual_member).

## What is an auto scale asset?

An asset is a fixed member in a group that does not contribute to the member count and is not automatically controlled in any way. The asset is present to provide additional information to the policy triggers. For instance, an asset can contribute to the whole group CPU percentage if CPU percentage is being used as a trigger. Currently, a group can only have virtual guest assets and there is no upper limit to how many. Additionally, a group is not required

## What is an auto scale group?

A group (scale group) is a regional group specific collection of assets, members, scale load balancers, VLANs, and, policies. A group has all the parameters for scaling including the member count boundaries and what the scaled VSI member templates.

## What is an auto scale member?

A member is a scaled unit in a group. Members are automatically provisioned or reclaimed based on policy actions. Currently all members are VSIs. On an active group, there is never fewer members than the minimum or more members than the maximum. Although members are usually added by actions of a policy, they can also be manually added by a user.
