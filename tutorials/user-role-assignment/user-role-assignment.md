---
title: Assign the User Roles
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

- You have an [enterprise global account](https://help.sap.com/docs/btp/sap-business-technology-platform/getting-global-account#loiod61c2819034b48e68145c45c36acba6e) in SAP BTP. To use services for free, you can sign up for a CPEA (Cloud Platform Enterprise Agreement) or a Pay-As-You-Go for SAP BTP global account and make use of the free tier services only. See [Using Free Service Plans](https://help.sap.com/docs/btp/sap-business-technology-platform/using-free-service-plans?version=Cloud).
- You have an S-user or P-user. See [User and Member Management](https://help.sap.com/docs/btp/sap-business-technology-platform/user-and-member-management).
- You are an administrator of the global account in SAP BTP.
- You have a subaccount in SAP BTP to deploy the services and applications.
- You have one of the following browsers that are supported for working in SAP Business Application Studio:
    - Mozilla Firefox
    - Google Chrome
    - Microsoft Edge
- You have deployed your application in either the SAP BTP, Cloud Foundry runtime or the SAP BTP, Kyma runtime. See [Deploy in SAP BTP, Cloud Foundry Runtime](deploy-to-cf) for deploying to the SAP BTP, Cloud Foundry runtime and [Deploy in SAP BTP, Kyma Runtime](deploy-to-kyma) for deploying to the SAP BTP, Kyma runtime.

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

      You have assigned the **Incident Management Support** role collection to your user.

> You might need to log out and log back in to make sure your new role collection is taken into account.
