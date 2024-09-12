---
title: Integrate Your Application with SAP Build Work Zone, Standard Edition
description: This tutorial shows you how to subscribe to the SAP Build Work Zone, standard edition.
keywords: cap 
parser: v2
auto_validation: true
time: 60
tags: [ tutorial>beginner, software-product-function>sap-cloud-application-programming-model, programming-tool>node-js, software-product>sap-business-technology-platform, software-product>sap-fiori]
primary_tag: software-product-function>sap-cloud-application-programming-model
author_name: Svetoslav Pandeliev
author_profile: https://github.com/slavipande
---

## You will learn

- How to subscribe to SAP Build Work Zone, standard edition
- How to assign the SAP Build Work Zone, standard edition role collection to your user
- How to integrate your application with SAP Build Work Zone, standard edition

## Prerequisites

- You have created a role collection and assigned it to a user. Follow the steps in the [Assign the User Roles](user-role-assignment) tutorial that is part of the [Deploy a Full-Stack CAP Application in SAP BTP, Cloud Foundry Runtime Following SAP BTP Developer’s Guide](https://developers.sap.com/group.deploy-full-stack-cap-application.html) tutorial group.
- You have an [enterprise global account](https://help.sap.com/docs/btp/sap-business-technology-platform/getting-global-account#loiod61c2819034b48e68145c45c36acba6e) in SAP BTP. To use services for free, you can sign up for an SAP BTPEA (SAP BTP Enterprise Agreement) or a Pay-As-You-Go for SAP BTP global account and make use of the free tier services only. See [Using Free Service Plans](https://help.sap.com/docs/btp/sap-business-technology-platform/using-free-service-plans?version=Cloud).
- You have a platform user. See [User and Member Management](https://help.sap.com/docs/btp/sap-business-technology-platform/user-and-member-management).
- You are an administrator of the global account in SAP BTP.
- You have a subaccount in SAP BTP to deploy the services and applications.
- You have one of the following browsers that are supported for working in SAP Business Application Studio:
    - Mozilla Firefox
    - Google Chrome
    - Microsoft Edge

### Subscribe to SAP Build Work Zone, standard edition

1. Navigate to your subaccount and choose **Services** &rarr; **Service Marketplace** on the left.

3. Search for the **SAP Build Work Zone, standard edition** tile and choose **Create**.

    <!-- border; size:540px --> ![Create SAP Build Work Zone, standard edition](./create_workzone_instance.png)

4. Keep the default setting for **Service** and choose **free** for **Plan**.

5. Choose **Create**.

    <!-- border; size:540px --> ![New Instance or Subscription](./new_instance_dialog.png)

You have now subscribed to the SAP Build Work Zone, standard edition.

### Assign SAP Build Work Zone, standard edition role collection

You need to assign your user to the **Launchpad_Admin** role collection, so you don't get an error accessing the **SAP Build Work Zone, standard edition** site later on.

1. Choose **Security** &rarr; **Users** on the left.

2. Choose your user.

4. Under **Role Collections** on the right, choose **Assign Role Collection** and assign the **Launchpad_Admin** role collection to your user.

    <!-- border; size:540px --> ![Add role](./add_launchpad_admin_role.png)

    You've assigned the **Launchpad_Admin** role collection to your user.

> You might need to log out and log back in to make sure your new role collection is taken into account.

> See section [Initial Setup](https://help.sap.com/viewer/8c8e1958338140699bd4811b37b82ece/Cloud/en-US/fd79b232967545569d1ae4d8f691016b.html) in the SAP Build Work Zone, standard edition's documentation for more details.


###  Integrate your application with SAP Build Work Zone, standard edition

> There are two different ways to integrate your application with SAP Build Work Zone, standard edition. You have to choose one way or the other and follow through. It's currently not possible to mix the approaches. Both ways provide the same result. You can integrate your application with SAP Build Work Zone, standard edition through Common Data Model or through the SAP BTP cockpit. The tabs **Common Data Model** and **SAP BTP Cockpit** provide detailed steps for each alternative way.  

[OPTION BEGIN [Common Data Model]]

The Common Data Model (CDM) is the basis for interoperability and content federation of business content from various SAP and third-party products, all integrated in a unified manner in one site that serves as a central entry point. Business content items such as roles, catalogs, groups, spaces, and pages can be included in the CDM. For this purpose, developers must specify the business content of the site in a **cdm.json** file that is part of the multitarget application (MTA) and also define design-time and runtime destinations in the **mta.yaml** file. For more information, see [Developing HTML5 Business Solutions as Content Providers](https://help.sap.com/docs/build-work-zone-advanced-edition/sap-build-work-zone-advanced-edition/developing-html5-apps-for-cross-subaccount-consumption) and [About the Common Data Model](https://help.sap.com/docs/build-work-zone-advanced-edition/sap-build-work-zone-advanced-edition/creating-cdm-json-file-for-multi-tenancy-html5-app).

#### Update the mta.yaml file

1. Add `html5-repo-runtime` under resources:

    ```yaml[3-8]
       resources:
        ...
        - name: incidents_html_repo_runtime
          type: org.cloudfoundry.managed-service
          parameters:
            service: html5-apps-repo
            service-name: incidents-html5-app-runtime-service
            service-plan: app-runtime
    ```

2. Update the `incident-management-destination-content` module:

    1. Add the `incidents-html-repo-runtime` to the `requires` section:

        ```yaml[4-7]
        - name: incident-management-destination-content
          type: com.sap.application.content
          requires:
          - name: incidents_html_repo_runtime
            parameters:
              service-key:
                name: incidents-html5-app-runtime-service-key
        ```

    2. Locate the entry `instance` under `parameters.content` and change it to `subaccount`.

        ```yaml[7]
        - name: incident-management-destination-content
          type: com.sap.application.content
          requires:
          ...
          parameters:
            content:
                subaccount:
                    destinations:
                    - Name: incidents_incident_management_html_repo_host
                    ...
        ```

    3. Replace existing destinations with the following code snippet:

        ```yaml[9-12]
        - name: incident-management-destination-content
          type: com.sap.application.content
          requires:
          ...
          parameters:
            content:
                subaccount:
                    destinations:
                    - Name: incident-management_cdm
                      ServiceInstanceName: incidents-html5-app-runtime-service
                      ServiceKeyName: incidents-html5-app-runtime-service-key
                      URL: https://html5-apps-repo-rt.${default-domain}/applications/cdm/<cloud-service-name>
        ```
    
    > You can find the value for `<cloud-service-name>` in the **app/incidents/webapp/manifest.json** file.
    > ```json[3]
    > "sap.cloud": {
    >    "public": true,
    >    "service": "incidents"
    >}
    >```
    

3. Update the `incident-management-app-deployer` module:

    
    1. Add the following parameters:

        ```yaml[3-11]
        - name: incident-management-app-deployer
          type: com.sap.application.content
          path: .
          parameters:
            config:
                destinations:
                - forwardAuthToken: true
                  name: incident-management-srv-api
                  url: ~{srv-api/srv-url}
                - name: ui5
                  url: https://ui5.sap.com
        ```

    2. Under the `requires` section, add the following dependencies:

        ```yaml[2-3]
        requires:
        - name: srv-api
        - name: incident-management-auth
        ```
    


    <!--  - the name of the destination inserted here should be the same as in the **xsapp.json** file `destination` parameter in the `routes` section. -->

4. Update the `incident-management-destination-service` resource:

    1. Delete `HTML5Runtime_enabled: true` under `parameters`.

    2. Change `instance` to `subaccount`.

        ```[6]
        - name: incident-management-destination-service
          type: org.cloudfoundry.managed-service
          parameters:
            config:
              init_data:
                subaccount:
                  destinations:
                  ...
        ```

    3. Update destinations with the following:

        ```yaml[3-8]
        subaccount:
          destinations:
          - Authentication: NoAuthentication
            Name: incident-management-rt
            ProxyType: Internet
            Type: HTTP
            URL: https://<subdomain>.launchpad.${default-domain}
            CEP.HTML5contentprovider: true
        ```

        > Replace `<subdomain>` with your SAP BTP account's subdomain. You can find the subdomain in the **Overview** page in SAP BTP Cockpit. 

    4. Delete the `requires` section and its content:

        ```yaml
        requires:
          - name: srv-api
        ```

5. Update the build parameters:

    ```yaml[7-8]
    build-parameters:
    before-all:
      - builder: custom
      commands:
        - npm ci
        - npx cds build --production 
        - mkdir -p resources
        - cp workzone/cdm.json resources/cdm.json  
    ```

6. Verify. Here's how your **mta.yaml** file should look like at this stage:

    ```yaml
    _schema-version: 3.3.0
    ID: incident-management
    description: A simple CAP project.
    version: 1.0.0
    modules:
    - name: incident-management-srv
      type: nodejs
      path: gen/srv
      requires:
      - name: incident-management-db
      - name: incident-management-auth
      provides:
      - name: srv-api
        properties:
          srv-url: ${default-url}
      parameters:
        buildpack: nodejs_buildpack
        readiness-health-check-http-endpoint: /health
        readiness-health-check-type: http
      build-parameters:
        builder: npm
    - name: incident-management-db-deployer
      type: hdb
      path: gen/db
      requires:
      - name: incident-management-db
      parameters:
        buildpack: nodejs_buildpack
    - name: incident-management-app-deployer
      type: com.sap.application.content
      path: .
      parameters:
        config:
            destinations:
            - forwardAuthToken: true
              name: incident-management-srv-api
              url: ~{srv-api/srv-url}
            - name: ui5
              url: https://ui5.sap.com
      requires:
      - name: srv-api
      - name: incident-management-auth
      - name: incident-management-html5-repo-host
        parameters:
          content-target: true
      build-parameters:
        build-result: resources/
        requires:
        - artifacts:
          - nsincidents.zip
          name: nsincidents
          target-path: resources/
    - name: incident-management-destination-content
      type: com.sap.application.content
      requires:
      - name: incident-management-destination-service
        parameters:
          content-target: true
      - name: incidents_html_repo_runtime
        parameters:
          service-key:
            name: incidents-html5-app-runtime-service-key
      - name: incident-management-html5-repo-host
        parameters:
          service-key:
            name: incident-management-html5-repo-host-key
      - name: incident-management-auth
        parameters:
          service-key:
            name: incident-management-auth-key
      parameters:
        content:
          subaccount:
            destinations:
            - Name: incident-management_cdm
              ServiceInstanceName: incidents-html5-app-runtime-service
              ServiceKeyName: incidents-html5-app-runtime-service-key
              URL: https://html5-apps-repo-rt.${default-domain}/applications/cdm/incidents
            existing_destinations_policy: ignore
      build-parameters:
        no-source: true
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
        config:
          tenant-mode: dedicated
          xsappname: incident-management-${org}-${space}
        path: ./xs-security.json
        service: xsuaa
        service-plan: application
    - name: incidents_html_repo_runtime
      type: org.cloudfoundry.managed-service
      parameters:
        service: html5-apps-repo
        service-name: incidents-html5-app-runtime-service
        service-plan: app-runtime
    - name: incident-management-destination-service
      type: org.cloudfoundry.managed-service
      parameters:
        config:
          init_data:
            subaccount:
              destinations:
              - Authentication: NoAuthentication
                Name: incident-management-rt
                ProxyType: Internet
                Type: HTTP
                URL: https://sveto-test-eu12-kanmyy65.launchpad.${default-domain}
                CEP.HTML5contentprovider: true
              existing_destinations_policy: update
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
        - mkdir -p resources
        - cp workzone/cdm.json resources/cdm.json  
    ```

#### Create the CDM configuration

1. Create a new folder **workzone** in the root of your project.

2. Create file **cdm.json** in the **workzone** folder and paste the following code snippet in the file:

    ```json
    [
        {
        "_version": "3.0",
        "identification": {
            "id": "defaultCatalogId",
            "title": "{{title}}",
            "entityType": "catalog"
        },
        "payload": {
            "viz": [
            {
                "appId": "ns.incidents",
                "vizId": "incidents-display"
            }
            ]
        },
        "texts": [
            {
            "locale": "",
            "textDictionary": {
                "title": "Default Catalog Title"
            }
            }
        ]
        },
        {
        "_version": "3.0",
        "identification": {
            "id": "defaultGroupId",
            "title": "{{title}}",
            "entityType": "group"
        },
        "payload": {
            "viz": [
            {
                "appId": "ns.incidents",
                "vizId": "incidents-display"
            }
            ]
        },
        "texts": [
            {
            "locale": "",
            "textDictionary": {
                "title": "Business Apps"
            }
            }
        ]
        },
        {
        "_version": "3.0",
        "identification": {
            "id": "defaultRole",
            "entityType": "role",
            "title": "Default Role"
        },
        "payload": {
            "apps": [
            {
                "id": "ns.incidents"
            }
            ],
            "catalogs": [
            {
                "id": "defaultCatalogId"
            }
            ],
            "groups": [
            {
                "id": "defaultGroupId"
            }
            ]
        }
        }
    ]
    ```

#### Deploy your application in SAP BTP, Cloud Foundry runtime

1. Open a terminal in your project's root folder and run the following commands:

    ```bash
    mbt build
    cf deploy mta_archives/incident-management_1.0.0.mtar 
    ```

#### Create a new content provider

1. Open your subaccount and navigate to **Instances and Subscriptions**.

2. Choose the application **SAP Build Work Zone, standard edition**.

    <!-- border; size:540px --> ![WorkZone0](./integrate_launchpad_0.png)

3. In the menu on the left side, choose the icon for **Channel Manager**.

4. Choose **New** &rarr; **Content Provider** on the right.

    <!-- border; size:540px --> ![New Content Provider](./new-content-provider.png)

5. In the **New Content Provider** dialog:

    1. Enter **CDM** in the **Title** field.

    2. Choose **incident-management_cdm** from the dropdown in the **Design-Time Destination** field.

    3. Choose **incident-management-rt** from the dropdown in the **Runtime Destination** field.

    4. Choose **Save**.


     <!-- border; size:540px --> ![New Content Provider dialog](./new-content-provider-dialog.png)


6. Choose the refresh icon to fetch the updated content from the **CDM** content provider. 

    <!-- border; size:540px --> ![New Content Provider fetch](./new-content-provider-fetch.png)

#### Create site

15. Navigate to **Site Directory** and choose **Create Site**.

    <!-- border; size:540px --> ![WorkZone7](./integrate_launchpad_7.png)

17. Enter the site name as **Incident Management Site** and choose **Create**.

    <!-- border; size:540px --> ![WorkZone7](./integrate_launchpad_71.png)

18. When you are forwarded to your created site, choose **Edit**.

    <!-- border; size:540px --> ![WorkZone7](./integrate_launchpad_72.png)

19. Choose the **Assign Items** field, add **+** the **Default Role** to your site and choose **Save**.

    <!-- border; size:540px --> ![WorkZone7](./integrate_launchpad_73.png)

#### Assign CDM role collection

You need to assign your user to the **~cdm_defaultRole** role collection, so you are able to see the application tile when accessing the **SAP Build Work Zone, standard edition** site later on.

1. Open your subaccount and navigate to **Security** &rarr; **Users**.

2. Choose your user.

4. Under **Role Collections** on the right, choose **Assign Role Collection** and assign the **~cdm_defaultRole** role collection to your user.

    <!-- border; size:540px --> ![Add CDM role](./add_cdm_role.png)

    You've assigned the **~cdm_defaultRole** role collection to your user.

> You might need to log out and log in again to make sure your new role collection is taken into account.

[OPTION END]

[OPTION BEGIN [SAP BTP Cockpit]]

#### Update content

1. Open your subaccount and navigate to **Instances and Subscriptions**.

2. Choose the application **SAP Build Work Zone, standard edition**.

    <!-- border; size:540px --> ![WorkZone0](./integrate_launchpad_0.png)

3. In the menu on the left side, choose the icon for **Channel Manager**.

4. Choose the refresh icon to fetch the updated content.

    <!-- border; size:540px --> ![WorkZone1](./integrate_launchpad_1.png)

#### Add application to Content Explorer

5. Choose **Content Manager** in the menu on the left and choose the **Content Explorer** button.

    <!-- border; size:540px --> ![WorkZone2](./integrate_launchpad_20.png)

6. Select the tile **HTML5 Apps** with your respective subdomain name.

    <!-- border; size:540px --> ![WorkZone2](./integrate_launchpad_2.png)

7. In the items table, set checkmark for the app **Incidents** and choose the button **Add**.

    <!-- border; size:540px --> ![WorkZone3](./integrate_launchpad_3.png)

#### Create a group

8. Go back to the **Content Manager**, choose **Create** &rarr; **Group**.

    <!-- border; size:540px --> ![WorkZone4](./integrate_launchpad_4.png)

9. Add the title **Incident Management Group**.

10. In the **Assignment Status**, choose the slider to assign the **Incident-Management** app to the group.

11. Choose **Save**.

    <!-- border; size:540px --> ![WorkZone5](./integrate_launchpad_5.png)

#### Add application to the Everyone role

12. Back in the **Content Manager**, choose role **Everyone** and choose **Edit**.

13. In the **Assignment Status**, choose the slider to assign the **Incident-Management** app to the role.

14. Choose **Save**.

    <!-- border; size:540px --> ![WorkZone6](./integrate_launchpad_6.png)

#### Create site

15. Navigate to **Site Directory** and choose **Create Site**.

    <!-- border; size:540px --> ![WorkZone7](./integrate_launchpad_7.png)

17. Enter the site name as **Incident Management Site** and choose **Create**.

    <!-- border; size:540px --> ![WorkZone7](./integrate_launchpad_71.png)

18. Now, you are forwarded to your created site.

[OPTION END]

### Test your site

19. Navigate to **Site Directory** and find your site.

    <!-- border; size:540px --> ![WorkZone8](./integrate_launchpad_8.png)

21. Choose **Go to site**. 

    <!-- border; size:540px --> ![WorkZone9](./integrate_launchpad_9.png)

3. Choose the Incident Management application from the launch page. 

    <!-- border; size:540px --> ![WorkZone10](./integrate_launchpad_10.png)

    You should be able to see the list report page.

    <!-- border; size:540px --> ![WorkZone11](./integrate_launchpad_11.png)

### Summary

Congratulations! You now have finished the development of your application and you have integrated SAP Build Work Zone, standard edition to have one central entry point to show all of your SAP BTP applications.
