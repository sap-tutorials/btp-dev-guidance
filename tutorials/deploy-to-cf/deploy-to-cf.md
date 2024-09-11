---
title: Deploy in SAP BTP, Cloud Foundry Runtime
description: This tutorial shows you how to deploy your CAP application as a multitarget application (MTA).
keywords: cap 
parser: v2
auto_validation: true
time: 45
tags: [ tutorial>beginner, software-product-function>sap-cloud-application-programming-model, programming-tool>node-js, software-product>sap-business-technology-platform, software-product>sap-fiori]
primary_tag: software-product-function>sap-cloud-application-programming-model
author_name: Svetoslav Pandeliev
author_profile: https://github.com/slavipande
---


## You will learn

- How to deploy your CAP application as multi-target application (MTA)

## Prerequisites

- You have configured the respective entitlements, enabled the Cloud Foundry runtime in your subaccount, and created an SAP HANA Cloud service instance in the SAP BTP cockpit. Follow the steps in the [Prepare for Deployment in the SAP BTP, Cloud Foundry Runtime](prepare-btp-cf) tutorial that is part of the [Deploy a Full-Stack CAP Application in SAP BTP, Cloud Foundry Runtime Following SAP BTP Developer’s Guide](https://developers.sap.com/group.deploy-full-stack-cap-application.html) tutorial group.
- You have an [enterprise global account](https://help.sap.com/docs/btp/sap-business-technology-platform/getting-global-account#loiod61c2819034b48e68145c45c36acba6e) in SAP BTP. To use services for free, you can sign up for an SAP BTPEA (SAP BTP Enterprise Agreement) or a Pay-As-You-Go for SAP BTP global account and make use of the free tier services only. See [Using Free Service Plans](https://help.sap.com/docs/btp/sap-business-technology-platform/using-free-service-plans?version=Cloud).
- You have a platform user. See [User and Member Management](https://help.sap.com/docs/btp/sap-business-technology-platform/user-and-member-management).
- You are an administrator of the global account in SAP BTP.
- You have a subaccount in SAP BTP to deploy the services and applications.
- You have one of the following browsers that are supported for working in SAP Business Application Studio:
    - Mozilla Firefox
    - Google Chrome
    - Microsoft Edge


### Introduction

The SAP BTP, Cloud Foundry environment allows you to create polyglot cloud applications in Cloud Foundry. It contains the SAP BTP, Cloud Foundry runtime, which is based on the open-source application platform managed by the Cloud Foundry Foundation.

The SAP BTP, Cloud Foundry environment enables you to develop new business applications and business services, supporting multiple runtimes, programming languages, libraries, and services.

For more information about the Cloud Foundry environment, see [Cloud Foundry Environment](https://help.sap.com/docs/btp/sap-business-technology-platform/cloud-foundry-environment).

### Set up the MTA for deployment

A multitarget application (MTA) is logically a single application comprised of multiple parts created with different technologies, which share the same lifecycle.

The developers of the MTA describe the desired result using the MTA model which contains MTA modules, MTA resources and interdependencies between them. Afterwards, the MTA deployment service validates, orchestrates, and automates the deployment of the MTA, which results in Cloud Foundry (CF) applications, services and SAP-specific contents.

You will use the [Cloud MTA Build Tool](https://sap.github.io/cloud-mta-build-tool/) to deploy the Incident Management application. The modules and services are configured in the **mta.yaml** deployment descriptor file.

1. In SAP Business Application Studio, go to your **IncidentManagement** dev space.

    > Make sure the **IncidentManagement** dev space is in status **RUNNING**.

2. From the root of the **INCIDENT-MANAGEMENT** project, choose the burger menu, and then choose **Terminal** &rarr; **New Terminal**.

3. Run the following command to generate the **mta.yaml** deployment descriptor:

    ```bash
    cds add mta
    ```

### Configure managed application router

The managed application router enables you to access and run SAPUI5 applications in a cloud environment without the need to maintain your own runtime infrastructure. See [Managed Application Router](https://help.sap.com/docs/btp/sap-business-technology-platform/managed-application-router?version=Cloud).

> See also the documentation about [deploying content with Generic Application Content Deployment](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/d3e23196166b443db17b3545c912dfc0.html)

1. In SAP Business Application Studio, invoke the Command Palette of the INCIDENT-MANAGEMENT application by choosing the burger menu and then choose **View** &rarr; **Command Palette**. Then, choose **Create MTA Module from Template**.

    <!-- border; size:540px --> ![V4 Template](./approuter0.png)

2. Choose the **Approuter Configuration** module template and then choose **Start**.

    <!-- border; size:540px --> ![V4 Template](./approuter1.png)

3. In the **Project Location** step, in the **Specify a path to the root project** field, choose the **incident-management** project. Choose **OK** and choose **Next**.

    <!-- border; size:540px --> ![V4 Template](./approuter2.png)

5. In the **Approuter Configuration** step:

      - In the **Select your HTML5 application runtime** dropdown menu, select **Managed Approuter**.
  
      - In the **Enter a unique name for the business solution of the project** field, enter **incidents**.

      - In the **Do you plan to add a UI?** dropdown menu, select **Yes**.
      
      - Choose **Next**.

      <!-- border; size:540px --> ![V4 Template](./approuter3.png)

7. In the **Action** step, in the **Overwrite incident-management/xs-security.json?** list, select **do not overwrite**.

8. Choose **Finish**.

    <!-- border; size:540px --> ![V4 Template](./approuter4.png)

The above steps will generate the following module and resources in the **mta.yaml** file:

```yaml[5-34, 38-55]
_schema-version: '3.1'
...
module:
  ...
- name: incident-management-destination-content
  type: com.sap.application.content
  requires:
  - name: incident-management-destination-service
    parameters:
      content-target: true
  - name: incident-management_html_repo_host
    parameters:
      service-key:
        name: incident-management_html_repo_host-key
  - name: incident-management-auth
    parameters:
      service-key:
        name: incident-management-auth-key
  parameters:
    content:
      instance:
        destinations:
        - Name: incidents_incident_management_html5_repo_host
          ServiceInstanceName: incident-management-html5-repo-host
          ServiceKeyName: incident-management-html5-repo-host-key
          sap.cloud.service: incidents
        - Authentication: OAuth2UserTokenExchange
          Name: incidents_incident_management_auth
          ServiceInstanceName: incident-management-auth
          ServiceKeyName: incident-management-auth-key
          sap.cloud.service: incidents
        existing_destinations_policy: ignore
  build-parameters:
    no-source: true
  ...
resources:
  ...
- name: incident-management-destination-service
  type: org.cloudfoundry.managed-service
  parameters:
    config:
      HTML5Runtime_enabled: true
      version: 1.0.0
    service: destination
    service-name: incident-management-destination-service
    service-plan: lite
parameters:
  deploy_mode: html5-repo
  enable-parallel-deployments: true
build-parameters:
  before-all:
  - builder: custom
    commands:
    - npm ci
    - npx cds build --production    
```

- This snippet adds the destinations required by the SAP Build Work Zone, standard edition service: HTML5 repo host service and Authorization and Trust Management (XSUAA) service.

> **Important**
>
> Double check if line 24 in the snippet above was added correctly. In case it is missing, open the **mta.yaml** file and add it. 

### Add navigation target for Incidents UI

Navigation targets are required to navigate between applications, but also to start applications from SAP Build Work Zone, standard edition. In this step, you add the navigation target **incidents-display** to the application manifest file **manifest.json**.

1. In the **Application Info - incidents** tab, choose the **Add Fiori Launchpad Config** tile.

    > To open the **Application Info - incidents** tab: 
    >
    >1. Invoke the Command Palette - **View** &rarr; **Command Palette** or <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> for macOS / <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> for Windows. 
    >2. Choose **Fiori: Open Application Info**.

    <!-- border; size:540px --> ![Fiori Launchpad Config](./fiori-launchpad-config1.png)

2. In the **Fiori Launchpad Configuration** step:

      - In the **Semantic Object** field, enter **incidents**.
      - In the **Action** field, enter **display**.
      - In the **Title** field, enter **Incident Management**.
      - Choose **Finish**.

      <!-- border; size:540px --> ![Fiori Launchpad Config](./fiori-launchpad-config2.png)

This navigation configuration adds the following section in **app/incidents/webapp/manifest.json**:


```json[10-23]
"sap.app": {
  "id": "ns.incidents",
  ...
  "sourceTemplate": {
    ...
  },
  "dataSources": {
    ...
  },
  "crossNavigation": {
    "inbounds": {
      "incidents-display": {
        "semanticObject": "incidents",
        "action": "display",
        "title": "{{flpTitle}}",
        "subTitle": "{{flpSubtitle}}",
        "signature": {
          "parameters": {},
          "additionalParameters": "allowed"
        }
      }
    }
  }
...
}

```

### Add the UI application

1. In the **Application Info - incidents** tab, choose the **Add Deploy Config** tile.

    <!-- border; size:540px --> ![V4 Template](./ui1.png)

    > To open the **Application Info - incidents** tab: 
    >
    >1. Invoke the Command Palette - **View** &rarr; **Command Palette** or <kbd>Command</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> for macOS / <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> for Windows. 
    >2. Choose **Fiori: Open Application Info**.

3. In the **Deploy Configuration** step:

      - In the **Please choose the target** dropdown menu, select **Cloud Foundry**.
      - In the **Destination name** dropdown menu, select **Local CAP Project API (Instance Based Destination)**.
      - In the **Editing the deployment configuration will overwrite existing configuration, are you sure you want to continue?** field, choose **Yes**.
      - Choose **Finish**.


      <!-- border; size:540px --> ![V4 Template](./ui2.png)


This deploy configuration adds SAP Cloud service at the end of **app/incidents/webapp/manifest.json**:

```json
 "sap.cloud": {
    "public": true,
    "service": "incidents"
  }
```

In addition, in the **mta.yaml** file, new modules have been generated and the resources have been updated:

```yaml[15-28, 36-51, 57]
_schema-version: '3.1'
...
module:
  ...
- name: incident-management-app-deployer
  type: com.sap.application.content
  path: .
  requires:
  - name: incident-management_html_repo_host
    parameters:
      content-target: true
  build-parameters:
    build-result: app/ # to be changed to resources/
    requires:
    - artifacts:
      - nsincidents.zip
      name: nsincidents
      target-path: app// # to be changed to resources/
- name: nsincidents
  type: html5
  path: app/incidents
  build-parameters:
    build-result: dist
    builder: custom
    commands:
    - npm install
    - npm run build:cf
    supported-platforms: []
resources:
... 
- name: incident-management-destination-service
  type: org.cloudfoundry.managed-service
  parameters:
    config:
      HTML5Runtime_enabled: true
      init_data:
        instance:
          destinations:
          - Authentication: NoAuthentication
            Name: ui5
            ProxyType: Internet
            Type: HTTP
            URL: https://ui5.sap.com
          - Authentication: NoAuthentication
            HTML5.DynamicDestination: true
            HTML5.ForwardAuthToken: true
            Name: incident-management-srv-api
            ProxyType: Internet
            Type: HTTP
            URL: ~{srv-api/srv-url}
          existing_destinations_policy: update
      version: 1.0.0
    service: destination
    service-name: incident-management-destination-service
    service-plan: lite
  requires:
  - name: srv-api
- name: incident-management_html_repo_host
  ...
parameters:
  ...  
```

> Make sure that your `mta.yaml` file does not include the following module:
>
> ```yaml
> - name: incidentmanagementincidents
>   type: html5
>   path: app/incidents
>   build-parameters:
>     build-result: dist
>     builder: custom
>     commands:
>     - npm ci
>     - npm run build
>     supported-platforms: []
> ``` 
> Delete if it exists because it duplicates the purpose of the `nsincidents` module.

> Make sure that your `mta.yaml` file does not include any references to the `incidentmanagementincidents` module: 
>
> ```yaml[10-14]
> - name: incident-management-app-deployer
>   type: com.sap.application.content
>   path: gen
>   requires:
>   - name: incident-management-html5-repo-host
>     parameters:
>       content-target: true
>   build-parameters:
>     build-result: app/
>     requires:
>     - artifacts:
>       - incidents.zip
>       name: incidentmanagementincidents
>       target-path: app/
>     - artifacts:
>       - nsincidents.zip
>       name: nsincidents
>       target-path: app//
> ``` 
> Delete if any exist.


### Update the `incident-management-app-deployer` module

1. Change the build result directory and the target path as follows:

    ```yaml[13, 18]
    _schema-version: '3.1'
    ...
    module:
      ...
    - name: incident-management-app-deployer
      type: com.sap.application.content
      path: .
      requires:
      - name: incident-management_html_repo_host
        parameters:
          content-target: true
      build-parameters:
        build-result: resources/
        requires:
        - artifacts:
          - nsincidents.zip
          name: nsincidents
          target-path: resources/
    - name: nsincidents
    ...
    ```

### Assemble with the Cloud MTA Build Tool

Run the following command to assemble everything into a single **mta.tar** archive:

```bash
mbt build
```

See [Multitarget Applications in the Cloud Foundry Environment](https://help.sap.com/products/BTP/65de2977205c403bbc107264b8eccf4b/d04fc0e2ad894545aebfd7126384307c.html?locale=en-US) to learn more about MTA-based deployment.

### Deploy in the SAP BTP, Cloud Foundry runtime

1. From the root of the **INCIDENT-MANAGEMENT** project, choose the burger menu, and then choose **Terminal** &rarr; **New Terminal**.

2. Log in to your subaccount in SAP BTP:

    ```bash
    cf api <API-ENDPOINT>
    cf login
    cf target -o <ORG> -s <SPACE>
    ```

    > You can find the API endpoint in the **Overview** section of your subaccount in the SAP BTP cockpit.

3. Run the following command to deploy the generated archive to the SAP BTP, Cloud Foundry runtime:

    ```bash
    cf deploy mta_archives/incident-management_1.0.0.mtar 
    ```

4. Check if all services have been created:

    ```bash 
    cf services
    ```

    You should see the following services in your space:

    <!-- border; size:540px --> ![Services after deploy](./cf-services.png)

5. Check if the apps are running:

    ```bash
    cf apps
    ```

    <!-- border; size:540px --> ![App after deploy](./cf-apps.png)

<!-- 6. Enter the route displayed for **incident-management-srv** in your browser. -->

<!-- border; size:540px ![Incident Management route](./incident-management-srv-route.png) -->

<!-- You see the CAP start page: -->

<!-- border; size:540px ![CAP start page](./cap-start-page.png) -->

<!-- 4. When you choose the **Incidents** service entity, you will see an error message.  -->

<!-- border; size:540px ![401 error](./401-error.png) -->

<!-- The service expects a so called JWT (JSON Web Token) in the HTTP Authorization header that contains the required authentication and authorization information to access the service.  -->

In the next tutorials, you will access your UIs from SAP Build Work Zone, standard edition. The SAP Build Work Zone, standard edition will trigger the authentication flow to provide the required token to access the service.
