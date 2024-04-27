
# Service Accounts: How to use client libraries to access BigQuery using a service account

Service account belongs to your applications rather than an individual end user.

Your application uses a service account as an identity to authenticate to Google Cloud services.

You grant roles to a service account so that the service account has permission to complete specific actions on the resources in your Cloud Platform project. For example, you might grant the `storage.admin` role to a service account so that it has control over objects and buckets in Cloud Storage.

Google BigQuery is a managed data warehouse service offered by Google Cloud Platform. It allows you to store, analyze, and query large datasets. To acess Bigquery from your applications, you will need to create a service account for the service e.g VM running your application.

The following steps help us to achieve the aim of creating a service account for our VM. But first, we will practice with a sample service account named `my-sa-123`

### STEPS

0. Navigate to cloudshell from the console and set the region and zone for your project. Click *continue* 

Set region and zone for your project:
#
    gcloud config set compute/region us-east4; gcloud config set compute/zone us-east4-c
Click **Authorize** (if prompted)

![set region and zone](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20155402.png)

(Ignore the property validation error)

1. Create sample service account named `my-sa-123`:

#
    gcloud iam service-accounts create my-sa-123 --display-name "my service account"

![my-sa-123](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20160048.png)

2. Grant `editor` role to the `my-sa-123` service account :

#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:my-sa-123@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/editor

You will see an output similar to the image below:
![my-sa-123-editor role](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20160340.png)

3. We have created a sample service role. Now create a service account named **bigquery-qwiklab** (you can name your service account whatever is appropriate)
#
    gcloud iam service-accounts create bigquery-qwiklab --display-name "bigquery-qwiklab"

![bigquery-service-role](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20160511.png)

4. Create roles `bigquery data viewer` and `bigquery user` for the service account. View the [documentation](https://cloud.google.com/bigquery/docs/access-control)

#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:bigquery-qwiklab@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/bigquery.dataViewer 

#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:bigquery-qwiklab@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/bigquery.user

5. Create a VM instance with the following information:
* **Name**: bigquery-instance
* **Region**: us-west1
* **Zone** : us-west1-d
* **Series**: E2
* **Machine type**: e2-medium
* **Boot Disk**: Debian GNU/Linux 11 (bullseye) x86/64
* **Service account**: biquery-qwiklab

Create the VM with the following command:
#
    gcloud compute instances create bigquery-instance \
    --project=$DEVSHELL_PROJECT_ID \
    --machine-type=e2-medium \
    --service-account=bigquery-qwiklab@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
    --scopes=https://www.googleapis.com/auth/cloud-platform \
    --image=projects/debian-cloud/global/images/family/debian-11
   
![bigquery-instance](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20162925.png)

6. Verify that the VM was created
#
    gcloud compute instances describe bigquery-instance

7. SSH into the VM by clicking on the SSH tab beside the VM on the console or run the following command:
#
    gcloud compute ssh bigquery-instance   
Approve the connection and continue without a passphrase

8. On the VM, in the SSH window, install the python libraries and prerequisites using the following commands.
#
    sudo apt-get update -y
    sudo apt-get install -y git python3-pip
    pip3 install --upgrade pip
    pip3 install google-cloud-bigquery
    pip3 install pyarrow
    pip3 install pandas
    pip3 install db-dtypes

Ignore all errors.
You will see an output similar to the image below:
![libraries-installation](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20163629.png)

9. Create a python file named `query.py` 

```
echo "
from google.auth import compute_engine
from google.cloud import bigquery

credentials = compute_engine.Credentials(
    service_account_email='YOUR_SERVICE_ACCOUNT')

query = '''
SELECT
  year,
  COUNT(1) as num_babies
FROM
  publicdata.samples.natality
WHERE
  year > 2000
GROUP BY
  year
'''

client = bigquery.Client(
    project='qwiklabs-gcp-04-ac30b309731d',
    credentials=credentials)
print(client.query(query).to_dataframe())
" > query.py
```
    
![query.py](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20163744.png)

Add the Project ID to `query.py` with the following command:
#
    sed -i -e "s/Your Project ID/$(gcloud config get-value project)/g" query.py

Verify with the following command:
#
    cat query.py

Add the service account email to query.py with:

#
    sed -i -e "s/YOUR_SERVICE_ACCOUNT/bigquery-qwiklab@$(gcloud config get-value project).iam.gserviceaccount.com/g" query.py

10. Run the query with the following command:
#
    python3 query.py
    
![query with python](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20165020.png)


*Troubleshooting*

*Ensure there is proper indentation. Python scripts are strict with indentations. Otherwise, it will throw an error. If there is no error, proceed.*

Error:
![Error](https://github.com/laraadeboye/GCP_projects/blob/main/service-account-bigquery/images/Screenshot%202024-04-27%20164246.png)

*To solve the error use an editor to open the query.py text file and correct the indentation*

## Conclusion
Usng a bigquery service account, we were able to make a request to BigQuery public dataset.
