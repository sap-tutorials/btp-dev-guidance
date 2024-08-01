---
title: Build a CAP Application
description: This guide is a step-by-step walkthrough to build a CAP Node.js application. 
parser: v2
auto_validation: true
time: 30
tags: [ tutorial>beginner, programming-tool>sapui5, software-product>sap-business-technology-platform, software-product>sap-fiori, software-product-function>sap-cloud-application-programming-model]
primary_tag: software-product-function>sap-cloud-application-programming-model
author_name: Svetoslav Pandeliev
author_profile: https://github.com/slavipande
---

## You will learn

- How to create a CAP project
- How to add a domain model
- How to create services
- How to add data to your database
- How to add calculated elements

## Prerequisites

You have the SAP Business Application Studio configured. See [Set Up SAP Business Application Studio for Development](set-up-bas).

### Create a CAP project

1. In SAP Business Application Studio, go to your **IncidentManagement** dev space.

    > Make sure the **IncidentManagement** dev space is in status **RUNNING**.

2. Choose the burger menu and choose **Terminal** &rarr; **New Terminal**.

    <!-- border; size:540px --> ![Terminal](./bas-terminal.png)

3. Navigate to the **projects** folder from the root directory.

    ```bash
    cd projects
    ```

    > Your main interaction will be through the `cds` command. It is used to build, run, and debug your CAP application. Besides that, you can also use `cds` to enrich your application with additional capabilities such as messaging, multitenancy, or cloud integration. You can learn more about the tool's capabilities in [Command Line Interface (CLI)](https://cap.cloud.sap/docs/tools/#command-line-interface-cli).

