# Setting Up Orchestra
Orchestra is a tool for managing Display and Video 360(DV360) ETL tasks like downloading ERFs and uploading them to BigQuery.

This walkthrough covers:
1. setting up a project with billing
2. Enabling the relevant GCP APIs for Orchestra
3. Setting up Composer (can take around 30 minutes)
4. Creating a service account
5. Create a new user for your service account in DV360
6. Configuring Orchestra
7. Adding worflow DAGs

<walkthrough-author name="Peter Lafferty">
</walkthrough-author>

<walkthrough-duration duration="45">
</walkthrough-duration>

## Project Setup
Select or create project with billing to use for this walkthrough:
<walkthrough-project-billing-setup>
</walkthrough-project-billing-setup>

<walkthrough-tutorial-card url="https://cloud.google.com/billing/docs/how-to/manage-billing-account"></walkthrough-tutorial-card>

## Enable APIs for {{project-name}}
<walkthrough-enable-apis apis="storage-component.googleapis.com,bigquery-json.googleapis.com,composer.googleapis.com,dataproc.googleapis.com">
</walkthrough-enable-apis>


## Create a Composer environment
This can take up to 30 minutes. The environment should be set to use Python 2.

Choose the LOCATION and the ENVIRONMENT_NAME for the composer environment. 

ENVIRONMENT_NAME must match the: `^[a-z](?:[-0-9a-z]{0,62}[0-9a-z])?$`

This command will set up the composer environment and use the default service account.

``` bash
  gcloud composer environments create --python-version 2 --location LOCATION ENVIRONMENT_NAME
```
<walkthrough-tutorial-card url="https://cloud.google.com/composer/docs/"></walkthrough-tutorial-card>
## Service Account
## Display and Video 360 Setup
## Orchestra Configuration Variables
## Adding Workflow DAGs
## Final Step
<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>
