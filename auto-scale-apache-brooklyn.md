---
copyright:
  years: 1994, 2018
lastupdated: "2017-02-19"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Auto scale with Apache Brooklyn

## Introduction

A company that provides critical consumer data to over a billion mobile users daily tasked SoftLayer with providing functionality to create auto-scaling groups of servers located on a private network. A key requirement was elasticity; where, based on the workload demands and user-experience response time, systems automatically scale up and scale down based on predetermined policies.

Auto scaling is based on CPU utilization. If average CPU utilization of the servers in a group is higher than the highest threshold, or trigger event, more resources need to be provisioned, or scaled up. If average CPU utilization of the servers in a group is lower than the lowest threshold, or trigger event, resources need to be scaled down. The customer had the following requirements:

1. Ability to specify the following parameters:
  * Initial number of servers in the group
  * Minimum and maximum number of servers in the group
  * High and low CPU thresholds
  * Number of additional servers to be scaled up and down on a trigger event
* Integrate SoftLayer Domain Name System (DNS)
  * After a server in the group is provisioned, or scaled up, and available, an appropriate SoftLayer DNS record is to be created for it.
  * When server is scaled down, the SoftLayer DNS record for it is to be removed.
* Integrate a Load Balancer
  * After a server in the group is provisioned, or scaled up, and available, the server record is to be added to the appropriate NetScaler load balancer and bound to the appropriate load-balancing group.
  * When server is scaled down, it has to be unbound from the load balancing group and the server record is to be removed from the NetScaler.
* Integrate Chef
  * After a server in the group is provisioned, or scaled up, and available, the Chef client is to run and register the node at the Chef server.
  * When server is scaled down, the server’s node and client record is to be removed from the Chef server.
* Server provisioning requirements
  * All servers in the group are to be provisioned as SoftLayer virtual machines (VMs) with private-only uplinks, 1,000MB port speed, and a specified private VLAN.
  * VMs are to be provisioned from the multi-disk standard template, created by the customer in SoftLayer.
* Operating system
  * CentOS 7.

Based on the customer’s requirements, Apache Brooklyn was selected as the auto-scaling tool to implement the solution.

## Apache Brooklyn Overview

