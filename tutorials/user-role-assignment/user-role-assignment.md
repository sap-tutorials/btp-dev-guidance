---
title: User Role Assignment
description: This tutorial shows you how to assign roles to users. 
parser: v2
auto_validation: true
time: 15
tags: [ tutorial>beginner, software-product-function>sap-cloud-application-programming-model, programming-tool>node-js, software-product>sap-business-technology-platform, software-product>sap-fiori]
primary_tag: software-product-function>sap-cloud-application-programming-model
author_name: Svetoslav Pandeliev
author_profile: https://github.com/slavipande
---

## You will learn

- How to create and assign a role collection in the SAP BTP subaccount.


## Prerequisites

You have deployed your application in SAP BTP, Cloud Foundry runtime. See [Deploy in SAP BTP, Cloud Foundry Runtime](../../deploy-to-cf.html)

### Create a role collection and add role

1. Open the SAP BTP cockpit and navigate to your subaccount.

1. Choose **Security** &rarr; **Role Collections**, and then choose **Create**.

      <!-- border; size:540px --> ![Role Collections](./create-role-collection.png)

2. In the **Create Role Collection** popup, enter **Incident Management Support** in the **Name** field and choose **Create**.

      <!-- border; size:540px --> ![Create Role Collection](./create-role-collection-popup.png)

3. Choose the role collection **Incident Management Support** from the list of role collections and choose **Edit** on the right.

      <!-- border; size:540px --> ![Edit Role Collection](./edit-role-collection.png)

4. Open the value help in the **Role Name** field.

      <!-- border; size:540px --> ![Value Help](./role-value-help.png)

5. Search for the role **support**, select it, and choose **Add**.

      <!-- border; size:540px --> ![Add Role](./add-role.png)

6. Choose **Save**.

### Assign a role collection to a user


1. Choose **Security** &rarr; **Users**, and then choose a user from the list.

2. Under **Role Collections** on the right, choose **Assign Role Collection**.

      <!-- border; size:540px --> ![role collection](./rolecollection1.png)

2. In the **Assign Role Collection** dialog, select the **Incident Management Support** role collection and choose **Assign Role Collection**.

      <!-- border; size:540px --> ![role collection](./rolecollection11.png)

