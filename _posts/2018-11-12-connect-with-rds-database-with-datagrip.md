---
title: Connect a RDS database with DataGrip
categories:
- Backend
- Systems
tags:
- RDS
- AWS
- databases
- DataGrip
- ssh
description: Steps to Connect a RDS database with DataGrip
date: 2018-11-12
author_profile: true
classes: wide
---

After creating the bastion host and RDS instance(in a private subnet), you might want to connect to the database with a client such as DataGrip. But since the RDS instance is in a private subnet(the general rule is to not allow database access directly from the internet), a few more steps are needed to connect to the database with a remote client.


The first step is to connect to the database like usual:
![datagrip-1](/assets/img/posts/2018-11-12-connect-with-rds-database-with-datagrip/datagrip-1.png)

The second step is to use SSH key to setup SSH:
![datagrip-2](/assets/img/posts/2018-11-12-connect-with-rds-database-with-datagrip/datagrip-2.png)


Link [1](https://medium.com/cory-mayfield/linking-amazon-rds-with-jetbrains-datagrip-d5cc0e2f44f4), link [2](https://github.com/aws-samples/aws-refarch-wordpress/issues/17#issuecomment-353771386)
