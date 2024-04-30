# How to set up a private Kubernetes Cluster on Google Cloud

Kubernetes Engine (GKE) offers private clusters for enhanced security. These clusters isolate your workloads by keeping the master (control plane) hidden from the public internet. Nodes, the workhorses running your applications, also operate within a private network, using internal IP addresses instead of public ones. This isolation creates a secure environment for your applications.

### Prerequisites
- experience creating and launching Kubernetes Clusters and be 
- CIDR blocks

## Outline
Step 0. Set the region and zone

Step 1. Create private cluster

Step 2. View your subnet and secondary address ranges

Step 3 Enable the master authorized networks

Step 4 Verify 

Step 5. Clean up



## Step 0.  Set the region and zone

Create the environmental variable for the region and zone and config set them.

#
    export REGION=europe-west4;export ZONE=europe-west4-b

#
    gcloud config set compute/region $REGION;gcloud config set compute/zone $ZONE
## Step 1. Create private cluster
We will create a cluster named `private-cluster` with a CIDR range of 172.16.0.16/28 for the masters(Note that ypou must specify a CIDR range of /28 for the VMS that run the master):

#
    gcloud beta container clusters create private-cluster \
    --enable-private-nodes \
    --master-ipv4-cidr 172.16.0.16/28 \
    --enable-ip-alias \
    --create-subnetwork ""


The cluster will take some time to be created.
##   Step 2. View your subnet and secondary address ranges  

To list the subnet in our network. Run the command below: (We created the cluster in the default network).

#
    gcloud compute networks subnets list --network default 

The output shows the subnet for your cluster. Mine is
`gke-private-cluster-subnet-2176dcd1`

To get the information about your automatically created subnet, run:
#
    gcloud compute networks subnets describe [SUBNET_NAME] --region=$REGION

Replace [SUBNET_NAME] with the subnet for your cluster.

Notice that privateIPGoogleAccess is set to true. This enables your cluster hosts, which have only private IP addresses, to communicate with Google APIs and services.
IMAGE
## Step 3 Enable the master authorized networks 
 At this point, the only IP addresses that have access to the master are the addresses in these ranges:

The primary range of your subnetwork. This is the range used for nodes.
The secondary range of your subnetwork that is used for pods.
To provide additional access to the master, you must authorize selected address ranges.

- Create a VM instance named `source-instance`to check the connectivity to Kubernetes clusters:
#
    gcloud compute instances create source-instance --zone=$ZONE --scopes 'https://www.googleapis.com/auth/cloud-platform'


- Get the external IP of the `source-instance` with the following command:
#
    gcloud compute instances describe source-instance --zone=$ZONE | grep natIP

- Run the following to Authorize your external address range, replacing [MY_EXTERNAL_RANGE] with the CIDR range of the external addresses from the previous output (your CIDR range is natIP/32). With CIDR range as natIP/32, we are allowlisting one specific IP address:

#
    gcloud container clusters update private-cluster \
    --enable-master-authorized-networks \
    --master-authorized-networks [MY_EXTERNAL_RANGE]
## Step 4 Verify   
Now that you have access to the master from a range of external addresses, you'll install kubectl so you can use it to get information about your cluster. For example, you can use kubectl to verify that your nodes do not have external IP addresses.

1. SSH into source-instance:
#
    gcloud compute ssh source-instance --zone=$ZONE

2. Install kubectl in the ssh shell:
#
    sudo apt-get install kubectl 

3. Create the environment variable of the zone and Configure access to the Kubernetes cluster from SSH shell with:

#
    export ZONE=europe-west4-b
#
    sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin gcloud container clusters get-credentials private-cluster --zone=$ZONE


    

4. Verify that the cluster nodes do not have external IP addresses:
#
    kubectl get nodes --output yaml | grep -A4 addresses

or with the following command:

#
    kubectl get nodes --output wide

5. exit the SSH shell. (Ctrl+D)
    
## Step 5. Clean up
Delete the cluster by running the following command
#
    gcloud container clusters delete private-cluster --zone=$ZONE
## Step 6: Private kubernetes cluster that uses a custom subnet

1. To create a subnet named `my-subnet` with secondary ranges service range `10.0.32.0/20`, pod range:`10.4.0.0/14` . Type the following command:

#
    gcloud compute networks subnets create my-subnet \
    --network default \
    --range 10.0.4.0/22 \
    --enable-private-ip-google-access \
    --region=$REGION \
    --secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14

2. Next, we will create a private cluster that uses the subnet we created:

#
    gcloud beta container clusters create private-cluster2 \
    --enable-private-nodes \
    --enable-ip-alias \
    --master-ipv4-cidr 172.16.0.32/28 \
    --subnetwork my-subnet \
    --services-secondary-range-name my-svc-range \
    --cluster-secondary-range-name my-pod-range \
    --zone=$ZONE

3. We are still using the external IP of the source instance we previously created. If you have deleted it, follow the previous steps to create another instance.
To retrieve the external IP:
#
    gcloud compute instances describe source-instance --zone=$ZONE | grep natIP

4. We will run the command below to  Authorize our external address range, replacing [MY_EXTERNAL_RANGE] with the CIDR range of the external addresses from the previous output (our CIDR range is natIP/32). With CIDR range as natIP/32, we are allowlisting one specific IP address:
#
    gcloud container clusters update private-cluster2 \
    --enable-master-authorized-networks \
    --zone=$ZONE \
    --master-authorized-networks [MY_EXTERNAL_RANGE]

5. We will then SSH into our source-instance once again with the `gcloud compute ssh` command.

6. We will configure access to the kubernetes cluster from the SSH shell with:
#
    gcloud container clusters get-credentials private-cluster2 --zone=$ZONE

7. We can also verify that our cluster nodes do not have external IP addresses with the command below:
#
    kubectl get nodes --output yaml | grep -A4 addresses



## Conclusion
We successfully created a private kubernetes cluster that could only be accessed from one IP