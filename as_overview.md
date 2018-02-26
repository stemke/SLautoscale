---
copyright:
  years: 1994, 2018
lastupdated: "2018-02-09"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# About auto scale

Auto scale provides you with the ability to automate the manual scaling process associated with adding or removing virtual servers to support your business applications. Auto scaling can be managed via the API or control portal, and enables

* Seamless and automatic scaling up of virtual servers when additional resources are required due to demand .
* Seamless and automatic scaling down of virtual servers by shedding unnecessary resources when demand goes down (saving you money)
* Flexible scaling triggers, including CPU percentage being used, outgoing public and private bandwidth, and incoming public and private bandwidth.
* Near-real-time status updates for scaling activity in groups.
    Optional integration of virtual LAN (VLAN) and local load balancers.

There are two common business solutions to which auto scaling can be applied - [schedule-based](traffic_spikes.html) and [resource-based](as_manage_resource.html) scaling. Schedule-based scaling can be used when a company is expecting traffic to spike, for example, a social networking site that requires additional resources based on a schedule. Resource-based scheduling occurs when there is a push to get a product to market or an e-commerce site is having a sale and resources are needed to sustain response times. Before you can use auto scale, you need to have

* An IBM Cloud account.
* Permission to use auto scale. Permission is granted by the account “Master User,” who automatically receives auto scale permission.
* Access to all Virtual Servers.

Auto Scale uses groups to contain the policies that change how your environment expands or shrinks. These policies use actions to add or remove virtual server based upon your business and application needs.