Apache Brooklyn is a framework for modeling, monitoring, and managing applications through autonomic blueprints. Click [here](https://brooklyn.incubator.apache.org/) for more information.

### Apache Brooklyn Concepts

The Apache Brooklyn documentation provides definitions of the Apache Brooklyn concepts such as

* Entities
* Application
* Parent and Membership Configuration
* Sensors and Effectors
* Lifecycle and Management Context
* Dependent Configuration
* Location, Policies
* Execution
* Implementation

![Figure 1: Solution design for load balancing and auto scaling](images/figure1-1.png)

`Figure 1: Solution design for load balancing and auto scaling`

### Integrating SoftLayer DNS, NetScaler, and Chef

Apache Brooklyn provides a base class for customizing location, `brooklyn.location.jclouds.BasicJcloudsLocationCustomizer`, which contains methods that allow for creating additional parameters when machines are provisioned. Actions can be executed after a machine is provisioned, and before and after it has been released when cancelled.

We created our own location customizer class that extended the `brooklyn.location.jclouds.BasicJcloudsLocationCustomizer` provided by Brooklyn. For our example, we will refer to it as `brooklyn.location.jclouds.ProjectJcloudsLocationCustomizer`.

The following three methods were overridden:

1. `public void customize(JcloudsLocation location, ComputeService computeService, TemplateOptions templateOptions)` - The method gets invoked before calling the cloud API to provision a server. We enabled the ability to supply provisioning properties such as `privateNetworkOnly, primaryBackendNetworkComponentVlanId, portSpeed,` and others.

* `public void customize(JcloudsLocation location, ComputeService computeService, JcloudsSshMachineLocation machine)` - The method gets invoked after a machine is provisioned and started, thus all server parameters, like server IP address, are available. The code to add a server to DNS and the load balancer has to be placed in this method. SoftLayer DNS and NetScaler REST API are used to create DNS records and bind the server to NetScaler. We utilized a package provided by `Brooklyn, brooklyn.util.http`, to submit REST calls to SoftLayer DNS and NetScaler APIs.

* `public void preRelease(JcloudsSshMachineLocation machine)` - The method gets invoked on machine cancelation before the server is stopped. We added REST calls to SoftLayer DNS and NetScaler here to remove DNS and NetScaler records and unbind the server from NetScaler’s group.

We did not have to do anything specific to connect the new server to Chef. The image, which was used to provision the server, had a Chef client installed on it and start-up script (the script runs the Chef client and connects it to the Chef server). If you prefer, you can utilize Brooklyn integration with Chef to create blueprints.

We also added a REST call to the Chef server to remove appropriate client and node records.

### Application blueprint

Within the application blueprint, we customized the location, metric collecting parameters, and auto-scaling properties of the servers. Following are the commands used to customize each component.

    name: Group1
    location:

      jclouds:softlayer:tor01:
      customizerType: brooklyn.location.jclouds.ProjectJcloudLocationCustomizer
      privateNetworkOnly: true
      primaryBackendNetworkComponentVlanId: 12345
      portSpeed: 1000
      imageId: 123456
      dnsZoneId: 12345
      netscalerGroup: app1
      netscalerGroupPort: 8080
      netscalerUrl: http://user:password@10.10.10.10/nitro/v1/

    services:
    - type: brooklyn.entity.group.DynamicCluster
      id: cluster
      name: dynamic cluster
      initialSize: 1
      memberSpec:
        $brooklyn:entitySpec:
        name: VMinSL
        type: brooklyn.entity.basic.EmptySoftwareProcess
        brooklyn.initializers:
        - type: brooklyn.entity.software.ssh.SshCommandSensor
          brooklyn.config:
            name: server.cpucount
            command: "LANG=en_US.UTF-8 sar -u 1 1|grep Average | awk '{print 100 - $8}'"
            targetType: java.lang.Double
            period: 10s
          brooklyn.config:
            post.launch.command: "sudo apt-get install -y sysstat"

      brooklyn.enrichers:
      - type: brooklyn.enricher.basic.Aggregator
        brooklyn.config:
          enricher.sourceSensor: $brooklyn:sensor("server.cpucount")
          enricher.targetSensor: $brooklyn:sensor("server.cpucount.averaged")
          enricher.aggregating.fromMembers: true
          transformation: average
        brooklyn.policies:
        - policyType: brooklyn.policy.autoscaling.AutoScalerPolicy
        brooklyn.config:
          metric: $brooklyn:sensor("server.cpucount.averaged")
          metricLowerBound: 10
          metricUpperBound: 60
          minPoolSize: 1
          maxPoolSize: 5

### Location

In the location section of the blueprint we specified provisioning properties such as

* Where the server is to be previsioned (data center and VLAN)
* Whether the server will be provisioned with privateNetworkOnly
* An image ID of the SoftLayer standard template or flex image,
* Port speed

We also specified the SoftLayer DNS zone ID, which is used by `ProjectJcloudLocationCustomizer` to add and delete records in the DNS zone, and NetScaler information for binding and unbinding the server from NetScaler.

    location:

      jclouds:softlayer:tor01:
        customizerType: brooklyn.location.jclouds.ProjectJcloudLocationCustomizer
        privateNetworkOnly: true
        primaryBackendNetworkComponentVlanId: 12345
        portSpeed: 1000
        dnsZoneId: 12345
        netscalerGroup: app1
        netscalerGroupPort: 8080
        netscalerUrl: http://user:password@10.10.10.10/nitro/v1/

### Services

In the services section of the blueprint, we specified

* That we were deploying a group of servers
* What Brooklyn was to install on each server
* How CPU metrics were to be collected and used
* The parameters of the auto-scaling policy

#### Application spec

    services:
    - type: brooklyn.entity.group.DynamicCluster #group of servers
      id: cluster
      name: dynamic cluster
      initialSize: 1 #number of servers initially created in the group

#### Member spec

The `brooklyn.entity.basic.EmptySoftwareProcess` call was used because Apache Brooklyn only needed to provision servers from an image and no additional installation or configuration was required. This section also specified a sensor - server.cpucount – which periodically collects CPU utilization from the server.

    memberSpec:
      $brooklyn:entitySpec:
      name: VMinSL
      type: brooklyn.entity.basic.EmptySoftwareProcess
      brooklyn.initializers:
      - type: brooklyn.entity.software.ssh.SshCommandSensor
        brooklyn.config:
          name: server.cpucount
          command: "LANG=en_US.UTF-8 sar -u 1 1|grep Average | awk '{print 100 - $8}'"
          targetType: java.lang.Double
          period: 10s
      brooklyn.config:
        post.launch.command: "sudo apt-get install -y sysstat"

### Metrics

In the metrics section of the blueprint, metrics from individual servers are collected and the average is assigned to the cluster level sensor.

    brooklyn.enrichers:
      - type: brooklyn.enricher.basic.Aggregator
        brooklyn.config:
          enricher.sourceSensor: $me0$brooklyn:sensor("server.cpucount.averaged")
          enricher.aggregating.fromMembers: true
          transformation: average

### Auto scaling policies

In this section, auto-scaling policy properties are defined:

    brooklyn.policies:
    - policyType: brooklyn.policy.autoscaling.AutoScalerPolicy
      brooklyn.config:
        metric: $brooklyn:sensor("server.cpucount.averaged")
        metricLowerBound: 10
        metricUpperBound: 60
        minPoolSize: 1
        maxPoolSize: 5
        minPeriodBetweenExecs: 10m
        resizeUpIterationIncrement: 2
        resizeUpIterationMax: 2
        resizeDownIterationIncrement: 2
        resizeDownIterationMax: 2

Once the solution was deployed, users were able to seamlessly auto scale up and down their applications in a timely manner, registering and de-registering the system records with load balancing, DNS, and Chef servers.
