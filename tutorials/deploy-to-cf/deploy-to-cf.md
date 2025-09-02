---
title: Deploy in SAP BTP, Cloud Foundry Runtime
description: This tutorial shows you how to deploy your CAP application as a multi-target application (MTA).
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

- How to deploy your CAP application as a multi-target application (MTA)

## Prerequisites

- You've configured the respective entitlements, enabled the Cloud Foundry runtime in your subaccount, and created an SAP HANA Cloud service instance in the SAP BTP cockpit. Follow the steps in the [Prepare for Deployment in the SAP BTP, Cloud Foundry Runtime](prepare-btp-cf) tutorial that is part of the [Deploy a Full-Stack CAP Application in SAP BTP, Cloud Foundry Runtime Following SAP BTP Developer’s Guide](https://developers.sap.com/group.deploy-full-stack-cap-application.html) tutorial group.
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

### Introduction

The SAP BTP, Cloud Foundry environment allows you to create polyglot cloud applications in Cloud Foundry. It contains the SAP BTP, Cloud Foundry runtime, which is based on the open-source application platform managed by the Cloud Foundry Foundation.

The SAP BTP, Cloud Foundry environment enables you to develop new business applications and business services, supporting multiple runtimes, programming languages, libraries, and services.

For more information about the Cloud Foundry environment, see [Cloud Foundry Environment](https://help.sap.com/docs/btp/sap-business-technology-platform/cloud-foundry-environment).

### Set up the MTA for deployment

A multi-target application (MTA) is logically a single application comprised of multiple parts created with different technologies, which share the same lifecycle.

The developers of the MTA describe the desired result using the MTA model which contains MTA modules, MTA resources, and interdependencies between them. Afterwards, the MTA deployment service validates, orchestrates, and automates the deployment of the MTA, which results in Cloud Foundry (CF) applications, services, and SAP-specific contents.

You use the [Cloud MTA Build Tool](https://sap.github.io/cloud-mta-build-tool/) to deploy the Incident Management application. The modules and services are configured in the **mta.yaml** deployment descriptor file.

1. In SAP Business Application Studio, go to your **IncidentManagement** dev space.

    > Make sure the **IncidentManagement** dev space is in status **RUNNING**.

2. From the root of the **INCIDENT-MANAGEMENT** project, choose the burger menu, and then choose **Terminal** &rarr; **New Terminal**.

3. Run the following command to generate the **mta.yaml** deployment descriptor:

    ```bash
    cds add mta
    ```

### Check configuration for SAP Build Work Zone, standard edition

> You can create a CAP project in either Node.js or Java. You have to choose one way or the other and follow through. The tabs **Node.js** and **Java** provide detailed steps for each alternative way.

[OPTION BEGIN [Node.js]]

2. Verify that the destinations module `incident-management-destinations` and resource `incident-management-destination` have been added to the **mta.yaml** file without errors:


    ```yaml
    _schema-version: '3.1'
    ...
    module:
      ...
    - name: incident-management-destinations
      type: com.sap.application.content
      requires:
        - name: incident-management-auth
          parameters:
            service-key:
              name: incident-management-auth-key
        - name: incident-management-html5-repo-host
          parameters:
            service-key:
              name: incident-management-html5-repo-host-key
        - name: srv-api
        - name: incident-management-destination
          parameters:
            content-target: true
      build-parameters:
        no-source: true
      parameters:
        content:
          instance:
            existing_destinations_policy: update
            destinations:
              - Name: incident-management-html5-repository
                ServiceInstanceName: incident-management-html5-repo-host
                ServiceKeyName: incident-management-html5-repo-host-key
                sap.cloud.service: incidentmanagement.service
              - Name: incident-management-auth
                Authentication: OAuth2UserTokenExchange
                ServiceInstanceName: incident-management-auth
                ServiceKeyName: incident-management-auth-key
                sap.cloud.service: incidentmanagement.service
        ...
    resources:
      ...
    - name: incident-management-destination
      type: org.cloudfoundry.managed-service
      parameters:
        service: destination
        service-plan: lite
        config:
          HTML5Runtime_enabled: true
          init_data:
            instance:
              existing_destinations_policy: update
              destinations:
                - Name: incident-management-srv-api
                  URL: ~{srv-api/srv-url}
                  Authentication: NoAuthentication
                  Type: HTTP
                  ProxyType: Internet
                  HTML5.ForwardAuthToken: true
                  HTML5.DynamicDestination: true
                - Name: ui5
                  URL: https://ui5.sap.com
                  Authentication: NoAuthentication
                  Type: HTTP
                  ProxyType: Internet
      requires:
        - name: srv-api
          group: destinations
          properties:
            name: srv-api # must be used in xs-app.json as well
            url: ~{srv-url}
            forwardAuthToken: true
    ```

[OPTION END]

[OPTION BEGIN [Java]]

2. Verify that all required modules (`incident-management-app-deployer`, `incidentmanagementincidents`, and `incident-management-destinations`) and resources (`incident-management-destination` and `incident-management-html5-repo-host`) have been added to the **mta.yaml** file without errors:

    ```yaml
    _schema-version: '3.1'
    ...
    module:
      ...
    - name: incident-management-app-deployer
      type: com.sap.application.content
      path: .
      requires:
        - name: incident-management-html5-repo-host
          parameters:
            content-target: true
      build-parameters:
        build-result: resources
        requires:
          - name: incidentmanagementincidents
            artifacts:
              - incidents.zip
            target-path: resources

    - name: incidentmanagementincidents
      type: html5
      path: app/incidents
      build-parameters:
        build-result: dist
        builder: custom
        commands:
          - npm ci
          - npm run build
        supported-platforms:
          []

    - name: incident-management-destinations
      type: com.sap.application.content
      requires:
        - name: incident-management-auth
          parameters:
            service-key:
              name: incident-management-auth-key
        - name: incident-management-html5-repo-host
          parameters:
            service-key:
              name: incident-management-html5-repo-host-key
        - name: srv-api
        - name: incident-management-destination
          parameters:
            content-target: true
      build-parameters:
        no-source: true
      parameters:
        content:
          instance:
            existing_destinations_policy: update
            destinations:
              - Name: incident-management-html5-repository
                ServiceInstanceName: incident-management-html5-repo-host
                ServiceKeyName: incident-management-html5-repo-host-key
                sap.cloud.service: incidentmanagement.service
              - Name: incident-management-auth
                Authentication: OAuth2UserTokenExchange
                ServiceInstanceName: incident-management-auth
                ServiceKeyName: incident-management-auth-key
                sap.cloud.service: incidentmanagement.service
        ...
    resources:
      ...
    - name: incident-management-destination
      type: org.cloudfoundry.managed-service
      parameters:
        service: destination
        service-plan: lite
        config:
          HTML5Runtime_enabled: true
          init_data:
            instance:
              existing_destinations_policy: update
              destinations:
                - Name: incident-management-srv-api
                  URL: ~{srv-api/srv-url}
                  Authentication: NoAuthentication
                  Type: HTTP
                  ProxyType: Internet
                  HTML5.ForwardAuthToken: true
                  HTML5.DynamicDestination: true
                - Name: ui5
                  URL: https://ui5.sap.com
                  Authentication: NoAuthentication
                  Type: HTTP
                  ProxyType: Internet
      requires:
        - name: srv-api
          group: destinations
          properties:
            name: srv-api # must be used in xs-app.json as well
            url: ~{srv-url}
            forwardAuthToken: true
    - name: incident-management-html5-repo-host
      type: org.cloudfoundry.managed-service
      parameters:
        service: html5-apps-repo
        service-plan: app-host
    ```

[OPTION END]

### Make additional changes to the mta.yaml file

[OPTION BEGIN [Node.js]]

1. Update the `incident-management-app-deployer` module (`path`, `build-result`, and `target-path` parameters) as follows:

    ```yaml
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
          - name: incidentmanagementincidents
            artifacts:
              - incidents.zip
            target-path: resources/
            
    - name: incidentmanagementincidents
    ...
    ```

3. Verify the **mta.yaml** file before deployment.

    At this stage, your **mta.yaml** file looks like the following code snippet:

    ```yaml
    _schema-version: 3.3.0
    ID: incident-management
    version: 1.0.0
    description: "A simple CAP project."
    parameters:
      enable-parallel-deployments: true
      deploy_mode: html5-repo
    build-parameters:
      before-all:
        - builder: custom
          commands:
            - npm ci
            - npx cds build --production  
    modules:
      - name: incident-management-srv
        type: nodejs
        path: gen/srv
        parameters:
          buildpack: nodejs_buildpack
          readiness-health-check-type: http
          readiness-health-check-http-endpoint: /health
        build-parameters:
          builder: npm
        provides:
          - name: srv-api # required by consumers of CAP services (e.g. approuter)
            properties:
              srv-url: ${default-url}
        requires:
          - name: incident-management-db
          - name: incident-management-auth
          - name: incident-management-destination

      - name: incident-management-db-deployer
        type: hdb
        path: gen/db
        parameters:
          buildpack: nodejs_buildpack
        requires:
          - name: incident-management-db

      - name: incident-management-app-deployer
        type: com.sap.application.content
        path: .
        requires:
          - name: incident-management-html5-repo-host
            parameters:
              content-target: true
        build-parameters:
          build-result: resources/
          requires:
            - name: incidentmanagementincidents
              artifacts:
                - incidents.zip
              target-path: resources/

      - name: incidentmanagementincidents
        type: html5
        path: app/incidents
        build-parameters:
          build-result: dist
          builder: custom
          commands:
            - npm ci
            - npm run build
          supported-platforms:
            []

      - name: incident-management-destinations
        type: com.sap.application.content
        requires:
          - name: incident-management-auth
            parameters:
              service-key:
                name: incident-management-auth-key
          - name: incident-management-html5-repo-host
            parameters:
              service-key:
                name: incident-management-html5-repo-host-key
          - name: srv-api
          - name: incident-management-destination
            parameters:
              content-target: true
        build-parameters:
          no-source: true
        parameters:
          content:
            instance:
              existing_destinations_policy: update
              destinations:
                - Name: incident-management-html5-repository
                  ServiceInstanceName: incident-management-html5-repo-host
                  ServiceKeyName: incident-management-html5-repo-host-key
                  sap.cloud.service: incidentmanagement.service
                - Name: incident-management-auth
                  Authentication: OAuth2UserTokenExchange
                  ServiceInstanceName: incident-management-auth
                  ServiceKeyName: incident-management-auth-key
                  sap.cloud.service: incidentmanagement.service

    resources:
      - name: incident-management-db
        type: com.sap.xs.hdi-container
        parameters:
          service: hana
          service-plan: hdi-shared
      - name: incident-management-html5-repo-host
        type: org.cloudfoundry.managed-service
        parameters:
          service: html5-apps-repo
          service-plan: app-host
      - name: incident-management-auth
        type: org.cloudfoundry.managed-service
        parameters:
          service: xsuaa
          service-plan: application
          path: ./xs-security.json
          config:
            xsappname: incident-management-${org}-${space}
            tenant-mode: dedicated
      - name: incident-management-destination
        type: org.cloudfoundry.managed-service
        parameters:
          service: destination
          service-plan: lite
          config:
            HTML5Runtime_enabled: true
            init_data:
              instance:
                existing_destinations_policy: update
                destinations:
                  - Name: incident-management-srv-api
                    URL: ~{srv-api/srv-url}
                    Authentication: NoAuthentication
                    Type: HTTP
                    ProxyType: Internet
                    HTML5.ForwardAuthToken: true
                    HTML5.DynamicDestination: true
                  - Name: ui5
                    URL: https://ui5.sap.com
                    Authentication: NoAuthentication
                    Type: HTTP
                    ProxyType: Internet
        requires:
          - name: srv-api
            group: destinations
            properties:
              name: srv-api # must be used in xs-app.json as well
              url: ~{srv-url}
              forwardAuthToken: true
    
    ```

[OPTION END]

[OPTION BEGIN [Java]]

3. Verify the **mta.yaml** file before deployment.


    At this stage, your **mta.yaml** file looks like the following code snippet:

    ```yaml
    _schema-version: 3.3.0
    ID: incident-management
    version: 1.0.0-SNAPSHOT
    description: "A simple CAP project."
    parameters:
      enable-parallel-deployments: true
      deploy_mode: html5-repo
    modules:
      - name: incident-management-srv
        type: java
        path: srv
        parameters:
          buildpack: sap_java_buildpack_jakarta
          readiness-health-check-type: http
          readiness-health-check-http-endpoint: /actuator/health/readiness
        properties:
          SPRING_PROFILES_ACTIVE: cloud,sandbox
          JBP_CONFIG_COMPONENTS: "jres: ['com.sap.xs.java.buildpack.jre.SAPMachineJRE']"
          JBP_CONFIG_SAP_MACHINE_JRE: '{ version: 21.+ }'
        build-parameters:
          builder: custom
          commands:
            - mvn clean package -DskipTests=true --batch-mode
          build-result: target/*-exec.jar
        provides:
          - name: srv-api # required by consumers of CAP services (e.g. approuter)
            properties:
              srv-url: ${default-url}
        requires:
          - name: incident-management-db
          - name: incident-management-auth
          - name: incident-management-destination

      - name: incident-management-db-deployer
        type: hdb
        path: db
        parameters:
          buildpack: nodejs_buildpack
        build-parameters:
          builder: custom
          commands:
            - npm run build
        requires:
          - name: incident-management-db

      - name: incident-management-app-deployer
        type: com.sap.application.content
        path: .
        requires:
          - name: incident-management-html5-repo-host
            parameters:
              content-target: true
        build-parameters:
          build-result: resources
          requires:
            - name: incidentmanagementincidents
              artifacts:
                - incidents.zip
              target-path: resources

      - name: incidentmanagementincidents
        type: html5
        path: app/incidents
        build-parameters:
          build-result: dist
          builder: custom
          commands:
            - npm ci
            - npm run build
          supported-platforms:
            []

      - name: incident-management-destinations
        type: com.sap.application.content
        requires:
          - name: incident-management-auth
            parameters:
              service-key:
                name: incident-management-auth-key
          - name: incident-management-html5-repo-host
            parameters:
              service-key:
                name: incident-management-html5-repo-host-key
          - name: srv-api
          - name: incident-management-destination
            parameters:
              content-target: true
        build-parameters:
          no-source: true
        parameters:
          content:
            instance:
              existing_destinations_policy: update
              destinations:
                - Name: incident-management-html5-repository
                  ServiceInstanceName: incident-management-html5-repo-host
                  ServiceKeyName: incident-management-html5-repo-host-key
                  sap.cloud.service: incidentmanagement.service
                - Name: incident-management-auth
                  Authentication: OAuth2UserTokenExchange
                  ServiceInstanceName: incident-management-auth
                  ServiceKeyName: incident-management-auth-key
                  sap.cloud.service: incidentmanagement.service

    resources:
      - name: incident-management-db
        type: com.sap.xs.hdi-container
        parameters:
          service: hana
          service-plan: hdi-shared
      - name: incident-management-auth
        type: org.cloudfoundry.managed-service
        parameters:
          service: xsuaa
          service-plan: application
          path: ./xs-security.json
          config:
            xsappname: incident-management-${org}-${space}
            tenant-mode: dedicated
      - name: incident-management-destination
        type: org.cloudfoundry.managed-service
        parameters:
          service: destination
          service-plan: lite
          config:
            HTML5Runtime_enabled: true
            init_data:
              instance:
                existing_destinations_policy: update
                destinations:
                  - Name: incident-management-srv-api
                    URL: ~{srv-api/srv-url}
                    Authentication: NoAuthentication
                    Type: HTTP
                    ProxyType: Internet
                    HTML5.ForwardAuthToken: true
                    HTML5.DynamicDestination: true
                  - Name: ui5
                    URL: https://ui5.sap.com
                    Authentication: NoAuthentication
                    Type: HTTP
                    ProxyType: Internet
        requires:
          - name: srv-api
            group: destinations
            properties:
              name: srv-api # must be used in xs-app.json as well
              url: ~{srv-url}
              forwardAuthToken: true
      - name: incident-management-html5-repo-host
        type: org.cloudfoundry.managed-service
        parameters:
          service: html5-apps-repo
          service-plan: app-host

    
    ```

[OPTION END]

### Assemble with the Cloud MTA Build Tool

Run the following commands to assemble everything into a single **mta.tar** archive:

```bash
npm install
mbt build
```

See [Multitarget Applications in the Cloud Foundry Environment](https://help.sap.com/products/BTP/65de2977205c403bbc107264b8eccf4b/d04fc0e2ad894545aebfd7126384307c.html?locale=en-US) to learn more about MTA-based deployment.

### Deploy in the SAP BTP, Cloud Foundry runtime

[OPTION BEGIN [Node.js]]

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

    <!-- border; size:540px --> ![Services after deploy](./cf-services.png)

5. Check if the apps are running:

    ```bash
    cf apps
    ```

    <!-- border; size:540px --> ![App after deploy](./cf-apps.png)

<!-- 6. Enter the route displayed for **incident-management-srv** in your browser. -->

<!-- border; size:540px ![Incident Management route](./incident-management-srv-route.png) -->

<!-- undeploy command: `cf undeploy incident-management --delete-service-keys --delete-services` -->

<!-- You see the CAP start page: -->

<!-- border; size:540px ![CAP start page](./cap-start-page.png) -->

<!-- 4. When you choose the **Incidents** service entity, you will see an error message.  -->

<!-- border; size:540px ![401 error](./401-error.png) -->

<!-- The service expects a so called JWT (JSON Web Token) in the HTTP Authorization header that contains the required authentication and authorization information to access the service.  -->

[OPTION END]

[OPTION BEGIN [Java]]

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
    cf deploy mta_archives/incident-management_1.0.0-SNAPSHOT.mtar 
    ```

4. Check if all services have been created:

    ```bash
    cf services
    ```

    <!-- border; size:540px --> ![Services after deploy](./cf-services.jpg)

5. Check if the apps are running:

    ```bash
    cf apps
    ```

    <!-- border; size:540px --> ![App after deploy](./cf-apps.png)


[OPTION END]

In the next tutorials, you'll access your UIs from SAP Build Work Zone, standard edition. The SAP Build Work Zone, standard edition triggers the authentication flow to provide the required token to access the service.
