# MANUFACTURE USECASE – Backend App Deployment Details  

⚙️ In this use case, participants will simulate interactions with a manufacturing‑machines database—querying machine information, retrieving error histories, and troubleshooting solutions building an agentic solution in **watsonx Orchestrate**. As the instructor guiding the hands‑on lab, you’ll prepare a backend API layer that exposes endpoints for querying machines, error events, and maintenance records (e.g., fetch recent fault occurrences, log a new service entry, trigger threshold alerts). During the bootcamp, learners will step through pre‑configured playbooks in watsonx Orchestrate, invoking your APIs to authenticate, pull diagnostics, record upkeep actions, and even recommend preventive steps—giving them end‑to‑end experience orchestrating real‑world manufacturing maintenance workflows. 


## Section 1 - Milvus Database Setup
 - This application utilizes the Milvus Vector Database to store the machine information, error code information and machine logs.
 - Please follow the [Milvus Setup Guide](./setup_guides/milvus_setup_guide.md) to setup the Milvus Database.

## Section 2 - Backend Application Deployment
### 1. Application Information

 - To simulate the interactions, a backend application will be deployed and interacted with. This application will host APIs that can be used as tools in the participant labs. 
 - You can find the application script [here](./manufacturing_app/main.py). 

The [Deployment Guide](#3-deployment-guide) section will provide the steps to deploy the backend application.

### 2. API Documentation
 - The OpenAPI specification [manufacture.yaml](./manufacturing_app/openapi_spec_files/opeanapi_spec.json) provides API details that can be imported into **watsonx Orchestrate**.
 - Once the backend application has been deployed, The OpenAPI specs will need to be modified to replace the `url` (under *servers*) to reference the application deployment URL (steps below) and shared with the hands-on lab practitioners.

### 3. Deployment Guide
 - The backend application service will be deployed on **IBM Serverless Containers (Code Engine)** and the build image created in **IBM Container Registry**. 

    #### 3.1. Prerequisites
     - Create an unencrypted [SSH key](../../environment-setup/common/sshkey.md) (without a passphrase) and save the public key in your `github.ibm.com` user settings
     - Create an [IBM Cloud API key](../../environment-setup/common/apikey.md) for the TechZone Cloud account. Download this API Key as it will be required in 
        * [Section 3.3.1](/usecase-setup/manufacturing/README.md#331-manual-deployment)
        * [Environment Secret: Keys & Values](/usecase-setup/manufacturing/setup_guides/env_secrets_values_guide.md)

    #### 3.2. Reserve a TechZone cloud environment
     - The instructor will need to reserve a TechZone bundle which includes access to **watsonx.ai**, **watsonx Orchestrate** Trial Bundle, **IBM Container Registry**, **IBM Cloud Object Storage** and **IBM Serverless Containers (Code Engine)**. 
     - For instructor setup, make sure to reserve a single environment. Please follow the instructions in the [Environment Setup](../../environment-setup/readme.md) for making the reservation.

    #### 3.3. Deploy application

    ##### 3.3.1 Manual deployment
    - For manual steps to deploy the backend application, go [here](./setup_guides/Tool_Deployment_Guide.md).
    - Take note of the deployment URL. You will need it in:
        - [Step 3.3.3](#333-update-api-spec-file-with-deployment-url)
        - [Section 1 - Data Ingestion in Milvus Database](/usecase-setup/manufacturing/setup_guides/ingest_data_in_milvus_guide.md#section-1-steps-for-ingesting-data-via-api-endpoint)
        - [Section 2 - New Incident Ingestion into Milvus Database](/usecase-setup/manufacturing/setup_guides/ingest_data_in_milvus_guide.md#section-2-generate-new-incident-record-and-ingest-into-milvus-database).

    ##### 3.3.2 Testing the APIs
     - Once the application is deployed, you can see what APIs are available by going to `<app_url>/docs`. This loads the Swagger API page, where you can access the various APIs defined by the deployed code. 

    ##### 3.3.3. Update API spec file with deployment URL
     - An OpenAPI specification ([manufacture.yaml](./manufacturing_app/openapi_spec_files/opeanapi_spec.json)) is provided with details that will point to the deployed backend application APIs. 
     - Replace the URL of deployed `log-agent-tools` application (noted in [Step 3.3.1](#331-manual-deployment)) in `manufacture.yaml` file as shown in image below. 

        <img width="1000" alt="image" src="./setup_guides/assets/openapi_spec.png">

    > **NOTE: You will need to provide this updated `manufacture.yaml` spec file to participants.**

### 4. Default app data
 - A pre-populated database for Machine Information, Error Code Information and Logs is deployed as part of the backend application.
 - You can view the data [here](./manufacturing_app/documents/)
    - [Machine Information](./manufacturing_app/documents/machine_codes_updated.csv)
    - [Error Code Information](./manufacturing_app/documents/error_codes_updated.csv)
    - [Log File](./manufacturing_app/documents/final_incident_log_with_machine_type_match.csv)

> ***NOTE: To test the application, use Machine Type, Error Codes, Error description as provided in the data***


## Section 3 – Ingest Data in the Milvus Vector Database
 - Once the Milvus Vector Database is setup in [Section 1](/usecase-setup/manufacturing/README.md#section-1---milvus-database-setup) and the application is deployed in [Section 3](/usecase-setup/manufacturing/README.md#3-deployment-guide) . You need to ingest data into the database. 
 - Please follow the guide [Section 1 - Ingest Data in Milvus Database](/usecase-setup/manufacturing/setup_guides/ingest_data_in_milvus_guide.md#section-1-steps-for-ingesting-data-via-api-endpoint) to ingest the data in Milvus Database.

## Section 4 – Adding New Entries to the Database

To simulate a real-time logging server that updates incident log records with new entries, you have two primary options: manually generating and ingesting entries via the API, or automating the process by deploying a job that runs on a regular schedule using event subscriptions.

You can find the job script [here](./manufacturing_app/utils/job_runner.py).

### Option 1: Manual Generation and Ingestion
 - For one-time or on-demand additions, use the FastAPI `/generate_new_incidents` endpoint to create new entries and ingest them directly into the Milvus database. 
 - Please follow the guide: [Section 2 - New Incident Ingestion into Milvus Database](/usecase-setup/manufacturing/setup_guides/ingest_data_in_milvus_guide.md#section-2-generate-new-incident-record-and-ingest-into-milvus-database).

### Option 2: Automated Updates via Job and Event
For regular, scheduled updates (e.g., daily), deploy a job that runs the generation script automatically and subscribe it to an event trigger.

#### 4.1 Create and Deploy a Job
- Follow this guide to set up and deploy the job: [Create Job](./setup_guides/Create-Job-In-Code-Engine.md)

#### 4.2 Create an Event Subscription
- Follow this guide to create an event subscription for the deployed job, configuring it for your desired schedule (e.g., daily triggers): [Create an Event](./setup_guides/create-event-in-code-engine.md)

This automation ensures new entries are generated and ingested without manual intervention, mimicking a production logging server.


