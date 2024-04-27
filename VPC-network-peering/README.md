
# HOW TO CREATE VPC NETWORK PEERING ACROSS TWO PROJECTS IN GOOGLE CLOUD

![VPC network peering](https://github.com/laraadeboye/GCP_projects/blob/main/VPC-network-peering/images/Screenshot%202024-04-27%20173123.png)
<figcaption>VPC network peering image from google </figcaption>


## OVERVIEW

Imagine you have secure, private networks (VPCs) in Google Cloud. VPC Network Peering lets you connect these VPCs directly, allowing resources within them to talk to each other like they're on the same network. This works even if the VPCs are in different projects or belong to entirely different organizations!

#### Benefits of VPC Network Peering:

- **Private Connections**: Keep your data traffic secure by avoiding the public internet.
- **Simplified Management**: Consolidate services across your organization's VPCs, regardless of project structure.
- **Reduced Costs**: Communicate using internal IPs, potentially saving on egress bandwidth charges.
- **Improved Performance**: Enjoy lower latency compared to public connections.

#### Use Cases:

- **Shared Services within an Organization**: VPC Network Peering allows different departments (with separate VPCs) to access critical services hosted in another VPC.
- **Cross-Organization Collaboration**: Securely connect to services provided by a partner organization, all within a private network.


*Note*:
`**No Need to Worry About Project Details**: When setting up peering within your organization, project names are enough. Google Cloud will automatically identify the organization based on that information.`


## OUTLINE
- Prerequisites
- Create a custom network in two projects
- Set up a VPC Network peering session
## Step 0. Prerequisites 
1. Within this account, two projects had been created each with project id
- qwiklabs-gcp-01-ad8775fbae89 (PROJECT-A)

- qwiklabs-gcp-01-97d557cfb486 (PROJECT-B)

From the console we can view the projects by clicking on the project drop-down, then select **ALL**

IMAGE

2. We will configure cloudshell to set the individual project IDs. Open cloudshell from the console then click on the + sign on cloudshell tab to open two tabs within the cloudshell console. In the first tab, run the following command:

*PROJECT-A*
#
    gcloud config set project qwiklabs-gcp-01-ad8775fbae89

In the second tab run the following command:

*PROJECT-B*
#
    gcloud config set project qwiklabs-gcp-01-97d557cfb486



## Step 1. Create a custom network in two projects

#### **FOR PROJECT A**
We will navigate to the first cloudshell tab that we configured for PROJECT A and create a custom network having a subnet and a VM instance.

1. To create the custom network, run the following command:
#
    gcloud compute networks create network-a --subnet-mode custom

2.  We create a subnet within the `us-central1` region with a network range of `10.0.0.0/16`:
#
    gcloud compute networks subnets create network-a-subnet --network network-a \
    --range 10.0.0.0/16 --region us-central1

3. Then, we create a VM instance named `vm-a` within the subnet we created:
#
    gcloud compute instances create vm-a --zone us-central1-a --network network-a --subnet network-a-subnet --machine-type e2-small

4. We will enable firewall rules for the custom network `network-a`  to allow SSH on port 22 and icmp. ICMP is a network layer protocol mainly used to determine whether or not data is reaching its intended destination in a timely manner.
#
    gcloud compute firewall-rules create network-a-fw --network network-a --allow tcp:22,icmp

#### **FOR PROJECT B**
We will switch to the second cloudshell tab that we configured for project B and also create a custom network having one subnet and a VM instance created within the subnet.

1. We will create the custom network by running the following command:
#
    gcloud compute networks create network-b --subnet-mode custom

2. The subnet will also be craeted in the `us-central1` region with the network range of `10.8.0.0/16`:
#
    gcloud compute networks subnets create network-b-subnet --network network-b \
    --range 10.8.0.0/16 --region us-central1

3. We will create the VM instance by running the following command:
#
    gcloud compute instances create vm-b --zone us-central1-a --network network-b --subnet network-b-subnet --machine-type e2-small

4. To enable SSH on port 22 and icmp:
#
    gcloud compute firewall-rules create network-b-fw --network network-b --allow tcp:22,icmp

## Step 2. Set up a VPC Network peering session

We will establish a VPC Network Peering between network-a in PROJECT A and network-b in PROJECT B. The network peering can only be successful if it is separately configured in the individual networks

IMAGE

Let's start with **PROJECT A**

From the console, go to VPC Network Peering and create a connection with network b. We will name this connection `peer-ab`. Also set the **Your VPC network** (which is the network you want to peer) as `network-a`. the **Peered VPC network** is `In another project`. We will also paste the project ID of the second project as shown (in the image below). Remember that our project ID for the second project is `qwiklabs-gcp-01-97d557cfb486`. You will need to type in the **VPC network name** of the second network (network-b)
Finally, Click **Create**.

*Your project ID will likely be different if you are using this project as a guide.*

For now, peering state remains INACTIVE because there is no matching configuration in network-b in project-B. You should observe the Status message, *Waiting for peer network to connect*.

IMAGE

**PROJECT B**
From the console, go to VPC Network Peering and create a connection with network a. 
- **Name**:`peer-ab`
- **Your VPC network** ( the network you want to peer): `network-b`. t - **Peered VPC network**: `In another project`.  
- **Project ID**: `qwiklabs-gcp-01-ad8775fbae89`. 
- **VPC network name** (Type the name of the second network): network-a
Click **Create**.

IMAGE

You will observe that the VPC Network Peering becomes active because traffic is now flowing between the two VPCs.

In the individual cloudshell consoles, if you run the following command, replacing [PROJECT ID] with the respective project ids, you will see the lists of routes.

#
    gcloud compute routes list --project [PROJECT ID]

PROJECT A: qwiklabs-gcp-01-ad8775fbae89


IMAGE

PROJECT B: qwiklabs-gcp-01-97d557cfb486

IMAGE
## Step 3. Test the connectivity
In this step, we will tet the network connectivity by copying the internal IP addresses for voth vm-a and vm-by

Mine are as follows:
- vm -a 10.0.0.2
- vm -b 10.8.0.2

You can SSH into the two VMs from their individual consoles or from there cloudshell tabs

In PROJECT A run the following command:
#
    ping -c 5 <INTERNAL_IP_OF_VM_B>

See the output below:

IMAGE

In PROJECT B run the following command:
#
    ping -c 5 <INTERNAL_IP_OF_VM_A>

See the output below:

IMAGE

## Conclusion
We have been able to set up a VPC Network Peering connection between two different VPCs in different projects. 
