---
title: 'Building Your Own Open-Source SIEM, Part I: AAAAAH!'
date: '2022-18-04'
author: hacker0ni
layout: post
categories:
    - Guides
---

# Intro
This will be a post series where I discuss the details of an open-source SIEM and how you can deploy it on your own infrastructure. This will heavily focus on Security Onion 2 as it gives us nice out-of-the-box functionality.

The aim of this series is not to promote any product(s), but to show you that with a little work and some patience, a lot can be achieved.

# To SIEM or not to SIEM
The word SIEM has been around for a while, and many companies are still pouring money into paid services so they can have a 'single pane of glass' into their infrastructure, however with completely free and open-source tools you can also build your own SIEM. The only catch is that it takes work, but the end result is a highly customizable and highly efficient system that only alerts you when specific conditions set by you are met.

![An unironic world-map image that's somehow associated with a SIEM](/assets/img/world-map.jpg)

# Why Security Onion 2
[Security Onion](https://docs.securityonion.net/en/2.3/about.html) also has been around a while, although the earlier versions focused more on a singular VM image that'd sniff traffic and replay PCAPs etc. to do basic threat hunting on a local network. The second version focuses on a distributed deployment with multiple nodes that have different functions from each other, which makes it highly scalable and SIEM worthy.

## Core Components
This section covers some basic component information about Security Onion 2, you can skip this part if you feel like you know enough about how it works.

### The Grid
The Grid is the name Security Onion gives its nodes. It comprises of Forward, Search, Manager, Receiver nodes and they mostly do what their name states. Forward nodes sniff traffic from a SPAN/mirror port on your network using tools like [Zeek](https://zeek.org/), [Suricata](https://suricata.io/) and [Stenographer](https://github.com/google/stenographer). The traffic is then logged and sent to the Manager node where it's relayed into Search nodes.

Manager node is the head of the whole operation, there is usually a single Manager node in every Grid (though if you've managed to configure a multi-manager deployment please let me know lol). This node hosts basic Elasticsearch components like Logstash and ...well Elasticsearch. It also queues the incoming logs with its Redis queue and sends them over to Search nodes for *ingesting* and indexing into Elastic. Receiver node is an alternative Manager node that's capable of gathering logs. Receiver node was introduced later on to address the scalability issues.

### SaltStack
[SaltStack](https://saltproject.io/) is a very powerful configuration management tool with horrible documentation. I challenge you to find any good documentation on it. It uses YAML syntax to configure its 'minions'. Salt Master is the Manager node in Security Onion 2 and you can configure other nodes using it. We'll dive a little deeper into its usage in upcoming posts.

### Elasticsearch
[Elasticsearch](https://www.elastic.co/) is another open-source project that aims to aggregate, correlate and visualize data in any form. Its main components in an SO Grid are Filebeat, Logstash and Elasticsearch. Filebeat picks up logs on every node and forwards them to (well under normal circumstances) Logstash listeners. Logstash manipulates and filters data then sends it over to Elasticsearch where it can be stored, searched and visualized by Kibana.

### Other Components
There are other components in Security Onion 2 that comes in handy when doing threat hunting, alert generation etc. *Cases* works like the JIRA of the platform. *Playbook*  uses sigma rules to generate alerts based on incoming logs. CyberChef allows you to convert data into other kinds of data (think base64'ing or vice-versa), finally FleetDM allows you to query your hosts by deploying Osquery endpoint agents on to them.

# Deployment
This part is always fun! But really all you have to do to deploy Security Onion 2 is to follow the [documentation](https://docs.securityonion.net/en/2.3/index.html). For other unexpected stuff, I tried to cover some of them here.

## Manager Node
This node needs to be the first one to be deployed, since all other nodes will connect to it to retrieve their configurations. For best results make sure it has its own IP address and set the hostname to something you like, it's quite difficult to change it to something else afterwards (might as well re-deploy).

## Search Nodes
You can deploy as many Search nodes as you want, but preferably they should be geographically close to your Manager node if your environment is not on a global scale. We will cover other methods of sending logs into Security Onion 2 by directly hitting Search nodes instead of relaying them through the Manager node, but those methods won't be covered in this post. Ensure that they have good resources because these are the big boys of your Grid where a lot of processing is done.

Search nodes need to be able to communicate with the Manager node directly.

## Forward Nodes
Forward nodes sniff traffic to generate logs. They can be deployed as both IDS and IPS solutions thanks to Suricata. Running through the installation steps is usually a breeze, unless you want to deploy them in a NAT'd environment. Usually all you have to do is configure a SPAN/mirror port on your most-edge router/switch in the network and plug it into the second network interface you have on the hardware. I've tried to sniff traffic before by using a GRE tunnel from another server and royally failed. Let me know if this can be done. Also ensure the NIC that's sniffing the traffic must be able to handle the traffic passing the switch/router it's connected to, if you want to avoid packet loss.

### The NAT Problem and Salt Firewall Configuration
The NAT problem I've mentioned arises because of SaltStack. When a minion registers itself to its master, all the configuration is supposed to be the same in order for them to successfully communicate. So say you have a forward node with a private IP address of 192.168.23.5 and this gets translated to 80.191.22.5 on the edge of the internal network. After the installation is completed, you have to configure the manager node to allow access to both those IPs in the firewall. Because the Forward node (naturally) thinks its IP is the Private IP it's assigned, whereas the Manager node only sees the connection coming from the Public IP address. For good measure, ensure the private IP assigned to the Forward node doesn't change.

But wait! Everything in Security Onion is configured via Salt, so you're not getting away with a simple `iptables` command. The way the firewall is set up in Security Onion is a little complicated but for good reason. There are hostgroups, portgroups and assigned hostgroups. You don't need to edit these corresponding files in the Salt configuration, because you can use the provided commandline tools like `so-allow`. This tool allows certain hostgroups to access certain ports on the node. By default the usage is something like `so-allow analyst 192.168.1.1` which allows the mentioned private IP access to analyst ports. What are analyst ports? Well they're defined [here](https://github.com/Security-Onion-Solutions/securityonion/blob/master/salt/firewall/assigned_hostgroups.map.yam) in the Salt configuration. For more advanced firewall configuration, you can use `so-firewall`. Back to our example, you need to include the Forward node's IP addresses in `minion` and `sensor` portgroups.

You can use the following commands to achieve this:
```
so-firewall includehost minion <IP>
so-firewall includehost sensor <IP>
so-firewall apply
```
Ensure that you add both IPs with the first two commands to both groups so they can communicate. I wanted to cover this section, because this was a common problem amongst the users.

# After Deployment
The initial deployment is a process that you'll get used to after a while, but make sure to document every tidbit you have faced and solved for your own good. Please help the  [community](https://github.com/Security-Onion-Solutions/securityonion/discussions/) by sharing them as well.

Assuming you have a working Grid now, get a feel of the UI and read more about the documentation. It is still an evolving project that deserves the attention in my opinion. Feel free to check out some of the guides I've shared with the community [here](https://github.com/Security-Onion-Solutions/securityonion/discussions/categories/show-and-tell?discussions_q=author%3Ahacker0ni+category%3A%22Show+and+tell%22).

In the upcoming posts I'll get into the nitty gritty of managing an open-source SIEM, this will include connecting an external Logstash instance to the Search nodes for scalability (you can read it now [here](https://github.com/Security-Onion-Solutions/securityonion/discussions/5819)), configuring SSL certificate verification between the external Logstash and the Search node, writing new Ingest Pipelines and Grokking. Hopefully by the end of the series, you'll be able to ingest any kind of data to your open-source SIEM by using the ELK pipeline, generate alerts from them and send them to other sources like Slack. Let me know about your opinions and suggestions. Thanks for reading!