# Google-Cloud-Repository
****************Automate Interactions with Contact Center AI******************************
              



git clone https://github.com/GoogleCloudPlatform/dataflow-contact-center-speech-analysis.git



           -------------------------------------step 1-----------------------------
Task 1: Create a Cloud Storage Bucket


gsutil mb -l us-central1 gs://[project id]/

Task 2: Create a Cloud Function
Go to the “saf-longrun-job-func” directory
gcloud functions deploy safLongRunJobFunc --runtime nodejs8 --trigger-resource [project id] --region us-central1 --trigger-event google.storage.object.finalize






Task 3: Create a BigQuery Dataset and Table
# To create the dataset
bq mk saf



Task 4: Create Cloud Pub/Sub Topic
gcloud pubsub topics create [Topic_Name]




Task 5: Create a Cloud Storage Bucket for Staging Contents
gsutil mb -l [BUCKET_LOCATION] gs://[BUCKET_NAME]/
Create a folder “DFaudio” in the bucket

do it manually



Task 6: Deploy a Cloud Dataflow Pipeline
Go to the saf-longrun-job-dataflow directory


python -m virtualenv env -p python3
source env/bin/activate
pip install apache-beam[gcp]
pip install dateparser


Now, execute the command given below:
python saflongrunjobdataflow.py \
--project=[PROJECT_ID] \
--region=us-central1 \
--input_topic=projects/[PROJECT_ID]/topics/[Topic_Name] \
--runner=DataflowRunner \
--temp_location=gs://[BUCKET_NAME_FOR_STAGING_CONTENT]/[FOLDER] \
--output_bigquery=[PROJECT_ID]:saf.transcripts \
--requirements_file=requirements.txt





Task 7: Upload Sample Audio Files for Processing
gsutil -h x-goog-meta-callid:1234567 -h x-goog-meta-stereo:false -h x-goog-meta-pubsubtopicname:[TOPIC_NAME] -h x-goog-meta-year:2019 -h x-goog-meta-month:11 -h x-goog-meta-day:06 -h x-goog-meta-starttime:1116 cp gs://qwiklabs-bucket-gsp311/speech_commercial_mono.flac gs://[YOUR_UPLOADED_AUDIO_FILES_BUCKET_NAME(task1 bucket name)]

gsutil -h x-goog-meta-callid:1234567 -h x-goog-meta-stereo:true -h x-goog-meta-pubsubtopicname:[TOPIC_NAME] -h x-goog-meta-year:2019 -h x-goog-meta-month:11 -h x-goog-meta-day:06 -h x-goog-meta-starttime:1116 cp gs://qwiklabs-bucket-gsp311/speech_commercial_stereo.wav gs://[YOUR_UPLOADED_AUDIO_FILES_BUCKET_NAME(task1 bucket name)]



After performing Task 7: Upload Sample Audio Files for Processing, we have to wait....until we see output in bigquery > dataset > table.





Task 8: Run a Data Loss Prevention Job



select * from (SELECT entities.name,entities.type, COUNT(entities.name) AS count FROM saf.transcripts, UNNEST(entities) entities GROUP BY entities.name, entities.type ORDER BY count ASC ) Where count > 5

Now, click on “Save Query Results” and select “BigQuery table” option

Enter a name for a new table and save.
Go to the new table in which result is saved and then Click on Export > Scan with DLP
