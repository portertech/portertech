---
title: "IaC Morning Market"
date: 2012-12-10
draft: false
toc: false
images:
tags: 
  - iac
  - automation
  - devops
  - cloud
  - chef
  - puppet
---

**Morning market**: A daily routine that involves examining rejects made the previous day before work begins so that countermeasures can be adopted as soon as possible.

I recently read "[Gemba Kaizen](http://www.amazon.com/Gemba-Kaizen-Commonsense-Management-ebook/dp/B006402MAG)" by Masaaki Imai, a fantastic book that introduced me to the idea of the morning market. After putting down the book, I began to think of ways I could apply this idea to IT operations, and more specifically **I**nfrastructure **a**s **C**ode.

I would like to share my thoughts on creating a morning market for Infrastructure as Code. This post is fairly OpsCode Chef centric, however, I am certain it can still be helpful to those using another toolkit.

The amount of interest in Chef cookbook design and testing has recently exploded. Tools have emerged, books and blog posts have been written, and conference attendees rant with beer(s) in hand. The amount of energy and traction is fantastic, however, I do not hear enough about the observation of production Chef runs. There is nothing quite like production (yes, a cliché); this is where Chef does its business, ideally bringing systems into an expected state. A Chef cookbook morning market could entail the daily review of Chef runs, followed by the creation of prioritized tasks, to solve immediate problems and improve efficiency.

### The Gathering

A morning market is a problem solving process. In order to fully understand a problem and find a solution, one must first gather relevant data. Chef and many other tools like it, offer a report and exception API, a way to execute code in response to a run succeeding or failing. This API is perfect for gathering the data needed to identify, understand, and fix problems. I have created a few handlers for my own experiments, which are available on GitHub.

* [Chef Metrics](https://github.com/portertech/chef-metrics), for collecting run metrics.
* [Chef IRC Snitch](https://github.com/portertech/chef-irc-snitch), for recording run exceptions and notifying individuals via IRC.
* [Chef Journalist](https://github.com/portertech/chef-journalist), for providing a list of updated resources.

![Chef Metrics](/images/2012-12-10/chef-metrics.png)

At the recent Chef Community Summit, I had the pleasure to attend a session on [Chef Driven Monitoring and Metrics](http://wiki.opscode.com/display/chef/Chef+driven+Monitoring+and+Metrics), where we discussed sub topics ranging from “what to record?” to “tools used?”. A few tools mentioned in the session tickled my fancy, and I recommend checking them out.

* [Chef Elapsed Time](https://github.com/jamesc/chef-handler-elapsed-time), for a per-resource elapsed time report.
* [Grill](http://cyclecomputing.com/grill/overview), a Chef run monitoring tool, unfortunately it’s **freeware**.
* [Bistro](https://github.com/ampledata/bistro), a [Splunk](http://www.splunk.com/) application for Chef, in combination with the [Splunk handler](https://github.com/ampledata/chef-handler-splunk).

These tools can be used in conjunction with traditional monitoring and metric tools, providing visibility into systems and the Chef code that configures them.

### The Meeting

A morning market is a meeting, to quickly review the status of what is being worked on, identify new problems, and quickly manage who is working on what until the next meeting. Problems are not solved during a morning market, it simply facilitates the effective organization of work. There will always be the temptation to discuss details and ask questions, but this is to be done after the meeting, by those assigned to the problem.

Problems identified could be categorized by what is known about them, for instance, the cause of a problem may or may not be known or understood. Categorizing problems could help identify how much effort may be required to solve them. Problems could be prioritized by severity, for example, one that prevents application deployment would have a higher priority than a garbage collection issue. As any process, a morning market would evolve and be refined, and the problems identified will vary as systems change.

* A service fails to start on newly bootstrapped nodes.
* A resource is not idempotent, causing a service to restart on every run.
* Application deployment occasionally fails on a number of nodes.
* Search results are not limited to an environment, causing clustering issues.
* A third-party SCM service outage causes run failures.
* Chef daemon is consuming a large amount of memory, ineffective GC.

### Final Words

I believe any organization can benefit greatly from organized problem solving. I see the morning market as a simple yet effective process for rapid problem resolution. Implementing such a process for **I**nfrastructure **a**s **C**ode seems like a no-brainer, however, there is still much to ponder. How will it coincide with other processes, and will they compliment each other? How will resources, people and their time, be allocated? How will effectiveness be measured?

