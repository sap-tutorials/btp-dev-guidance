---
title: Prepare for Production
description: As you have followed the tutorials for the Incident Management application, you’ve developed an application with an in-memory database and basic authentication. To have such an application on production, you need to make the respective configurations.
keywords: cap 
parser: v2
auto_validation: true
time: 20
tags: [ tutorial>beginner, software-product-function>sap-cloud-application-programming-model, programming-tool>node-js, software-product>sap-business-technology-platform, software-product>sap-fiori]
primary_tag: software-product-function>sap-cloud-application-programming-model
author_name: Svetoslav Pandeliev
author_profile: https://github.com/slavipande
---

## You will learn

- How to configure SAP HANA Cloud in your project
- How to configure SAP Authorization and Trust Management service in your project


## Prerequisites

You have added the basic test cases in your application. See [Add Test Cases](../../add-test-cases.html).


### Add SAP HANA Cloud

1. In SAP Business Application Studio, go to your **IncidentManagement** dev space.

    > Make sure the **IncidentManagement** dev space is in status **RUNNING**.

2. From the root of the **INCIDENT-MANAGEMENT** project, choose the burger menu, and then choose **Terminal** &rarr; **New Terminal**.

3. To add an SAP HANA Cloud client to your application, run the following command:

    ```bash
    cds add hana --for production
    ```

    > The **cds add hana** command adds the **@sap/cds-hana** module that allows SAP HANA Cloud to access the **package.json** file and the database configuration **"kind": "hana"** that uses SAP HANA Cloud when the application is started on production.
    >
    > The **cds add hana** command adds to the **package.json** file the highlighted lines:

    ```json[5, 11-15]
    {
        "name": "incident-management",
        "dependencies": {
            ...
            "@sap/cds-hana": "^x"
        },
        ...
        "cds": {
            "requires": {
                ...
                "[production]": {
                    "db": {
                        "kind": "hana"
                    }
                }
            }
        }
    }
    ```
    
    > To learn more, see: 
    >
    > - [Using Databases](https://cap.cloud.sap/docs/guides/databases#get-hana)
    > - [CAP Configuration](https://cap.cloud.sap/docs/node.js/cds-env)
    >
    > By default, these are the available profiles: 
    >
    > - For local testing: **development**
    > - For hybrid testing: **hybrid**
    > - For productive testing: **production** 
    >
    > Deployments are done using the **production** profile automatically. You can inspect the effective production configuration with the **cds env** command:
    > 
    > ```bash
    > cds env requires -4 production
    > ```
    >
    > The output of this command looks like this:
    >
    > ```js
    > {
    >   db: {
    >     impl: '@sap/cds/libx/_runtime/hana/Service.js',
    >     kind: 'hana',
    >     'deploy-format': 'hdbtable'
    >   },
    >   auth: { strategy: 'JWT', kind: 'jwt-auth', vcap: { label: 'xsuaa' } },
    >   approuter: { kind: 'cloud-foundry' }
    > }
    > ```

2. Verify that your application still works locally. If you closed it, choose the **Preview Application** option in the **Application Info - incidents** tab and select the **watch-incidents** npm script.

    > To open the **Application Info - incidents** tab: 
    >
    >1. Invoke the Command Palette - **View** &rarr; **Command Palette** or <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> for macOS / <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> for Windows. 
    >2. Choose **Fiori: Open Application Info**.


### Configure the SAP Authorization and Trust Management service

1. Run the following command in the terminal:

    ```bash
    cds add xsuaa --for production
    ```

    > Running **cds add xsuaa** does two things:
    >
    >- Adds the SAP Authorization and Trust Management service service to the **package.json** file of the **INCIDENT-MANAGEMENT** project.
    >- Creates the SAP Authorization and Trust Management service security configuration (that is, the **xs-security.json** file) for the **INCIDENT-MANAGEMENT** project.

2. Check if the following line has been added to the **package.json** file:
    
    ```json[5-6, 14]
    {
      "name": "incident-management",
      "dependencies": {
          ...
          "@sap/xssec": "^x",
          "passport": "^x"
      },
      ...
      "cds": {
        "requires": {
          ...
          "[production]": {
            "db": "hana",
            "auth": "xsuaa"
          }
        }
      }
    }
    ```

3. Check the content of the **xs-security.json** file.

    You have already added authorization with the **requires** annotations in the CDS service model (that is the **processor-service.cds** file in the **srv** folder). See [Add Authorization](../../add-authorization.html).

    ```js
    annotate ProcessorService with @(requires: ['support']);
    ```
    
    This is now translated into scopes and role templates for the SAP Authorization and Trust Management service. Hence, a scope and role template for the **support** role are created in the **xs-security.json** file:

    ```json
    {
      "scopes": [
        {
          "name": "$XSAPPNAME.support",
          "description": "support"
        }
      ],
      "attributes": [],
      "role-templates": [
        {
          "name": "support",
          "description": "generated",
          "scope-references": [
            "$XSAPPNAME.support"
          ],
          "attribute-references": []
        }
      ]
    }
    ```

You can learn more about authorization in CAP in [Authorization and Access Control](https://cap.cloud.sap/docs/guides/authorization).

### Run a test build

To validate everything is prepared as expected, run a test build with the following command:

```bash
cds build --production
```
You should get an output like:

```bash
[cds] - build completed in 511 ms
```