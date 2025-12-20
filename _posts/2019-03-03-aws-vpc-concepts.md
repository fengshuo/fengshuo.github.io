---
title: AWS VPC(Virtual Private Cloud) Concepts
categories:
- DevOps
- Containers
tags:
- docker
description: amazon web service - virtual private cloud concepts
date: 2019-03-03
author_profile: true
classes: wide
---

In [this post](https://shuofeng.dev/deploy-a-more-secure-rails-app-on-aws-with-vpc-part1/) I wrote how to deploy a more secure rails app with VPC and NAT gateway. In this post, I want to write down AWS VPC related concepts to have a better understand what we are dealing with and why we are supposed to setup VPC in a certain way.


# VPC

> Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways. You can use both IPv4 and IPv6 in your VPC for secure and easy access to resources and applications.

In another words, VPC resembles on premise data center or corporate network you have control of networking configuration, and it is also scalable.

AWS release VPC after EC2 service, so if you have an old account, there is another networking platform called EC2-classic. But if you are using accounts created after December 2013, your account will have a default VPC created in each region with a default subnet created in each availability zone.

A few important things to know about VPC including:

1) One VPC can only be in One Region, but one VPC can span multiple availability zones in a region, this means you can have redundant resources in separate AZ but allow them to communicate with each other within the same VPC, which provides foundation for high availability and fault tolerant infrastructure.

2) When you create an Amazon VPC, you must specify the IPv4 address range by choosing a CIDR block.

3) You can launch instance into a subnet.

4) You can use layers of security in VPC with NACL(Network Access Control Lists) and security group.

# Subnet

A subnet is a segment of a VPC's IP address range. CIDR blocks define subnets (Although AWS reserves the first four IP address and the last IP address of every subnet for internal networking purposes).

One subnet must reside entirely in one availability zone and cannot span zones. However, you could have multiple subnets in one AZ.

Subnets can be classified as public, private, or VPN-only. A public subnet means it is associated with a route table that has an Internet Gateway attached, a private subnet is associated with a route table that doesn't have Internet Gateway, instances launched into a private subnet cannot communicate with the open internet, the benefit of it is higher level of security, although it has drawbacks, such as the instance cannot install software, but it can be solved by routing traffic through a NAT instance.

![attach-igw-to-route-table](/assets/images/content/vpc-attach-igw-to-route-table.png)

Subnets must be associated with a route table, newly created subnets are automatically associated with the default route table('main'), and by default all subnets traffic is allowed to communicate with each other via the local target in the route table.

# Internet Gateways

Internet Gateway is a VPC component that allows communication between instances in VPC and the Internet. It is horizontally scaled, redundant and highly available. The way you use Internet Gateway is to attach it to your VPC (only 1 IGW can be attached to a VPC), and use it as a target in a route table with subnets associated.

![attach-igw](/assets/images/content/vpc-attach-igw.png)

What's also interesting about IGW is that it also provides NAT(network address translation) translation for instances that have been assigned a public IP address. EC2 inside a VPC are only aware of their private IP address, when an instance receives traffic from the Internet, the IGW translates the destination address (the public IP address or Elastic IP address) to the instance's private IP address, and vice versa, it also maintains this public IP address to private IP address mapping.

Note that you must assign a public IP address or Elastic IP address to enable EC2 instance to send and receive traffic from the Internet.

Note that you cannot detach IGW if there is any instance having public IP address(or Elastic IP address)

# Route Tables

A route table contains a set of rules to determine where network traffic is directed, it has destination(The CIDR block range of the target) and a target (a name identifier of where the data is being routed to). The default VPC already has a route table named 'main'.

Each route table contains a default route called the local route, which enables communication within the VPC, you cannot modify the local route.

![private-subnet-route-table](/assets/images/content/vpc-private-subnet-route-table.png)

Since you can have multiple route tables in a VPC, it's better to create new route tables when needed.

# Network Access Control Lists (NACL)

NACL is another (optional) layer of security that acts like a stateless firewall **on a subnet level**, stateless means that you have to specify both inbound and outbound rules(`ALLOW` or `DENY` rule). Rules are evaluated in order, starting with the lowest rule number, the last rule is a catch-all deny rule. It's better to have an increment of 10 for rule number so that it won't cause an issue if you need add a certain rule.

![vpc-nacl-rules](/assets/images/content/vpc-nacl-rules.png)

# Security Group

Security groups are similar to NACL, but with some differences, the first difference is security group is stateful, which means you don't have to specify outbound rule(return traffic is allowed regardless of rules), it only support `ALLOW` rules, and it adds security on **instance level**.
