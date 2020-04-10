---
title: "Perfume.com on AWS"
date: 2010-02-28T15:23:32-07:00
draft: true
toc: false
images:
tags: 
  - cloud
  - aws
  - ec2
---

"Perfume.com has grown into a leading online fragrance and beauty superstore, providing more than 12,000 brand name perfume, cologne, hair care and skin care products directly to consumers around the world at discount warehouse prices. Perfume.com is an Internet Retailer Top 500 E-Retailer."

Over the past three months, I have had the opportunity at Live Current Media to build a new platform for Perfume.com. The goal of the project was to reduce cost while increasing site performance and redundancy, *insert cloud marketing pitch here*. The new site infrastructure had to be able to scale up and most importantly down with the rise and fall of traffic. System failures had to been handled automatically and seamlessly, minimizing downtime as much as possible.

The goals had been set and a raw path laid out, the big decision had to be made... Which cloud provider should be used? Amazon Web Services. The decision had pretty much been decided before I joined the team. Live Current Media has deployed websites using the cloud in the past. Cricket.com, for example, runs on Rackspace Cloud Servers, but Amazon's broad range of services was too tempting.

I will begin with a brief layout and explanation of the infrastructure. This is just the first of many posts to come.

Amazon Web Services to be used:

- EC2 (Elastic Compute Cloud) with Auto Scaling 
- S3 (Simple Storage Service) 
- ELB (Elastic Load Balancing) 
- CloudWatch 

To help with the explanation, I created this very pretty diagram for you.

![perfume.com architecture](/images/2010-02-28/perfume-arch.png)

Starting at the top, a host makes a request to www.perfume.com, a cname (alias) to an ELB (Elastic Load Balancing) host name. Amazon's ELB service provides the ability to distribute incoming traffic across your EC2 instances in an availability zone (data center) or multiple if required. ELB can detect the health of EC2 instances, distributing traffic to only healthy application servers ([More about ELB](http://aws.amazon.com/elasticloadbalancing/)). Since redundancy is important, application servers are placed into two availability zones, having ELB check their health and send the request to an application server that is ready to process it.

An "Auto Scaling" group spans both availability zones, an Amazon service to start up and shut down EC2 instances when certain conditions are met ([More about Auto Scaling](http://aws.amazon.com/autoscaling/)). The application servers are part of this Auto Scaling group to provide the ability to scale up and down by adding or reducing the number of application servers (using CPU utilization as a metric). Newly created application instances are automatically added to ELB, having traffic distributed to them when they become healthy (ready to accept requests).

When an application server requires information from the database or shared cache, it queries using an "Elastic IP Address", a static IP address associated with an account and not a particular instance (it maps to an instance). The Elastic IP can be remapped fairly quickly, allowing for a quicker database/cache failover, reducing site downtime ([More about Elastic IP Addresses](http://aws.amazon.com/ec2/#features)). A database server resides in each of the availability zones, exact replicas in case of a server or complete availability zone failure. Database failover occurs automatically, using a Heartbeat over a VPN tunnel to detect failure and a script using Amazons API to remap the Elastic IP Address.

Static website content is served from S3, Amazon's Simple Storage Service ([More about S3](http://aws.amazon.com/s3/)). Moving the static content onto S3 allows for the infrastructure to focus on the dynamic components, and allow for the use of Amazons CDN (Cache Delivery Network) CloudFront, for low latency and extremely fast data transfers ([More about CloudFront](http://aws.amazon.com/cloudfront/)).

To monitor the dynamic infrastructure, CloudWatch is used, Amazons monitoring tool for cloud resources. CloudWatch provides basic visibility into resource utilization (cpu, memory, network) and is required for Auto Scaling groups ([More about CloudWatch](http://aws.amazon.com/cloudwatch/)).

I hope you found this quick post interesting, perhaps I will do a follow up in the near future.
