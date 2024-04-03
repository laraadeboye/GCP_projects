# DEPLOYING A HELLO APP ON GKE

The purpose of this hands-on lab is to practice deploying a containerized application on the Google Kubernetes Engine (GKE)

&nbsp; 

## Overview
Kubernetes helps to automate the management of containerized applications. GKE is a managed kubernetes engine on Google cloud. It provides advanced management features like load balancing, node pools, automatic scaling, automatic upgrades, node auto-repair, and logging and monitoring.
More information can be found in the [google documentation](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)

&nbsp; 
## Outline
Step 0: Activate google cloudshell

Step 1: Create a GKE cluster

Step 2: Get auth details for the cluster

Step 3: Deploy an application to the cluster

Step 4: Delete the cluster


&nbsp; 
## Step 0: Activate google cloudshell

1. We will open cloudshell from the google console then optionally list the active account name with the following command. (This step will verify that a project has been created. If a project has not been created, we must create one.):
#
    gcloud auth list

2. We will be prompted to click **Authorize**


3. We will set the environmental variables for the region and zone we want to use with the command:
#
    export REGION=us-east4

#
    export ZONE=us-east4-a
If it is successfully run, no output will be seen. We will move to the next step.


4. We will then set a default compute region and zone with the command:

#
    gcloud config set compute/region $REGION; gcloud config set compute/zone $ZONE

INSERT IMAGE

 &nbsp; 

## Step 1: Create a GKE cluster
1. To create a cluster named `lab-cluster`, we will run the command:

#
    gcloud container clusters create --machine-type=e2-medium --zone=$ZONE lab-cluster

It may take close to 10 minutes for the cluster to be created:
PICTURE



&nbsp; 
## Step 2: Get auth details for the cluster
1. Next we will get the authentication details of the cluster by running the command below:
#
    gcloud container clusters get-credentials lab-cluster

PICTURE

&nbsp; 
## Step 3: Deploy an application to the cluster
1. Since the cluster has been created we can now run `kubectl` commands.
We will deploy an application named `hello-app` located in the google container registry. The version of the app is 1.0. We will name the deployment `hello-server` 
#
    kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0

PICTURE

2. We will also create a service of type `loadbalancer` on port `8080` that will help expose our application to external traffic with the following command:

#
    kubectl expose deployment hello-server --type=LoadBalancer --port 8080

PICTURE

We can confirm that both the deployment and service have been created by running the following commands respectively:
#
    kubectl get deployment

#
    kubectl get service

PICTURE

4. The application can now be viewed from a webserver with the command below (Replace [EXTERNAL-IP]with the external ip for hello-server ):
#
    http://[EXTERNAL-IP]:8080


## Step 4: Delete the cluster
We can the delete the cluster by running the following command.(If we leave clusters running we will incure costs). 
It may take several minutes for the cluster to be deleted. Grab a coffee while you wait. :
#
    gcloud container clusters delete lab-cluster

## Conclusion

We have been able to deploy a simple hello app on a managed kubernetes cluster in google cloud.