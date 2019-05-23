
# Setting Up Orchestra
Orchestra is a tool for managing DV360 tasks like downloading ERFs, parsing them and uploading them to BigQuery.

Ocrhestra is a configuration of Google Composer which is a managed implementation of Apache Airflow. 

Airflow is a platform to programmatically, author, schedule and monitor workflows.

Workflows in Airflow are represented by DAGs.

Orchestra includes several DAGs for working with DV360 and the Google Marketing Platform.

## Before starting 
You will need:
* a Google Cloud Platform account
* a DV360 Account

<walkthrough-author name="Peter Lafferty">
</walkthrough-author>

<walkthrough-tutorial-duration duration="45">
</walkthrough-tutorial-duration>


## Outline of Walkthrough
This walkthrough covers:
1. setting up a project with billing
2. Enabling the relevant Google Cloud Platform APIs for Orchestra
3. Setting up Composer (can take around 30 minutes)
4. Creating a service account
5. Create a new user for your service account in DV360
6. Create a Google BigQuery Dataset
7. Configuring Orchestra
8. Adding worflow DAGs


## Project Setup
Select or create project with billing to use for this walkthrough:
<walkthrough-project-billing-setup>
</walkthrough-project-billing-setup>

[Read more](https://cloud.google.com/billing/docs/how-to/manage-billing-account) about setting up billing in the documentation.

## Enable APIs for {{project-name}}
<walkthrough-enable-apis apis="storage-component.googleapis.com,bigquery-json.googleapis.com,composer.googleapis.com,dataproc.googleapis.com">
</walkthrough-enable-apis>


## Create a Composer environment
This can take up to 30 minutes. 

The environment should be set to use Python 2 as Orchestra currently only supports Python 2.

Choose the LOCATION and the NAME for the composer environment. 

The location is not the same as the Compute Engine locations [choose from the available locations](https://cloud.google.com/composer/pricing#pricing_table).

The NAME must start with a lowercase letter followed by up to 63 lowercase letters, numbers, or hyphens, and cannot end with a hyphen.


This command will set up the composer environment and use the default service account.

``` bash
  gcloud composer environments create --python-version 2 --location LOCATION NAME
```
There are a lot more options available. 

[Read more](https://cloud.google.com/composer/docs/how-to/managing/creating) about setting up composer in the documentation.

## Service Account
A service account will be used to grant access to Google Cloud Platform from DV360.

For simplicity you can use the `Compute Engine default service account`. 

Run the following command to see the current service accounts:

``` bash
gcloud iam service-accounts list --project={{project-name}}
```

Take a note of this email address as you will add it to DV360.

[Read more](https://cloud.google.com/iam/docs/service-accounts) about service accounts in the documentation.
## Display and Video 360 Setup
Using the service account's email address [create a new user in DV360](https://support.google.com/displayvideo/answer/2723011?hl=en).

### Account Configuration
Give the new user the following:
* the email address of the service account
* select all the advertisers to access
* Give the user Read and Write permissions

### Entity Read File Authorization
The service account needs access to the Entity Read Files in DV360. 

Access to the ERFs is granted through a Google Group. The name of the google group can be found in DV360

The settings menu is on the left. You might need to search for a partner or advertiser before the relevant menu shows.

**Settings > Basic Details > Entity Read Files Configuration  > Entity Read Files Read Google Group**

[Add the service account](https://github.com/peterlafferty/orchestra-walkthrough/blob/master/erf.png) to the Entity Read Files **Read** Google Group.


### Multiple Partners
If you are intending to use many google groups, it is also possible to set up a single Google Group containing all other Google Groups. You can then Add the Service account to this Google Group to grant access to all accounts at once

## Create a Cloud BigQuery Dataset 

Data will be imported from Google Marketing Platform APIs and parsed in to a BigQuery Dataset.

You need to create that dataset before running the workflow.

> Dataset IDs must be alphanumeric (plus underscores) and must be at most 1024
characters long.

Use the following command to create a dataset set the NAME and change the location if required:

``` bash
bq --location=US mk {{project-name}}:NAME
```
[Read more](https://cloud.google.com/bigquery/docs/datasets#bigquery_create_dataset-cli) about creating datasets in the official documentation.


## Orchestra Configuration Variables

Name  | Description
------- | --------
gce_zone | The Compute Engine Zone for the Composer environment.
gcs_bucket | The Cloud Storage Bucket for the DAGs
cloud_project_id | The project ID: {{project-name}}.
erf_bq_dataset | The name of the BigQuery Dataset to use.
parnters_id | The list of partners ids from DV360, used for Entity Read Files, comma separated.
private_entity_types | A comma separated list of Private Entity Read Files you would like to import.
sequential_erf_dag_name | The name of your DAG as it will show up in the UI

Set the `gce_zone` you can find the zone in your list of composer environments:
``` bash
 gcloud --project {{project_id}} composer environments run [COMPOSER ENVIRONMENT] --location [LOCATION] variables -- --set gce_zone [ZONE]
```

Next link a bucket. You can view your buckets with:
``` bash
gsutil ls -p {{project_id}}
```

Set a configuration variable `gcs_bucket` for the bucket name without the `gs://` prefix:
``` bash
 gcloud --project {{project_id}} beta composer environments run [COMPOSER ENVIRONMENT] --location [LOCATION] variables -- --set gcs_bucket [BUCKET NAME]
```

Set the project id:
``` bash
 gcloud --project {{project_id}} beta composer environments run [COMPOSER ENVIRONMENT] --location [LOCATION] variables -- --set cloud_project_id {{project_id}}
```

Create a BigQuery dataseet and assign the name to `erf_bq_dataset`:
``` bash
gcloud --project {{project_id}} beta composer environments run [COMPOSER ENVIRONMENT] --location [LOCATION] variables -- --set erf_bg_dataset [dataset name]
```

Set the ``partner_ids`` separated by commas:
``` bash
gcloud --project {{project_id}} beta composer environments run [COMPOSER ENVIRONMENT] --location [LOCATION] variables -- --set partner_ids [partner ids]
```

Set the [Private ERF tables](https://developers.google.com/bid-manager/guides/entity-read/format-v2#private-tables) that you would like to import with `private_entity_types`. Separated by commas:
``` bash
gcloud --project {{project_id}} beta composer environments run [COMPOSER ENVIRONMENT] --location [LOCATION] variables -- --set private_entity_types [entity types]
```

Set a name for your dag as it will show in the UI with `sequential_erf_dag_name`:
``` bash
gcloud --project {{project_id}} beta composer environments run [COMPOSER ENVIRONMENT] --location [LOCATION] variables -- --set sequential_erf_dag_name [any name]
```



## Adding Workflow DAGs
## Final Step
<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>
