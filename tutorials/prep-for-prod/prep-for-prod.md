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

You have added test cases in your application. Follow the steps in the [Add Test Cases](add-test-cases) tutorial that is part of the [Develop a Full-Stack CAP Application Following SAP BTP Developer’s Guide](https://developers.sap.com/group.cap-application-full-stack.html) tutorial group.


### Add SAP HANA Cloud

1. In SAP Business Application Studio, go to your **IncidentManagement** dev space.

    > Make sure the **IncidentManagement** dev space is in status **RUNNING**.

2. From the root of the **INCIDENT-MANAGEMENT** project, choose the burger menu, and then choose **Terminal** &rarr; **New Terminal**.

3. To add an SAP HANA Cloud client to your application, run the following command:

    ```bash
    cds add hana --for production
    ```

    > The **cds add hana** command adds the **@sap/cds-hana** module that allows SAP HANA Cloud to access the **package.json** file and the database configuration **"db": "hana"** that uses SAP HANA Cloud when the application is started on production.
    >
    > The **cds add hana** command adds to the **package.json** file the highlighted lines:

    ```json[5, 11-13]
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
                    "db": "hana"
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
    > ```bash
    > {
    >   middlewares: true,
    >   auth: { kind: 'jwt', vcap: { label: 'xsuaa' } },
    >   db: { impl: '@sap/cds/libx/_runtime/hana/Service.js', kind: 'hana' }
    > }
    >```

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

2. Make sure that the following lines have been added to the **package.json** file:
    
    ```json[5, 13]
    {
      "name": "incident-management",
      "dependencies": {
          ...
          "@sap/xssec": "^x"
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

    > In case any of the lines is missing, go ahead and add it manually. 

3. Check the content of the **xs-security.json** file.

    You have already added authorization with the **requires** annotations in the CDS service model (that is the **services.cds** file in the **srv** folder). See [Add Authorization](add-authorization).

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

You can learn more about authorization in CAP in [CDS-based Authorization](https://cap.cloud.sap/docs/guides/security/authorization).


### Prepare HTML5 applications with deploy configurations


1. Run the following command in the terminal:

    ```bash
    cds add html5-repo
    ```

1. Make sure that the following content is added to **app/incidents/package.json**:

    ```json[11-31]
    {
      "name": "incidents",
      "version": "0.0.1",
      "description": "An SAP Fiori application.",
      "keywords": [
          "ui5",
          "openui5",
          "sapui5"
      ],
      "main": "webapp/index.html",
      "scripts": {
          "deploy-config": "npx -p @sap/ux-ui5-tooling fiori add deploy-config cf",
          "build:cf": "ui5 build preload --clean-dest --config ui5-deploy.yaml --include-task=generateCachebusterInfo",
          "build": "ui5 build preload --clean-dest --include-task=generateCachebusterInfo",
          "start": "ui5 serve"
      },
      "devDependencies": {
          "@sap/ui5-builder-webide-extension": "^1.1.8",
          "ui5-task-zipper": "^0.5.0",
          "mbt": "^1.2.18",
          "@ui5/cli": "^3.11.0",
          "ui5-middleware-simpleproxy": "^3.2.10"
      },
      "ui5": {
          "dependencies": [
              "@sap/ui5-builder-webide-extension",
              "ui5-task-zipper",
              "mbt"
          ]
      },
      "private": true
    }
    ```

2. Check the newly-created file **ui5-deploy.yaml** in the folder **app/incidents/**.

    ```yaml
    # yaml-language-server: $schema=https://sap.github.io/ui5-tooling/schema/ui5.yaml.json
    specVersion: '2.4'
    metadata:
      name: ns.incidents
    type: application
    resources:
      configuration:
        propertiesFileSourceEncoding: UTF-8
    builder:
      resources:
        excludes:
          - "/test/**"
          - "/localService/**"
      customTasks:
      - name: webide-extension-task-updateManifestJson
        afterTask: replaceVersion
        configuration:
          appFolder: webapp
          destDir: dist
      - name: ui5-task-zipper
        afterTask: generateCachebusterInfo
        configuration:
          archiveName: nsincidents
          additionalFiles:
          - xs-app.json
    ```

3. In the terminal, navigate to the **app/incidents/** folder and run the following command:

    ```bash
    npm install
    ```

### Run a test build

To validate everything is prepared as expected, run a test build with the following command:

```bash
cds build --production
```
You should get an output like:

```bash
[cds] - build completed in 511 ms
```