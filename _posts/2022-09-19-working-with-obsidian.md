---
title: "Working with Obsidian.md"
original_date_label: 2022-09-19T14:27:54-04:00
last_modified_at: 2022-09-19T14:27:54-04:00
#classes: wide
#toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - obsidian
excerpt: "Working with Obsidian"
---

# gcloud Commands


Alerting email address for quicklabs
```
sunny.joy5608@fastmail.com
```

Variables in Cloud Shell
```
$GOOGLE_CLOUD_PROJECT
```


## Getting Started


Step 1:
```
gcloud auth list
```
Output:
```
Credentialed Accounts

ACTIVE: *
ACCOUNT: student-04-46af402f7e14@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`
```

```
gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-f
```

```
gcloud compute instances create --help
```



## App Engine


Browse your app - generates link to click
```
gcloud app browse
```



## Compute Engine



Create Compute Engine Instance
- Debian GNU/Linux 10 (buster)
- Firewall allow HTTPS
```
gcloud compute instances create web-server-vm --project=qwiklabs-gcp-04-da8671513efa --zone=us-central1-a --machine-type=e2-medium --network-interface=network-tier=PREMIUM,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=297405465117-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=https-server --create-disk=auto-delete=yes,boot=yes,device-name=web-server-vm,image=projects/debian-cloud/global/images/debian-10-buster-v20220822,mode=rw,size=10,type=projects/qwiklabs-gcp-04-da8671513efa/zones/us-central1-a/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```


#### Working with Zones

Store the region you are working in:
```
MY_REGION=us-central1
```


Get a list of al available zones in that region:
```
gcloud compute zones list | grep $MY_REGION
```

Select a zone and store it in a variable:
```
MY_ZONE=[ZONE]
```

Set your zone you need to work in:
```
gcloud config set compute/zone $MY_ZONE
```

Specify a VM name:

```
MY_VMNAME=second-vm
```

Create a VM with gcloud:

```
gcloud compute instances create $MY_VMNAME \
--machine-type "e2-standard-2" \
--image-project "debian-cloud" \
--image-family "debian-11" \
--subnet "default"
```

List the virtual machine instances in your project:

```
gcloud compute instances list
```

### Identity and Access Management (IAM)
#iam

In Cloud Shell, execute the following command to create a new service account:
```
gcloud iam service-accounts create test-service-account2 --display-name "test-service-account2"
```

In Cloud Shell, execute the following command to grant the second service account the Project viewer role:
```
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT --member serviceAccount:test-service-account2@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com --role roles/viewer
```

Authenticate as a service account in Cloud Shell
```
gcloud auth activate-service-account --key-file credentials.json
```

To verify the active account, execute the following command:
```
gcloud config list
```

To switch to the lab account, execute the following command, replacing `[USERNAME]` with the username provided in the Qwiklabs Connection Details pane on the left of the lab instructions page:
```
gcloud config set account [USERNAME]
```




## Cloud Storage
Copy a picture of a cat from a Google-provided Cloud Storage bucket to your Cloud Shell:
```
gsutil cp gs://cloud-training/ak8s/cat.jpg cat.jpg
```

Copy the file into the first buckets that you created earlier:
```
gsutil cp cat.jpg gs://$MY_BUCKET_NAME_1
```

Copy the file from the first bucket into the second bucket:
```
gsutil cp gs://$MY_BUCKET_NAME_1/cat.jpg gs://$MY_BUCKET_NAME_2/cat.jpg
```

Get the default access list that's been assigned to `cat.jpg`

```
gsutil acl get gs://$MY_BUCKET_NAME_1/cat.jpg  > acl.txt
cat acl.txt
```

Make the first Cloud Storage bucket readable by everyone, including unauthenticated users:
```
gsutil iam ch allUsers:objectViewer gs://$MY_BUCKET_NAME_1
```




## Cloud Run


Set the variable to the URL of the App
```
URL=$(gcloud run services list --platform managed --format="value(URL)" | grep hello-logging)
```

Show Variable
```
echo $URL
```


## Logging | Logs Explorer

```
resource.type="cloud_run_revision"
resource.labels.service_name="hello-logging"
log_name="projects/qwiklabs-gcp-04-0efe5dd2102f/logs/run.googleapis.com%2Fstdout"

-(textPayload=~"Called /random-error, and it worked")
logName=("projects/qwiklabs-gcp-04-0efe5dd2102f/logs/run.googleapis.com%2Fstderr" OR "projects/qwiklabs-gcp-04-0efe5dd2102f/logs/run.googleapis.com%2Frequests")

-(httpRequest.status="200")
```


## BigQuery
```
SELECT 
  textPayload
FROM `qwiklabs-gcp-04-0efe5dd2102f.hello_logging_logs.run_googleapis_com_stderr_20220901` 
WHERE
  textPayload LIKE 'ReferenceError%'
LIMIT 1000
```

```
SELECT
  errors / total_requests
FROM (
  SELECT
    (
    SELECT
      COUNT(*)
    FROM
      `qwiklabs-gcp-04-0efe5dd2102f.hello_logging_logs.run_googleapis_com_stderr_20220901`) AS total_requests,
    (
    SELECT
      COUNT(textPayload)
    FROM
      `qwiklabs-gcp-04-0efe5dd2102f.hello_logging_logs.run_googleapis_com_stderr_20220901`
    WHERE
      textPayload LIKE 'ReferenceError%') AS errors)
```











