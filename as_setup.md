---
copyright:
  years: 1994, 2018
lastupdated: "2018-02-09"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Setting up auto scale


Following are the basic steps to set up auto scale. Click here for specific steps to set up scheduled-based scaling, and here for resource-based scheduling.

1. Log into SoftLayer Portal and select Devices and then [Auto Scale](https://control.softlayer.com/autoscale).
* [Create an Auto Scale group](how-do-i-add-auto-scale-group.html). An auto scale group has a unique [name](auto-scale-terms.html#group_name) and is tied to a [region](auto-scale-terms.html#region). Within the group, a [termination policy](auto-scale-terms.html#termination) is set up, which describes how the group selects which [member](auto-scale-terms.html#group_name) to remove when scaling down to normal server levels. Within the group configuration, the virtual server resources can be set to private only and the public and private VLANs may also be selected. The group settings include the selection of the [minimum member count](auto-scale-terms.html#min_virtual_guest), the [maximum member count](auto-scale-terms.html#max_virtual_guest), and the [Group Cooldown](auto-scale-terms.html#cooldown).
* Configure member servers. Member configuration includes many of the same steps used when purchasing a virtual server instance. Each virtual server instance has an established hostname to which a unique value will be added. For example, device name `socialhost.mysocialwebsite.com` becomes the member name `socialhost-2be5.mysocialwebsite.com`, with the unique value being `2be5`.<br/>![Figure 1](images/member_config.png)
* During member configuration the selection of local or remote storage are network (SAN) is selected; as well as the setup of post installation scripts required for your environment.
* Configure the policy. Every group requires [policies to be configured](how-do-i-setup-auto-scale-policy.html) to manage the automated scaling of virtual server resources. Note: A group may have more than one policy. A [policy](auto-scale-terms.html#policies) holds actions and [triggers](auto-scale-terms.html#triggers) used for scaling. Currently, there is only one type of action, a scale action, which controls how members are added or removed from a group. There are three types of triggers that are an optional feature of policy configuration. Triggers let you configure execute scaling actions in different ways. Only one trigger needs to be satisfied for the action to be invoked.<br/>![Figure 2](images/policy_name.png)
* Apply local load balancers. A group can contain local [load balancer](auto-scale-terms.html#scalelb) settings to be applied to members once provisioning is complete. Local load balancers work and leverage some of the load balancer API. The virtual IP address of the load balancer must be on the same account as the group and it must be in a valid location in accordance with the VLAN, member configuration, and group region. Currently if any load balancers are provided, member configuration MUST include the data center. To access the Local Load Balancers or to order one go to the [Local Load Balancer](https://control.softlayer.com/network/loadbalancing/local) page in the customer portal and select the load balancer you would like to configure.
  * Click on the Edit link on the top right. This will allow you to edit the balancing method, virtual port, and the connection allocation.
  * To the right of the Edit link there is an Add Service link. This will allow the configuration of the basic settings such as IP address, port, and health check type.<br/>![Figure 3](images/load_balancer_details.png)

Click [here for additional Load Balancing](../load-balancing/load-balancing.html) information. To learn more about Auto Scale and for answers to specific questions, click [here](auto-scale-faq.html) for the Auto Scale FAQs.
