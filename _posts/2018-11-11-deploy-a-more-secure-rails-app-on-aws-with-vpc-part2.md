---
title: Deploy A More Secure Rails App on AWS(ElasticBeanStalk, VPC, NAT gateway, Bastion
  Host) - Part 2
categories:
- Backend
- Systems
tags:
- rails
- aws
- vpc
- ElasticBeanStalk
- gateway
- Route table
- Bastion Host
- security
- cloud
description: Steps to deploy A More Secure Rails App on AWS(ElasticBeanStalk, VPC,
  NAT gateway, Bastion Host)"
date: 2018-11-11
author_profile: true
classes: wide
---

The goal of this post is to understand why and how to deploy a secure web app(rails app in this case) on AWS.

- Start a simple rails application with ElasticBeanStalk.
- Create a RDS instance
- Start a Bastion host
- Create database in the RDS instance via Bastion host
- Deploy app via ElasticBeanStalk


# Create a simple rails app with ElasticBeanStalk

Go to ElasticBeanStalk, create a new application, for now, I will just use the sample ruby rails app

![vpc-2-0](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-0.png)

Before you create the new app, click on 'configure more options' , one thing to note is that, if you are using the low cost presets, you can't configure capacity and load balancer options, in this example, we want to add a load balancer for scaling (i won't get into the detail of setting up load balancer here, use the default here, just want to show that you can configure it here too)

![vpc-2-1](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-1.png)

![add elastic load balancer](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-2.png)

Note that you can change the platform setting such as ruby version, make sure the app will be initiated with the right ruby version. Otherwise you might see rails package installations issues(such as [this one](https://gist.github.com/fengshuo/947789fb9f704fb099ef06a27bdcf587)).

![vpc-2-3](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-3.png)

You can change the configuration in the 'network' tab: select the new VPC, assign public subnet to the load balancer, select private subnet in instance settings.

![vpc-2-4](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-4.png)

Click the 'instances' tab, update the EC2 security group to 'Web Server'.

We will set up database separately so no need to change database config here.

when the app is created successfully, click on the link you should see a welcome page.

# Create the RDS instances

Since we are using postgresql, we will create a new RDS instance in the private subnet of the new VPC.

Step 1: Go to RDS dashboard, click 'subnet groups' on the left nav, create a private subnet group:

Step 2: Launch a new instance(you can click only show free tier to avoid potential costs), select postgresql version, give it an instance name and user credential. Go to advanced settings. Here select the VPC we created(demo-vpc), choose a private subnet group, select the databse security groups we created earlier, select no for public accessibility since it's not secure to allow public access to the database.

![vpc-2-5](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-5.png)

![vpc-2-6](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-6.png)

Type in a database name, and leave the rest as default.

Once the instance is up and running, you should be able to see a endpoint of this instance on the instance description tab, we will use this endpoint soon.


# Start A Bastion Host

Once the RDS instance status turns into available, let's create a Bastion server.

Go to EC2 page, create a new ec2 instance under the same VPC with a public subnet, enable public IP address

![vpc-2-7](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-7.png)

in the configure security group, select the bastion server security group

![vpc-2-8](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-8.png)

Before starting the instance, it will ask you about key pairs, if you don't have existing key pairs it will generate one and download the pem file, if you already have one, you can use the existing pair. We will need this to ssh into the instance.

![vpc-2-9](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-9.png)

Now let's ssh into the bastion server with the public IP address (using the key pair), such as:

```
ssh -i ./demo.pem ec2-user@XX.XXX.XX.XX
```

it will give you a hint to `sudo yum update`, then let's install postgres with `sudo yum install postgresql95`.


# Create database in the RDS instance via Bastion host

Now since we are in the bastion host, we have access to the private subnet, let's connect to the RDS instance:

```
psql -h [HOST] -U [USERNAME] -d postgres
```

Now we can setup users and databases [source here](https://blog.cmgresearch.com/2017/05/07/step-3-configure-rds-deploying-a-rails-application-to-elastic-beanstalk.html)

```
CREATE ROLE dev with encrypted password '[STRONG_PASSWORD1]' LOGIN;
GRANT dev TO [MASTER_USERNAME];
CREATE ROLE prod with encrypted password '[STRONG_PASSWORD2]' LOGIN;
GRANT prod TO [MASTER_USERNAME];
CREATE DATABASE dev with owner dev;
CREATE DATABASE prod with owner prod;
```

# Deploy app via ElasticBeanStalk

let's create a new rails app(check your ruby version before you create the new app) with `rails new MyApp --database postgresql`, open the `config/database.yml` file to the production section, as mentioned in the comment, we can use a URL with this syntax `postgres://myuser:mypass@localhost/somedatabase` to connect with the database.

Now let's go the the app in ElasticBeanStalk, click on configuration - software, add two new environment properties:

```
DATABASE_URL: postgres://myuser:mypass@localhost/somedatabase
SECRET_KEY_BASE: (get this by running rails secret in the code)

```

## Create a ElasticBeanStalk user role in IAM with ElasticBeanStalkFullAccess Policy attached

create a new aws profile with `aws configure --profile demo`

![vpc-2-10](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-10.png)

you can add this user to a group has ElasticBeanStalk deploy permission, but here just attach the existing policy ElasticBeanStalkFullAccess to the user

![vpc-2-11](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-11.png)

## Use ElasticBeanStalk CLI

Install it with `brew install aws-elasticbeanstalk`

Init with eb `eb init --profile demo`, it will ask you a few questions, remember to select the correct region, otherwise it cannot find the right application

![vpc-2-12](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-12.png)


## Edit ElasticBeanStalk config files

After the previous step, there should be a new config file in the `.elasticbeanstalk` folder, in that file you can also specify another branch such as `master` branch to deploy to production env. So when you are on master branch, use `eb deploy --profile demo` will deploy it to production env. But you will need to have a prod env in ElasticBeanStalk first.

![vpc-2-13](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-13.png)

The quick way to do it is to clone the existing env, but change the environment property, such as using the prod database and the prod user for postgresql

![vpc-2-14](/assets/img/posts/2018-11-11-deploy-a-more-secure-rails-app-on-aws-with-vpc-part2/vpc-2-14.png)


Links:

- [Setup VPC: Deploying a Rails Application to Elastic Beanstalk](https://blog.cmgresearch.com/2017/05/01/deploying-a-rails-app-to-elastic-beanstalk.html)

- [Deploy a Rails application to AWS – using VPC, EC2, RDS and ELB](http://www.clovescarneirojr.com/2016/02/10/deploy-rails-app-to-aws-vpc-ec2-rds-elb.html)
