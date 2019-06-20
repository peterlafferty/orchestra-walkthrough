
# Setting Up Orchestra
Orchestra is a tool for managing DV360 tasks like downloading ERFs, parsing them and uploading them to BigQuery.

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


## Location 
Choose the LOCATION for the composer environment. 

Save the location in an environment variable:
``` bash
 export LOCATION=your_choice
```
The location is not the same as the Compute Engine locations [choose from the available locations](https://cloud.google.com/composer/pricing#pricing_table).

# Environment Name

Choose the NAME for the composer environment. 

> The NAME must start with a lowercase letter followed by up to 63 lowercase letters, numbers, or hyphens, and cannot end with a hyphen.

Save the name in an environment variable:
``` bash
 export ENVIRONMENT=your_choice
```

## Create Composer Environment
This command will set up the composer environment and use the default service account.

``` bash
  gcloud composer environments create --python-version 2 --location $LOCATION $ENVIRONMENT
```

[Read more](https://cloud.google.com/composer/docs/how-to/managing/creating) about setting up composer in the documentation.

## Service Account
A service account will be used to grant access to Google Cloud Platform from DV360.

Run the following command to see the current service accounts:

``` bash
gcloud iam service-accounts list --project={{project-name}}
```

Take a note of the default service account email address as you will add it to DV360.

[Read more](https://cloud.google.com/iam/docs/service-accounts) about service accounts in the documentation.
## Display and Video 360 Setup
Using the service account email address [create a new user in DV360](https://support.google.com/displayvideo/answer/2723011?hl=en).

### Account Configuration
Give the new user the following:
* the email address of the service account
* select all the advertisers to access
* Give the user Read and Write permissions

### Entity Read File Authorization
The service account needs access to the Entity Read Files in DV360. 

Access to the ERFs is granted through a Google Group. The name of the google group can be found in DV360

The settings menu is on the left. You might need to search for a partner or advertiser before the relevant menu shows:

**Settings > Basic Details > Entity Read Files Configuration  > Entity Read Files Read Google Group**.

[Add the service account](https://github.com/peterlafferty/orchestra-walkthrough/blob/master/erf.png) to the Entity Read Files **Read** Google Group.


### Multiple Partners
If you are intending to use many google groups, it is also possible to set up a single Google Group containing all other Google Groups. You can then Add the Service account to this Google Group to grant access to all accounts at once

## Create a Cloud BigQuery Dataset 

Data will be imported from Google Marketing Platform APIs and parsed in to a BigQuery Dataset.

You need to create that dataset before running the workflow.

> Dataset IDs must be alphanumeric (plus underscores) and must be at most 1024
characters long.

Use the following command to create a dataset. Set the NAME and change the location if required:

``` bash
bq --location=US mk {{project-name}}:NAME
```
[Read more](https://cloud.google.com/bigquery/docs/datasets#bigquery_create_dataset-cli) about creating datasets in the official documentation.


## Orchestra Configuration Variables

The following variables will need to be set in Airflow.

Name  | Description
------- | --------
gce_zone | The Compute Engine Zone for the Composer environment.
gcs_bucket | The Cloud Storage Bucket for the DAGs
cloud_project_id | The project ID: {{project-name}}.
erf_bq_dataset | The name of the BigQuery Dataset to use.
parnters_id | The list of partners ids from DV360, used for Entity Read Files, comma separated.
private_entity_types | A comma separated list of Private Entity Read Files you would like to import.
sequential_erf_dag_name | The name of your DAG as it will show up in the UI


## Zone - Configure Orchestra

Set the `gce_zone` using the LOCATION and ENVIRONMENT variables set earlier:

``` bash
 gcloud --project {{project_id}} composer environments run $ENVIRONMENT --location $LOCATION variables -- --set gce_zone $LOCATION
```

## Cloud Storage - Configure Orchestra


Link a bucket. There is a bucket already linked to your composer environment:
``` bash
 gcloud composer environments describe $ENVIRONMENT --location $LOCATION | grep bucket
```

Set the `gcs_bucket` variable to the bucket name without the schema:
``` bash
 gcloud --project orchestra-walkthrough composer environments run $ENVIRONMENT --location $LOCATION variables -- --set gcs_bucket BUCKET
```

## Project ID - Configure Orchestra


Set the `cloud_project_id`:
``` bash
 gcloud --project {{project_id}} beta composer environments run $ENVIRONMENT --location $LOCATION variables -- --set cloud_project_id {{project_id}}
```

## BigQuery - Configure Orchestra


List BigQuery datasets
``` bash
 bq ls
```

Set  `erf_bq_dataset` to be the name of a dataset in BigQuery:
``` bash
 gcloud --project {{project_id}} beta composer environments run $ENVIRONMENT --location $LOCATION variables -- --set erf_bg_dataset DATASET
```

old code:
``` bash
 gcloud --project {{project_id}} beta composer environments run $ENVIRONMENT --location $LOCATION variables -- --set erf_bq_table DATASET
```

## Partners - Configure Orchestra


Set the ``partner_ids`` (separated by commas):
``` bash
 gcloud --project {{project_id}} beta composer environments run $ENVIRONMENT --location $LOCATION variables -- --set partner_ids PARTNERTIDS
```

## ERF Tables - Configure Orchestra

Set  `private_entity_types` to the [Private ERF tables](https://developers.google.com/bid-manager/guides/entity-read/format-v2#private-tables) (eparated by commas).

The private tables are (at the time of writing):
 * Advertiser
 * Campaign
 * Creative
 * CustomAffinity
 * InsertionOrder
 * InventorySource
 * LineItem
 * Partner
 * Pixel
 * UniversalChannel

Set the variable:

``` bash
gcloud --project {{project_id}} beta composer environments run $ENVIRONMENT --location $LOCATION variables -- --set private_entity_types PRIVATE_TABLES
```

## DAG name - Configure Orchestra

Set `sequential_erf_dag_name`:
``` bash
gcloud --project {{project_id}} beta composer environments run $ENVIRONMENT --location $LOCATION variables -- --set sequential_erf_dag_name ANY_NAME
```

## Adding Workflow DAGs
As with any other Airflow deployment, you will need DAG files describing your Workflows to schedule and run your tasks; plus, you'll need hooks, operators and other libraries to help building those tasks.

In the Orchestra repo you will find the following directories:
### dags 
Includes a sample DAG file to upload multiple partners ERF files from the Cloud Storage Bucket to BigQuery.

### hooks
Includes the hooks needed to connect to the reporting APIs of GMP platforms (CM and DV360).

### operators
Includes two subfolders for basic operators for CM and DV360 APIs, respectively.

### schema
Includes files describing the structure of most CM and DV360 entities (can be useful when creating new report or to provide the schema to create a BQ table).
### utils
A general purpose folder to include utility files.

## Upload Workflows to Cloud Storage
List the Google Cloud Storage options and choose the one associated with your Composer environment.

``` bash
 gsutil ls
``` 

to make the next steps easier export the URI to a variable:
``` bash
 export BUCKET=gs://bucketname/
```

Need to figure out where the orchestra code sits:
```
gsutil cp operators ${BUCKET}dags/operators
gsutil cp -r hooks ${BUCKET}dags/hooks
gsutil cp -r schema ${BUCKET}dags/schema
gsutil cp -r utils ${BUCKET}/dags/utils
gsutil cp dags/sequential_erf_uploader_to_bq_dag.py ${BUCKET}/dags/


```






## Final Step
<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>
