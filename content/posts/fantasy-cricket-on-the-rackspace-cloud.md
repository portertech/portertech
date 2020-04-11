---
title: "Fantasy Cricket on the Rackspace Cloud"
date: 2010-05-28
draft: false
toc: false
images:
tags: 
  - cloud
  - rackspace
---

I feel like sharing what we've been working on at Gastown Labs, so here is an outline of the infrastructure for [Fantasy Cricket](http://fantasy.cricket.com). Fantasy Cricket lives on the [Rackspace Cloud](http://www.rackspace.com/cloud/) (Cloud Servers), providing on-demand resources to handle traffic spikes and growth.

![Fantasy Cricket architecture outline](/images/2010-05-28/fantasy-cricket-infrastructure.png)

A user makes a request for fantasy.cricket.com, which resolves to a network load balancer. The load balancer determines which application server will serve the request, however, the load balancer also serves the static content for the request (style sheets, images, etc). The application server will churn away at the request, pulling data from cache or querying the database. The database master handles all of the application writes, where the two replicas serve the read requests. Live cricket scores are updated via XML feeds uploaded to the load balancer and processed with a background process running on one of the application servers.

#### Load Balancer

Nginx is used to handle web requests and act as a reverse proxy to the application servers using its HTTP proxy module. The load balancer has a complete copy of the application, including static content which is served directly from here. Nginx uses its Memcached module to cache specific page assets, reducing future page loading times. VSFTPD is used to allow specific live cricket score feed providers to update the application, XML files are uploaded via FTP to a specified folder for processing.

#### Application Servers

Each application server uses Nginx to send requests to a Unicorn worker pool over a Unix Domain Socket. Unicorn workers are responsible for executing the application. There is an excellent post on Unicorn in detail [here](http://tomayko.com/writings/unicorn-is-unix), explaining on why it's so awesome. Background processes are distributed amongst the application servers, they are responsible for processing XML feeds and updating team scores etc. Each application server runs Memcached, each a part of the cache pool (volatile) used by the application and load balancer.

#### Database Servers

MySQL is our relational database of choice, Fantasy Cricket uses statement-based replication. All write operations are performed to the master, which are then replicated to the two slaves. Read operations are balanced between slaves, more slaves can be added to deal with an increase in read operations. Each database server runs MemcacheDB, not to be confused with Memcached. MemcacheDB is a distributed key-value storage system for persistent storage, used to cache assets that are too expensive to place in a volatile cache.
