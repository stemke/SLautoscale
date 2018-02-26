---
copyright:
  years: 1994, 2018
lastupdated: "2018-02-10"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Managing weekly traffic spikes with auto scale schedule-based scaling

Web traffic spikes can be a good thing and a bad thing. Spikes are good in the sense that more people are using your site; bad because of how a surge in activity can affect your response times. Auto scale provides you with ability to automatically scale your servers up or down to support your business needs. An example of how auto scale can be used to control traffic spikes is through schedule-based scaling. In this scenario, a Dallas, Texas-based social website experiences activity spikes during weekends. Two virtual servers are the minimum required by the site Monday through Friday; however, two additional virtual servers are needed every weekend to account for the increased traffic. The following steps take you through how to set up auto scale to support schedule-based scaling.

## Adding an Auto Scale Group

1\. Log into IBM Cloud and select **Services, Auto Scale**. Click on **Add Auto Scale Group** in upper right corner of the screen.

2\. Set up an auto scale group by first entering a **Group Name**, (Weekend Scale Up Group). Select the desired region and data center.
  1. Select the server termination policy. For example, if you choose **Oldest**, the server with the oldest provisioned date is selected when scaling down.
  * Indicate if your group is to be a multi-VLAN scale group and how to balance scaling down across the configured VLAN pairs
  * Select the public and private VLANs on which you want your servers provisioned. If the servers are to be accessed from public Internet (if the VLAN is running web servers), leave **Private Network Only** unchecked.
  * Set the **Minimum Member Counts** and **Maximum Member Counts** to 2 and 4, respectively. Set the **Cooldown Period** to 0. Because we’re using schedule-based scaling, there are no statistics gathered to trigger scaling actions.



## Defining member configuration for your group

3\. Scroll down to **Member Configuration** and enter a **Hostname** and **Domain** for the scaling server. Subsequent servers added to the group have unique suffixes appended to the host name.
  1. Specify desired hardware configuration of computing instances (cores, RAM, and so on.)
  * Select an operating system if you want to install the minimal operating system. You can also use a Public Image or Private Image to provision your instances.
  * If the instances require extra steps to install additional software, specify the URL of a Post-Install Script to install the software. (If you use a load balancer for the group, the post-install scripts are run before a member is placed behind a load balancer.)



## Setting up policies

In our scenario, the social website is using schedule-based scaling. Two policies are needed – one to scale the servers up on Friday afternoons, and a second to scale the servers down on Sunday evenings. The first policy is to scale the servers up.

4\. Scroll down to **Policies** and click **Add Policy**. Enter the **Policy Name**, (Friday Scale Up).
  1. Select the group cooldown option for the **Cooldown Period**.
  * Click **Add Trigger**, and specify a repeating trigger by selecting **Every** in the first drop-down, then select **Friday** and **10:00 PM**\* from the subsequent drop-down boxes. (The **Advanced Edit** checkbox lets you enter the trigger frequency in crontab–like notation.)
  * Click the **Scale By** drop-down box under **Action** and select **Relative**. Enter 2 in the **Members field**.
  * Click on **Add Policy** again to specify the second policy that scales the servers down on Sunday evenings. The cooldown period is the same as the group’s cooldown. The trigger is every Sunday at 1:00 AM*, with a relative scale action of -2.


\*The time entered in the **Triggers** field is based on UTC time, which is why you need to select 10:00 PM for 5:00 PM Central Daylight Time and 1:00 AM for 8:00 PM Central Daylight Time. During Central Standard Time, the times would be 9:00 AM and 12:00 AM respectively because UTC/GMT doesn’t acknowledge daylight savings time. Click [here](http://www.worldtimeserver.com/current_time_in_UTC.aspx) for a time converter.

## Adding a local load balancer

If you have a local load balancer set up, you can use it to load balance your auto scale group.

5\. Scroll down to **Local Load Balancers** and click on **Add Load Balancer**.
  1. Click on **View VIP Settings** (opens a separate page) to see local load balancers available in your account, and order new one if needed. (A load balancer needs to have service group associated with to be used with an auto scale group. Make sure the load balancer is located in the data center of your auto scale group.
  * Click **Add Load Balancer** on the Add Auto Scale Group page, click the drop-down arrow for Virtual IP, and select your load balancer.
  * Click the drop-down arrows for Service Group, Service Port, and Service Health Check Type and select the appropriate values.



6\. Click on **Add Group** to save your group.

  * If all your data centers, networks, and so on, are correctly specified, your auto scale group is created. Click on the group on the View All Groups screen to see your group details.


The result of setting up a schedule-based scaling is that two members are automatically provisioned to meet the minimum requirement of the website. On Fridays at 5:00 PM Central Time, two more members  are automatically provisioned when the first policy trigger fires. Then, on Sundays at 8:00 PM Central Time, two members are reclaimed thanks to the second policy trigger. If you specify a load balancer, all members created either at creation, or on Friday, are behind the load balancer after having the custom post-install script run.
