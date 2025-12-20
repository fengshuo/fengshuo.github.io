---
title: AWS IAM User, Group, Role, Policy
categories:
- Frontend
- Web
tags:
- AWS
- Amazon Web Service
- IAM
- Identity & Access Management
- user
- group
- role
- policy
- permission
- account
- security
description: Introduction of IAM user, group, role, policy, and differences between
  them
date: 2019-02-23
author_profile: true
classes: wide
---

When you create a new user/group/role, by default this user/group/role doesn't have any permissions, you must use a policy to grant permissions.

After creating a new user, for instance, you just create an admin user because you shouldn't use root user for day to day operations, you can go to IAM page to grant permissions, either attach policies already created by AWS, or create a custom policy. In this case, AWS actually has an admin access policy you can just use for the newly created admin user.

Groups makes it easier to manage multiple users with same policy, say you have a dev group, and you assign all the permissions(via policies) that are needed, then you can add users to this group, which is similar to other permission control system like creating groups in database and assign user to a group.

IAM roles are similar but also very different.

> An IAM role is an IAM entity that defines a set of permissions for making AWS service requests.

However, roles are not associated or assigned to a specific user or group, instead, trusted entities *assume* roles, trusted entities includes users, applications or services like EC2.

Why do we need to use roles? Roles allows you to delegate access to *trusted entities* without having to share long-term access keys. When trusted entities assume roles, it will call AWS Security Token Service (STS) AssumeRole APIs and those APIs will return temporary credentials. That means the major difference between a user and a role is: users have long term credentials and can interact with AWS services directly, but roles can not make direct requests to AWS services, it has to be assumed by entities such as users, service, etc.

One common scenario of using role is to give people different ([tutorial](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html)). For instance, you only want certain members in the dev team to have both staging and production access, for the rest of the team, you only want them to have staging access. You can create one user A role with production access, another user B role with staging access. And you want 5 team members to act as user A, you can ask them to create their own AWS account, setup the user A role, this way it *allows management of resources across AWS accounts using a single user ID and password.*

Or you want a consultant to do a security check without creating a new user for the consultant, you can create a read only role, and let the consultant to switch to that role. In a nutshell, using IAM roles, you can take advantage of cross-account access to five users access across AWS accounts when they need it.

Another use case is for applications or AWS services like EC2 to assume a role, and requesting other services such as S3, this way you need to maintain long term security credentials for each entity that requires access to a resource.