4. Create a new project using [`cds init`](https://cap.cloud.sap/docs/tools/#cds-init-add).

    ```bash
    cds init incident-management
    ```

    > This command creates a folder **incident-management** with your newly created CAP project.

5. Choose the **Explorer** icon and then choose **Open Folder**. Type `/home/user/projects/` in the field and select `incident-management`. Choose **OK** to open the project in SAP Business Application Studio.

    <!-- border; size:540px --> ![Open SAP Business Application Studio folder](./bas-open-folder.png)

    > You can also use the **Ctrl+Shift+E** key combination to quickly navigate to the **Explorer**.
  
6. While you are in the **incident-management** folder, choose the burger menu and then choose **Terminal** &rarr; **New Terminal**.

    Now you can use the terminal to start a CAP server.

    ```bash
    cds watch
    ```

    The CAP server serves all the CAP sources from your project. It also "watches" all the files in your projects and conveniently restarts whenever you save a file. Changes you've made are immediately served without you having to run the command again. In this newly created project the CAP server tells you that there are no models or service definitions yet that it can serve.

    This is the output you should get:

    ```
    cds serve all --with-mocks --in-memory?
    live reload enabled for browsers

    ___________________________

    No models found in db/,srv/,app/,schema,services.
    Waiting for some to arrive...
    ```

### Add a domain model

1. In the **db** folder, create a new **schema.cds** file.

2. Paste the following code snippet in the **schema.cds** file.

    ```js
    using { User, cuid, managed, sap.common.CodeList } from '@sap/cds/common';
    namespace sap.capire.incidents; 

    /**
    * Incidents created by Customers.
    */
    entity Incidents : cuid, managed {  
    customer     : Association to Customers;
    title        : String  @title : 'Title';
    urgency        : Association to Urgency default 'M';
    status         : Association to Status default 'N';
    conversation  : Composition of many {
        key ID    : UUID;
        timestamp : type of managed:createdAt;
        author    : type of managed:createdBy;
        message   : String;
    };
    }

    /**
    * Customers entitled to create support Incidents.
    */
    entity Customers : managed { 
    key ID        : String;
    firstName     : String;
    lastName      : String;
    name          : String = firstName ||' '|| lastName;
    email         : EMailAddress;
    phone         : PhoneNumber;
    incidents     : Association to many Incidents on incidents.customer = $self;
    creditCardNo  : String(16) @assert.format: '^[1-9]\d{15}$';
    addresses     : Composition of many Addresses on addresses.customer = $self;
    }

    entity Addresses : cuid, managed {
    customer      : Association to Customers;
    city          : String;
    postCode      : String;
    streetAddress : String;
    }

    entity Status : CodeList {
    key code: String enum {
        new = 'N';
        assigned = 'A'; 
        in_process = 'I'; 
        on_hold = 'H'; 
        resolved = 'R'; 
        closed = 'C'; 
    };
    criticality : Integer;
    }

    entity Urgency : CodeList {
    key code: String enum {
        high = 'H';
        medium = 'M'; 
        low = 'L'; 
    };
    }

    type EMailAddress : String;
    type PhoneNumber : String;
    ```

> Did you notice that the `Customers` entity includes calculated elements? Check the `name` field. The value for this field is calculated by taking into account the values of the fields `firstName` and `lastName`. 
> ```js[5]
> entity Customers : managed { 
>     key ID        : String;
>     firstName     : String;
>     lastName      : String;
>     name          : String = firstName ||' '|| lastName;
>     email         : EMailAddress;
>     phone         : PhoneNumber;
>     incidents     : Association to many Incidents on incidents.customer = $self;
>     creditCardNo  : String(16) @assert.format: '^[1-9]\d{15}$';
>     addresses     : Composition of many Addresses on addresses.customer = $self;
> }
> ```
> Elements can be specified with a calculation expression, in which you can refer to other elements of the same entity. These calculated elements are used for convenience. For more information, see [Calculated Elements](https://cap.cloud.sap/docs/cds/cdl#calculated-elements).


As soon as you save your file, the CAP server that is still running reacts immediately with a new output:

```bash
[cds] - loaded model from 1 file(s):

  db/schema.cds

[cds] - connect using bindings from: { registry: '~/.cds-services.json' }
[cds] - connect to db > sqlite { database: ':memory:' }
/> successfully deployed to in-memory database. 
```

This means that the CAP server detected the changes in **schema.cds** and automatically created an in-memory SQLite database when restarting the server process. However, the CAP server also prints this message:

```bash
No service definitions found in loaded models.
Waiting for some to arrive...
```

### Create services

It's a good practice in CAP to create single-purpose services. Hence, let's define a `ProcessorService` for support engineers to process incidents created by customers.

To create the service definition:

1. In the **srv** folder, create a new **services.cds** file.

2. Paste the following code snippet in the **services.cds** file:

    ```js
    using { sap.capire.incidents as my } from '../db/schema';

    /**
     * Service used by support personell, i.e. the incidents' 'processors'.
     */
    service ProcessorService { 
        entity Incidents as projection on my.Incidents;

        @readonly
        entity Customers as projection on my.Customers;
    }

    /**
     * Service used by administrators to manage customers and incidents.
     */
    service AdminService {
        entity Customers as projection on my.Customers;
        entity Incidents as projection on my.Incidents;
        }
    ```
  
This time, the CAP server reacted with additional output:

```bash
[cds] - serving ProcessorService { path: '/odata/v4/processor' }

[cds] - server listening on { url: 'http://localhost:4004' }
[cds] - [ terminate with ^C ]
```

As you can see in the log output, the new file created a generic service provider `ProcessorService` that serves requests on the `/odata/v4/processor` endpoint. If you open the link `http://localhost:4004` from SAP Business Application Studio in your browser, you'll see the generic `index.html` page:

<!-- border; size:540px --> ![index.html](./index.png)

You may have to stop the CAP server with <kbd>Ctrl</kbd> + <kbd>C</kbd> and start it again with the `cds watch` command.

### Generate comma-separated values (CSV) templates

Since we already have an SQLite in-memory database that was automatically created in the previous step, let's now fill it with some test data.

1. Run the following command in the **incident-management** root folder of your project:

    ```bash
    cds add data
    ```

2. Check the output.

    ```bash
    Adding feature 'data'...
    Creating db/data/sap.capire.incidents-Incidents.csv
    Creating db/data/sap.capire.incidents-Customers.csv
    Creating db/data/sap.capire.incidents-Status.csv
    Creating db/data/sap.capire.incidents-Urgency.csv
    Creating db/data/sap.capire.incidents-Incidents.conversation.csv
    Creating db/data/sap.capire.incidents-Status.texts.csv
    Creating db/data/sap.capire.incidents-Urgency.texts.csv

    Successfully added features to your project.
    ```


    You can find the generated CSV templates within the **db** folder, in a newly-created **data** folder.
     
### Fill in the test data

Replace the respective generated CSV templates with the following content:

- `db/data/sap.capire.incidents-Customers.csv`:

    ```csv
    ID,firstName,lastName,email,phone
    1004155,Daniel,Watts,daniel.watts@demo.com,+44-555-123
    1004161,Stormy,Weathers,stormy.weathers@demo.com,+359-2-0202-0202
    1004100,Sunny,Sunshine,sunny.sunshine@demo.com,+01-555-789
    ```

- `db/data/sap.capire.incidents-Incidents.conversation.csv`:

    ```csv
    ID,up__ID,timestamp,author,message
    2b23bb4b-4ac7-4a24-ac02-aa10cabd842c,3b23bb4b-4ac7-4a24-ac02-aa10cabd842c,1995-12-17T03:24:00Z,Harry John,Can you please check if battery connections are fine?
    2b23bb4b-4ac7-4a24-ac02-aa10cabd843c,3a4ede72-244a-4f5f-8efa-b17e032d01ee,1995-12-18T04:24:00Z,Emily Elizabeth,Can you please check if there are any loose connections?
    9583f982-d7df-4aad-ab26-301d4a157cd7,3583f982-d7df-4aad-ab26-301d4a157cd7,2022-09-04T12:00:00Z,Sunny Sunshine,Please check why the solar panel is broken.
    9583f982-d7df-4aad-ab26-301d4a158cd7,3ccf474c-3881-44b7-99fb-59a2a4668418,2022-09-04T13:00:00Z,Bradley Flowers,What exactly is wrong?
    ```

- `db/data/sap.capire.incidents-Incidents.csv`:

    ```csv
    ID,customer_ID,title,urgency_code,status_code
    3b23bb4b-4ac7-4a24-ac02-aa10cabd842c,1004155,Inverter not functional,H,C
    3a4ede72-244a-4f5f-8efa-b17e032d01ee,1004161,No current on a sunny day,H,N
    3ccf474c-3881-44b7-99fb-59a2a4668418,1004161,Strange noise when switching off Inverter,M,N
    3583f982-d7df-4aad-ab26-301d4a157cd7,1004100,Solar panel broken,H,I
    ```

- `db/data/sap.capire.incidents-Status.csv`:

    ```csv
    code;descr;criticality
    N;New;3
    A;Assigned;2
    I;In Process;2
    H;On Hold;3
    R;Resolved;2
    C;Closed;4
    ```

- `db/data/sap.capire.incidents-Urgency.csv`:

    ```csv
    code;descr
    H;High
    M;Medium
    L;Low
    ```

    > Notice that `cds add data` created seven files, while we're adding data to just five of them. We're leaving the files `sap.capire.incidents-Status.texts.csv` and `sap.capire.incidents-Urgency.texts.csv` empty because they hold translated text that will be filled once the application is localized and translations are created.

Upon detecting these new files, the CAP server prints a message stating that the content of the files has been filled into the database automatically:

```bash
[cds] - connect to db > sqlite { database: ':memory:' }
  > init from db\data\sap.capire.incidents-Incidents.conversation.csv 
  > init from db\data\sap.capire.incidents-Customers.csv 
  > init from db\data\sap.capire.incidents-Incidents.csv 
  > init from db\data\sap.capire.incidents-Status.csv 
  > init from db\data\sap.capire.incidents-Status.texts.csv 
  > init from db\data\sap.capire.incidents-Urgency.csv 
  > init from db\data\sap.capire.incidents-Urgency.texts.csv 
/> successfully deployed to in-memory database.
```
> Make sure your CAP server is still running. You can start it with `cds watch`.

Now that the database is filled with some initial data, you can send complex OData queries served by the built-in generic service providers. With the generic `index.html` page opened in your browser, paste the following queries at the end of the current URL and check the result:

- `/odata/v4/processor/Incidents`

    <!-- border; size:540px --> ![Incidents](./incidents.png)

- `/odata/v4/processor/Customers?$select=firstName&$expand=incidents`

    <!-- border; size:540px --> ![Incidents per customer](./incidents-per-customer.png)

> When you revisit the **Incidents** or the **Customers** endpoint, you might see something like this instead of the nicely formatted output from above.
>
> ![No JSON Viewer](./incidents-unformatted.png)
>
> However, this doesn't mean you've made a mistake in the tutorial. Rather, this is the correct output without any formatting. If you'd like to see a formatted output in your browser, you can add an extension to your browser. Here are a few exemplary JSON formatters for different browsers:
>
> - [Chrome](https://chrome.google.com/webstore/detail/jsonvue/chklaanhfefbnpoihckbnefhakgolnmc)
> - [Microsoft Edge](https://microsoftedge.microsoft.com/addons/detail/jsonview/kmpfgkgaimakokfhgdahhiaaiidiphco)
> - [Safari](https://apps.apple.com/us/app/json-peep-for-safari/id1458969831?mt=12)


    

