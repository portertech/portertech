---
title: "Sensu a Monitoring Framework"
date: 2011-01-11
draft: false
toc: false
images:
tags: 
  - sensu
---

At [Sonian](http://sonian.com), we monitor an ever changing number of [Amazon EC2](http://aws.amazon.com/ec2) instances. As I write this post, that number is **476**, expected to rise and fall before the day is done. The "**elastic**" nature of our infrastructure makes monitoring it a not so trivial task.

We have found the standard tools from the community toolbox to be inadequate when operating in the "**cloud**". Up until recently, Sonian utilized several tools in conjunction to monitor systems and collect metrics; [Nagios](http://nagios.org), [Collectd](http://collectd.org), [Graphite](http://graphite.wikidot.com), and [Ganglia](http://ganglia.info).

### The Evolution of Nagios at Sonian.

Our servers are grouped into "**stacks**", providing isolated environments that are globally distributed. In the past, a Nagios server would reside in each one. The servers were responsible for monitoring the components of their stack, triggering notifications when something was amiss. Check coverage was gradually increased over time, as applications began to require more moving parts. As the number of stacks increased, a centralized view of the organization was desired. To appease the engineering teams, a distributed Nagios solution was created. The monitoring server in each stack would forward their check results to a central Nagios server running the [**N**agios **S**ervice **C**heck **A**ccepter](http://exchange.nagios.org/directory/Addons/Passive-Checks/NSCA--2D-Nagios-Service-Check-Acceptor/details). The central server ran the Nagios web interface, displaying the status of every client and service under our control. Notifications could only be triggered by the central server, making it easier to silence notifications for a client or one of its services.

### The Neanderthal.

Nagios is **NOT** designed for the cloud. It expects your environment to be fairly static, with every aspect of it under your control. The initial release was May 14th, 1999. The concept of elastic **I**nfrastructure **a**s **a** **S**ervice didn't exist at the time of its creation, and it has failed to adapt.

Nagios' inability to discover clients is an excellent indication of its antiquity. Nagios must know of every client, group, and service on start. When a new server is spun up, the Nagios configuration must be updated and the service reloaded in order to begin monitoring it. **C**onfiguration **M**anagement is commonly used in this case as a semi-solution; using a method of server discovery, re-writing the configuration, and then triggering a service reload. It's only a semi-solution as the process usually only happens on a set interval or is too intensive for frequent changes. Distributing Nagios in a tiered fashion only complicates this further, making it far more difficult to begin monitoring a new server or deploy new checks. The following diagram depicts a sample of events that would require Nagios configuration changes.

![Problems with Nagios configuration](/images/2011-11-01/nagios-diagram.png)

### Our problems with Nagios

* Configuration is unpleasant & restrictive
* Cannot discover new servers on its own
* Easily overwhelmed with a high number of clients & checks
* Difficult to extend & hack

### A Brief Introduction To Sensu.

Enter [Sensu](http://www.sonian.com/cloud-tools/cloud-monitoring-sensu), a monitoring framework that aims to be simple, malleable, and scalable.

![Sensu architecture diagram](/images/2011-11-01/sensu-diagram.png)

### The Building Blocks.

In this modern world of computing, we're lucky to have ever improving **C**onfiguration **M**anagement tools, such as [OpsCode Chef](http://www.opscode.com) and [Puppet](http://puppetlabs.com). These tools already gather the information needed to effectively and efficiently monitor your systems. Not only are these tools a rich source of data, but they can also handle the distribution of supporting libraries and plugins. Sensu was built with the intention of being paired with a CM tool.

Message-oriented middleware is commonly used by developers to decouple and distribute components of their applications. [RabbitMQ](http://www.rabbitmq.com) is a robust messaging system that has proven itself at Sonian. Sensu uses RabbitMQ to securely route check requests and results, making it possible to scale out and back in on demand.

Open source key-value data stores have been around for a long time, recently gaining a lot of attention with NoSQL being all the rage. [Redis](http://redis.io) is a very fast in-memory "data structure server", with keys that can contain strings, hashes, lists, sets, and sorted sets. Its support for atomic operations and ability to persist to disk has made it a common choice for new projects. Sensu uses Redis as a non-persistent database, to store client and event data.

### The Concept.

The idea behind Sensu is simple, schedule the remote execution of checks and collect their results. As mentioned above, Sensu uses RabbitMQ to route check requests and results, this is the secret sauce. Checks will always have an intended target; servers with certain responsibilities, such as serving web pages (webserver) or data storage (elasticsearch). A Sensu client has a set of subscriptions based on its server’s responsibilities, the client will execute checks that are published to these subscriptions. A Sensu server has a result subscription, this is where clients publish check results. Since each component only connects to RabbitMQ, there is no need for an external discovery mechanism, new servers are monitored immediately.

### Code.

Sensu is written entirely in Ruby, using the [EventMachine](http://rubyeventmachine.com) library for evented IO, producing a fully functional, clean, and small code base.

Sonian has made it publicly available [on GitHub](https://github.com/sensu/sensu).

```
Language    files     blank   comment    code
Ruby            9        56         0     695
SUM:            9        56         0     695
```

### Configuration.

All configuration is done with JSON files, making it easy for Configuration Management and other automation tools to create and read them. The following are configuration snippets.

Client attributes

    {
      "client": {
        "name": "i-424242",
        "address": "127.0.0.1",
        "subscriptions": ["all", "webserver"]
      }
    }

Check definition

    {
      "chef_client": {
        "notification": "Chef client daemon is not running.",
        "command": "/etc/sensu/plugins/chef_client.rb",
        "subscribers": ["all"],
        "interval": 60
      }
    }

I hope this very shallow dive into Sensu has spiked your interest. For the nitty gritty, please check out the [GitHub repository](https://github.com/sensu/sensu), and jump on IRC (irc.freenode.net #sensu). I realize you probably have many questions, drop them into a comment bellow and I will do my best to fill the holes.

### What’s Next?

My plan for the next few days is to produce documentation ([wiki](https://github.com/sensu/sensu/wiki)) and a Vagrantfile for provisioning a VM with either OpsCode Chef or Puppet.
