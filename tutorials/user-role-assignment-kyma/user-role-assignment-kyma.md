---
title: Assign the User Roles
description: This tutorial shows you how to assign role collections to users. 
parser: v2
auto_validation: true
time: 15
tags: [ tutorial>beginner, software-product-function>sap-cloud-application-programming-model, programming-tool>node-js, software-product>sap-business-technology-platform, software-product>sap-fiori]
primary_tag: software-product-function>sap-cloud-application-programming-model
author_name: Grzegorz Karaluch
author_profile: https://github.com/grego952
---

## You will learn

- How to assign a role collection in the SAP BTP subaccount.


## Prerequisites

- You've deployed your application in the SAP BTP, Kyma runtime. Follow the steps in the [Deploy in SAP BTP, Kyma Runtime](deploy-to-kyma) tutorial that is part of the [Deploy a Full-Stack CAP Application in SAP BTP, Kyma Runtime Following SAP BTP Developer's Guide](https://developers.sap.com/group.deploy-full-stack-cap-kyma-runtime.html) tutorial group.
- You have an [enterprise global account](https://help.sap.com/docs/btp/sap-business-technology-platform/getting-global-account#loiod61c2819034b48e68145c45c36acba6e) in SAP BTP. To use services for free, you can sign up for an SAP BTPEA (SAP BTP Enterprise Agreement) or a Pay-As-You-Go for SAP BTP global account and use the free tier services only. See [Using Free Service Plans](https://help.sap.com/docs/btp/sap-business-technology-platform/using-free-service-plans?version=Cloud).
- You have a platform user. See [User and Member Management](https://help.sap.com/docs/btp/sap-business-technology-platform/user-and-member-management).
- You're an administrator of the global account in SAP BTP.
- You have a subaccount in SAP BTP to deploy the services and applications.
- You have a tenant of SAP Cloud Identity Services. See [Get Your Tenant](https://help.sap.com/docs/cloud-identity-services/cloud-identity-services/get-your-tenant) for details how to get a tenant of SAP Cloud Identity Services if you don't have one yet.
- You've established trust between your tenant of SAP Cloud Identity Services and your SAP BTP account. This established trust allows you to use your SAP Cloud Identity Services tenant as an identity provider or a proxy to your own identity provider hosting your business users. See [Establish Trust and Federation Between SAP Authorization and Trust Management Service and SAP Cloud Identity Services](https://help.sap.com/docs/btp/sap-business-technology-platform/establish-trust-and-federation-between-uaa-and-identity-authentication).
- You have one of the following browsers that are supported for working in SAP Business Application Studio:
    - Mozilla Firefox
    - Google Chrome
    - Microsoft Edge

> This tutorial follows the guidance provided in the [SAP BTP Developer's Guide](https://help.sap.com/docs/btp/btp-developers-guide/what-is-btp-developers-guide).

### Create a role collection and add role

1. Open the SAP BTP cockpit and navigate to your subaccount.

2. Choose **Security** &rarr; **Role Collections**, and then choose **Create**.

3. In the **Create Role Collection** popup, enter **Incident Management Support** in the **Name** field and choose **Create**.

4. Choose the role collection **Incident Management Support** from the list of role collections and choose **Edit**.

5. Open the value help in the **Role Name** field.

6. Search for the role **support**, select it, and choose **Add**.

7. Choose **Save**.

### Assign a role collection to a user


1. Choose **Security** &rarr; **Users**, and then choose a user from the list.

2. Under **Role Collections** on the right, choose **Assign Role Collection**.

3. In the **Assign Role Collection** dialog, select the **Incident Management Support** role collection and choose **Assign Role Collection**.

      You have assigned the **Incident Management Support** role collection to your user.

2. In the **Assign Role Collection** dialog, select the **support** role collection and choose **Assign Role Collection**.

      ![role collection](./rolecollection11.png)

      You've assigned the role collection to your user.

> Log out and log back in to make sure your new role collection is considered.
