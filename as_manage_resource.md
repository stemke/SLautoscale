---
copyright:
  years: 1994, 2018
lastupdated: "2018-02-09"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Managing traffic spikes with auto scale resource-based scaling

Launching a new product or having a sale on an existing product can cause a spike of traffic on your website. A spike in traffic can affect your response time, which affects your customers’ experience with your website. SoftLayer Auto Scale enables you to automatically respond to traffic spikes. An example of how Auto Scale can be used to control traffic spikes is through resource-based scaling. In this scenario, a Dallas, Texas-based e-commerce website requires three virtual servers to be online at all times. Business spikes between the hours of 9:00 AM and 5:00 PM every weekday when the company holds online sales. To help sustain website response times, three additional virtual servers are needed during these times. Additionally, when public inbound traffic averages over 5 MB per second across all virtual servers for 10 minutes, an additional two virtual servers should be provisioned. When traffic falls below 5 MB per second, those additional virtual servers should be cancelled and removed. The goal is to have no more than five virtual servers handling the traffic surge, which will keep the weekday virtual servers from being impacted by the additional weekday traffic. The following steps take you through how to set up Auto Scale to support resource-based scaling.

## Adding an Auto Scale Group

1\. Log into IBM Cloud and select **Devices, Auto Scale**. Click on **Add Auto Scale Group** in upper right corner of the screen.

2\. Set up an auto scale group by first entering a **Group Name**, e.g., Weekend Scale Up Group. Select the desired region and data center.
  1. Select the server termination policy. For example, if you choose Oldest, the server with the oldest provisioned date is selected when scaling down.
  * Indicate if your group is to be a multi-VLAN scale group and how to balance scaling down across the configured VLAN pairs
  * Select the public and private VLANs on which you want your servers provisioned. If the servers are to be accessed from public Internet ( if the VLAN is running web servers), leave **Private Network Only** unchecked.
  * Set the **Minimum Member Counts** and **Maximum Member Counts** to 3 and 6, respectively. Set the **Cooldown Period** to 0. Because we’re using schedule-based scaling, there are no statistics gathered to trigger scaling actions.

## Defining member configuration for you group

3\. Scroll down to **Member Configuration** and enter a **Hostname** and **Domain** for the scaling server. Subsequent servers added to the group have unique suffixes appended to the host name.
  1. Specify desired hardware configuration of computing instances (cores, RAM, and so on.)
  * Select an operating system if you want to install the minimal operating system. You can also use a **Public Image** or **Private Image** to provision your instances.
  * If the instances require extra steps to install additional software, specify the URL of a **Post-Install Script** to install the software. (If you use a load balancer for the group, the post-install scripts are run before a member is placed behind a load balancer.)

## Setting up policies

In our scenario, the e-commerce website is using resource-based scaling. Two policies are needed – one to have a relative scale action of +3 for the needed timeframe, and a second to have an absolution action of 3 once the spike is over. The first policy is to add the resources.

4\. Scroll down to **Policies** and click **Add Policy**. Enter the **Policy Name**, (Weekday Scale Up).
  1. Select the group cooldown option for the **Cooldown Period**.
  * Click **Add Trigger**, and specify a repeating trigger by selecting **Every** in the first drop-down, then highlight **Monday, Tuesday, Wednesday, Thursday,** and **Friday** and **2:00 PM**\* from the subsequent drop-down boxes. (The **Advanced Edit** checkbox lets you enter the trigger frequency in crontab–like notation.)
  * Click the **Scale By** drop-down box under **Action** and select **Exact**. Enter 3 in the **Members** field.
  * Click on **Add Policy** again to specify the second policy that removes the servers every afternoon. The cooldown period is the same as the group’s cooldown. The trigger is every Monday, Tuesday, Wednesday, Thursday, and Friday at 10:00 PM\*, with an exact scale action of 3.The time entered in the **Triggers** field is based on UTC time, which is why you need to select 2:00 PM for 9:00 AM Central Daylight Time and 10:00 PM for 5:00 PM Central Daylight Time. During Central Standard Time, the times would be 1:00PM and 9:00 PM respectively because UTC/GMT doesn’t acknowledge daylight savings time. Click [here](http://www.worldtimeserver.com/current_time_in_UTC.aspx) for a time converter.
  * Set up another group with Cooldown of 0, named, for example Traffic Burst Group, with a minimum member count of 0 and a maximum of 5. Use the same settings for group and member configuration as the Weekday Scale Up group, except for the group name.
  * Click on **Add Policy** to add the second policy that controls the two additional servers when public inbound traffic averages over 5 MB per second across all virtual servers for 10 minutes.
    1. Enter the **Policy Name**, e.g., Traffic Burst Group.
    * Select the group cooldown option for the **Cooldown Period**.
    * Click **Add Trigger**. Leave the default **If my** in the first box and select **Public Network Incoming Mbps** for the second box. Leave the default > sign in the third box. In the fourth box, enter **5** (5 MB), enter **600** in the fifth box and select **Seconds** in the last box to represent 10 minutes.
    * Click the **Scale By** drop-down box under **Action** and select **Relative**. Enter 2 in the **Members** field.
    * Click on **Add Policy** again to specify the second policy that removes the servers when the traffic has decreased to fewer than 5 Mbps\*The cooldown period will be the same as the group’s cooldown. The trigger is if my public network incoming Mbps is less than 5 (5 Mbps) for a period of 600 seconds (10 minutes).

When both groups are created, the Weekday Scale Up group creates three servers (the minimum). On weekdays, at 9:00 AM Central Time, three more servers are added, and at 5:00 PM Central time, the servers are removed. There is no other way for the members to be added or removed from that group. In a completely separate Traffic Burst Group, two servers are added for every 10 minutes in which the inbound public traffic across all members averages at 5 MB per second, until it hits the group maximum of five. After the inbound public traffic stays under that amount for 10 minutes, all servers in this group are removed.
