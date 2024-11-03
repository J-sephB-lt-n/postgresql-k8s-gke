This repo contains code to set up and run a [PostgreSQL](https://www.postgresql.org/) database on [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) using [CloudNativePG](https://cloudnative-pg.io/).

!!!this repo is still a work in progress!!!

Structure of the cluster:

1. A single API gateway pod which accepts external requests

2. A postgreSQL database hosted on GKE

    - Using CloudNative-PG

    - Runs scheduled backups writing to cloud storage

    - Illustrates how to recover the database after a failure, or to a desired point in time

3. An auto-scaling flask app which performs basic CRUD operations on the database

```bash
gcloud auth login
gcloud config set project $GCP_PROJECT_ID
gcloud config set run/region $GCP_REGION

gcloud components install kubectl

# create cluster #
gcloud beta container \
--project $GCP_PROJECT_ID \
clusters create-auto \
"cloud-native-postgresql-cluster" \
--region $GCP_REGION

# get authentication credentials to interact with the cluster #
gcloud container clusters \
get-credentials \
"cloud-native-postgresql-cluster" \
--region $GCP_REGION \
--project $GCP_PROJECT_ID
```

```bash
# deploy CloudNative-PostGreSQL #
kubectl apply -f \
    https://github.com/cloudnative-pg/cloudnative-pg/releases/download/v1.22.1/cnpg-1.22.1.yaml
```

```bash
# create a service account for the PostGreSQL operator #
export CN_POSTGRESQL_OPERATOR_SERV_ACCT_NAME="cloudnative-postgresql-operator"

gcloud iam service-accounts create $CN_POSTGRESQL_OPERATOR_SERV_ACCT_NAME \
--description="A service account for the Cloud-Native PostGreSQL operator on GKE"

gcloud projects add-iam-policy-binding $GCP_PROJECT_ID
--member="serviceAccount:${CN_POSTGRESQL_OPERATOR_SERV_ACCT_NAME}@${GCP_PROJECT_ID}.iam.gserviceaccount.com" 
--role="roles/storage.admin"

gcloud projects add-iam-policy-binding $GCP_PROJECT_ID
--member="serviceAccount:${CN_POSTGRESQL_OPERATOR_SERV_ACCT_NAME}@${GCP_PROJECT_ID}.iam.gserviceaccount.com" 
--role="roles/iam.workloadIdentityUser"
```


# References 

* https://jsilva.cloud/provisioning-cnpg/
