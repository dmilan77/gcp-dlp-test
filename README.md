# gcp-dlp-test
```
https://cloud.google.com/solutions/creating-cloud-dlp-de-identification-transformation-templates-pii-dataset

export KEY_RING_NAME=dmilankeyring; echo ${KEY_RING_NAME}

export KEY_NAME=dmilankeyname; echo ${KEY_NAME}

export KEK_FILE_NAME=dmilanfilename.json; echo ${KEK_FILE_NAME}





data-protection-269221-data-storage-bucket
data-protection-269221-dataflow-temp-bucket



export PROJECT_ID=$(gcloud config get-value project)
export DATA_STORAGE_BUCKET=${PROJECT_ID}-data-storage-bucket
export DATAFLOW_TEMP_BUCKET=${PROJECT_ID}-dataflow-temp-bucket
gsutil mb -c standard -l us-east1 gs://${DATA_STORAGE_BUCKET}
gsutil mb -c standard -l us-east1 gs://${DATAFLOW_TEMP_BUCKET}

export TEK=$(openssl rand -base64 32); echo ${TEK}
export KEY_RING_NAME=key-ring-name; echo ${KEY_RING_NAME}
export KEY_NAME=key-name; echo ${KEY_NAME}
export KEK_FILE_NAME=kek-file-name-json; echo ${KEK_FILE_NAME}

gcloud builds submit . --config dlp-demo-part-1-crypto-key.yaml --substitutions _GCS_BUCKET_NAME=gs://${DATA_STORAGE_BUCKET},_KEY_RING
_NAME=${KEY_RING_NAME},_KEY_NAME=${KEY_NAME},_TEK=${TEK}, _KEK=${KEK_FILE_NAME},_API_KEY='$(gcloud auth print-access-token)'

export SERVICE_ACCOUNT_NAME=service-account-name


export SERVICE_ACCOUNT_NAME=dlp-service-account-name

gcloud iam service-accounts create ${SERVICE_ACCOUNT_NAME} \
    --display-name "DLP Demo Service Account"

gcloud iam service-accounts keys create \
    --iam-account ${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com dlp_key.json


gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member serviceAccount:${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com \
    --role roles/editor

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
    --member serviceAccount:${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com \
    --role roles/storage.admin


gcloud auth activate-service-account --key-file dlp_key.json




gcloud builds submit . --config dlp-demo-part-2-dlp-template.yaml \
    --substitutions \
    _KEK_CONFIG_FILE=gs://${DATA_STORAGE_BUCKET}/${KEK_FILE_NAME},_GCS_BUCKET_NAME=gs://${DATA_STORAGE_BUCKET},_API_KEY=$(gcloud auth print-access-token)

gsutil cp gs://${DATA_STORAGE_BUCKET}/deid-template.json .


gsutil cp gs://${DATA_STORAGE_BUCKET}/inspect-template.json .

cat inspect-template.json

export DEID_TEMPLATE_NAME=$(jq -r '.name' deid-template.json);echo $DEID_TEMPLATE_NAME

export INSPECT_TEMPLATE_NAME=$(jq -r '.name' inspect-template.json);echo $INSPECT_TEMPLATE_NAME


gcloud dataflow jobs run ${jobId}  \
    --gcs-location gs://dataflow-templates/latest/Stream_DLP_GCS_Text_to_BigQuery \
    --parameters \
    inputFilePattern=gs://${DATA_STORAGE_BUCKET}/CCRecords_1564602825.csv,dlpProjectId=${PROJECT_ID},deidentifyTemplateName=${DEID_TEMPLATE_NAME},inspectTemplateName=${INSPECT_TEMPLATE_NAME},datasetName=${BQ_DATASET_NAME},batchSize=500


```
