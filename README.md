# graduation-project

1) setup:
    * create a terraform service account, create a key and save it:
    ```bash
    $ export PROJECT_NAME="<project_name_here>"
    $ gcloud iam service-accounts create terraform-admin \
        --description="service account for terraform" \
        --display-name="Terraform Admin" && \
    gcloud projects add-iam-policy-binding $PROJECT_NAME \
        --member=serviceAccount:terraform-admin@$PROJECT_NAME.iam.gserviceaccount.com \
        --role=roles/owner && \
    gcloud iam service-accounts keys create terraform/terraform-admin-key.json \
        --iam-account=terraform-admin@$PROJECT_NAME.iam.gserviceaccount.com
    ```

2) steps: 
    * specify project_id in variables.tf

    * enable all APIs needed for the project, and create a database on CloudSQL service:
    ```
    $ cd terraform && terraform init
    $ terraform apply -target=module.enable_apis -target=module.sql_database
    ```

    * add the INSTANCE_CONNECTION_NAME to the backend API script

    *  create a table from cloudSQL
    ```sql
    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        username VARCHAR(50) UNIQUE NOT NULL
    );
    ```

    * create cloud function, a cloud function service account, <br>
    allow service account to invoke the function and to access cloud sql:

    ```
    $ terraform apply -target=module.cloud_function
    ```

    * add user to the database:
    ```
    curl -X POST <function_trigger_url> -H "Content-Type: application/json" -d '{"username":"Sara"}'
    ```

    * specify the host (managed service) and the backend (function trigger url) in the api-config.yml.

    * create the gateway
    ```
    $ terraform apply -target=module.gateway
    ```

    * enable the created api from cloud console, generate API key, and restrict it to the API

    * test
    ```
    GET:      
     curl -G "<gateway_url>/users" -d "id=1" -d "key=<api_key>"

    POST:
     curl -X POST <gateway_url>/users?key=<api_key>" -H "Content-Type: application/json" -d '{"username": "ahmed"}'
    ```