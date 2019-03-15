# Setting Up Orchestra

Hello world 
## Introduction
<walkthrough-project-billing-setup>
</walkthrough-project-billing-setup>

## Enable APIs for {{project-name}}
<walkthrough-enable-apis apis="storage-component.googleapis.com,bigquery-json.googleapis.com,composer.googleapis.com,dataproc.googleapis.com">
</walkthrough-enable-apis>

<walkthrough-author name="Peter Lafferty">
</walkthrough-author>
<walkthrough-duration duration="120">
</walkthrough-duration>

## Create a Composer environment
This can take up to 30 minutes. The environment should be set to use Python 2.

Choose the LOCATION and the ENVIRONMENT_NAME for the composer environment. 

ENVIRONMENT_NAME must match the pattern Must match the pattern: ``^[a-z](?:[-0-9a-z]{0,62}[0-9a-z])?$``

This command will set up the composer environment and use the default service account.

``` bash
  gcloud composer environments create --python-version 2 --location LOCATION ENVIRONMENT_NAME
```

## Final Step
