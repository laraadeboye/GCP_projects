# Implement Cloud Security Fundamentals on Google Cloud: Challenge Lab

Challenge scenario
You have started a new role as a junior member of the security team for the Orca team in Jooli Inc. Your team is responsible for ensuring the security of the Cloud infrastructure and services that the company's applications depend on.

You are expected to have the skills and knowledge for these tasks, so don't expect step-by-step guides to be provided.

Your challenge
You have been asked to deploy, configure, and test a new Kubernetes Engine cluster that will be used for application development and pipeline testing by the Orca development team.

As per the organization's security standards you must ensure that the new Kubernetes Engine cluster is built according to the organization's most recent security standards and thereby must comply with the following:

The cluster must be deployed using a dedicated service account configured with the least privileges required.
The cluster must be deployed as a Kubernetes Engine private cluster, with the public endpoint disabled, and the master authorized network set to include only the ip-address of the Orca group's management jumphost.
The Kubernetes Engine private cluster must be deployed to the orca-build-subnet in the Orca Build VPC.
From a previous project you know that the [minimum permissions](https://cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster#use_least_privilege_sa) required by the service account that is specified for a Kubernetes Engine cluster is covered by these three built in roles:
```
roles/monitoring.viewer
roles/monitoring.metricWriter
roles/logging.logWriter
```
These roles are specified in the Google Kubernetes Engine (GKE)'s Harden your cluster's security guide in the Use least privilege Google service accounts section.

You must bind the above roles to the service account used by the cluster as well as a custom role that you must create in order to provide access to any other services specified by the development team. Initially you have been told that the development team requires that the service account used by the cluster should have the permissions necessary to add and update objects in Google Cloud Storage buckets. To do this you will have to create a new custom IAM role that will provide the following permissions:
```
storage.buckets.get
storage.objects.get
storage.objects.list
storage.objects.update
storage.objects.create
```
Once you have created the new private cluster you must test that it is correctly configured by connecting to it from the jumphost, `orca-jumphost`, in the management subnet `orca-mgmt-subnet`. As this compute instance is not in the same subnet as the private cluster you must make sure that the master authorized networks for the cluster includes the internal ip-address for the instance, and you must specify the `--internal-ip `flag when retrieving cluster credentials using the `gcloud container clusters get-credentials` command.

All new cloud objects and services that you create should include the `orca-` prefix.

Your final task is to validate that the cluster is working correctly by deploying a simple application to the cluster to test that management access to the cluster using the `kubectl` tool is working from the `orca-jumphost` compute instance.

For all tasks in this lab, use the `us-east4` region and the `us-east4-c` zone.


## Prerequisites
First let's create environmental variables for the region and zone; and config set the project.

#
    export REGION=europe-west1;export ZONE=europe-west1-c

#
    gcloud config set compute/region $REGION;gcloud config set compute/zone $ZONE
## Task 1 Create a Custom Security role
We would create a custom security role that will permit
- `storage.objects.update`
- `storage.objects.create`
- `storage.buckets.get`
- `storage.objects.get`
- `storage.objects.list`
We will use the yaml syntax:

1. Create a yaml file:
#
    nano role-definition.yaml

2. Enter the following details and save the file:
```
title: "orca_storage_editor_906"
description: "Storage bucket and object permissions"
stage: "ALPHA"
includedPermissions:
- storage.objects.update
- storage.objects.create
- storage.buckets.get
- storage.objects.get
- storage.objects.list
```

3. Then we will run the following command:
#
    gcloud iam roles create orca_storage_editor_233 --project $DEVSHELL_PROJECT_ID \--file role-definition.yaml

    
## Task 2.  Create a service account
We will create a service account that will be used as the service account for our new private cluster named `orca-private-cluster-904-sa`

#
    gcloud iam service-accounts create orca-private-cluster-904-sa --display-name "orca-private-cluster-904-sa"


## Task 3. Bind a custom security role to a service account

We will now bind the custom security roles for cloud operations logging and monitoring as well as the IAM storage permissions role we created:

1. roles/monitoring.viewer
#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \--member serviceAccount:orca-private-cluster-904-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.viewer

2. roles/monitoring.metricWriter
#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \--member serviceAccount:orca-private-cluster-904-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.metricWriter


3. roles/logging.logWriter
#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \--member serviceAccount:orca-private-cluster-904-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/logging.logWriter


4. projects/$DEVSHELL_PROJECT_ID/roles/orca_storage_editor_906
#
    gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \--member serviceAccount:orca-private-cluster-904-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role projects/$DEVSHELL_PROJECT_ID/roles/orca_storage_editor_233

Note that the name of the custom role has to be in the form of :
`projects/{project-id}/roles/{role-id}`

## Task 4. Create and configure a new Kubernetes Engine private cluster
1. First, we will list the subnet in our network `orca-build-vpc` 

#
    gcloud compute networks subnets list --network orca-build-vpc

Add secondary CIDR ranges to an existing subnet `orca-build-subnet`
#
    gcloud compute networks subnets update orca-build-subnet \
    --region $REGION \
    --add-secondary-ranges my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14

To get the information about your created subnet
#
    gcloud compute networks subnets describe [SUBNET_NAME] --region=$REGION

2. Next we will create a new private kubernetes cluster using the service account we configured `orca-private-cluster-904-sa`

#
    gcloud beta container clusters create orca-cluster-321 \
    --enable-private-nodes \
    --enable-private-endpoint \
    --enable-ip-alias \
    --enable-master-authorized-networks \
    --master-ipv4-cidr 172.16.0.32/28 \
    --network orca-build-vpc \
    --subnetwork orca-build-subnet \
    --services-secondary-range-name my-svc-range \
    --cluster-secondary-range-name my-pod-range \
    --service-account=orca-private-cluster-904-sa@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com \
    --zone=$ZONE

Note that I specified the network and subnetwork..






## Task 5. Deploy an application to a private Kubernetes Engine Cluster
1. First create a VM instance named `orca-jumphost` to check the connectivity to Kubernetes clusters (if not created):

#
    gcloud compute instances create orca-jumphost --zone=$ZONE --scopes 'https://www.googleapis.com/auth/cloud-platform'

2. Get the internal ip

#
    gcloud compute instances describe orca-jumphost --zone=$ZONE | grep IP

The internal Ip is the one named networkIP
3. Authorize the IP of your instance for access to the cluster
#
    gcloud container clusters update orca-cluster-321 \
    --enable-master-authorized-networks \
    --master-authorized-networks 192.168.10.2/32


4. SSH to the jumphost
#
    gcloud compute ssh orca-jumphost --zone=$ZONE

5. Make sure to use the gke-gcloud-auth-plugin, which is needed for continued use of kubectl. You can install it by running the following commands. Make sure to replace your GKE cluster name and zone, as well as your Project ID.

#
    sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
    echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc
    source ~/.bashrc
    gcloud container clusters get-credentials orca-cluster-321 --internal-ip --project=$DEVSHELL_PROJECT_ID --zone $ZONE
    
6. Install kubectl for running kubernetes commands:
#
    sudo apt-get install kubectl 

7. Deploy the application.
This deploys an application that listens on port 8080 that can be exposed using a basic load balancer service for testing.

#
    kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0