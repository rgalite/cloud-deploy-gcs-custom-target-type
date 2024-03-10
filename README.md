# Introduction

# Create infrastructure

```bash
export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")
export REGION=europe-west1
```

- Active the required APIs

```bash
gcloud services enable cloudbuild.googleapis.com \
clouddeploy.googleapis.com
```

- Create a bucket to serve assets for dev.

```bash
gcloud storage buckets create gs://$PROJECT_ID-dev
```

- Create a bucket to serve assets for production.

```bash
gcloud storage buckets create gs://$PROJECT_ID-prod
```

- Make buckets publically readable

```bash
gcloud storage buckets add-iam-policy-binding gs://$PROJECT_ID-dev --member=allUsers --role=roles/storage.objectViewer
```

```bash
gcloud storage buckets add-iam-policy-binding gs://$PROJECT_ID-prod --member=allUsers --role=roles/storage.objectViewer
```

- Give permissions to SA

```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$PROJECT_NUMBER-compute@developer.gserviceaccount.com \
--role=roles/clouddeploy.jobRunner
```

- Give permissions to the cloud build SA to create releases

```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
--role=roles/clouddeploy.releaser
```

- Give permissions to the cloud build SA to impersonate the default compute developer SA

```bash
gcloud iam service-accounts add-iam-policy-binding $PROJECT_NUMBER-compute@developer.gserviceaccount.com \
--member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com \
--role=roles/iam.serviceAccountUser \
--project=$PROJECT_ID
```

- Deploy the custom target type gcs

```bash
gcloud deploy apply --file=./cd-manifests/target-type-gcs.yaml --region=$REGION
```

- Deploy the custom target web-dev

```bash
gcloud deploy apply --file=./cd-manifests/target-dev.yaml --region=$REGION
```

- Deploy the custom target web-prod

```bash
gcloud deploy apply --file=./cd-manifests/target-prod.yaml --region=$REGION
```

- Deploy the pipeline

```bash
gcloud deploy apply --file=./cd-manifests/pipeline.yaml --region=$REGION
```
