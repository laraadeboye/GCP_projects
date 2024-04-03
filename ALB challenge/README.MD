
# Project Title: Implement Load Balancing on Compute Engine: Challenge Lab

This is a project from one of Google's essentials courses.

### Challenge scenario
You have started a new role as a Junior Cloud Engineer for Jooli, Inc. You are expected to help manage the infrastructure at Jooli. Common tasks include provisioning resources for projects.

You are expected to have the skills and knowledge for these tasks, so step-by-step guides are not provided.

Some Jooli, Inc. standards you should follow:

- Create all resources in the default region or zone, unless otherwise directed. 
- The default region is **REGION**, and the default zone is **ZONE**.
- Naming normally uses the format team-resource; for example, an instance could be named `nucleus-webserver1`.
- Allocate cost-effective resource sizes. Projects are monitored, and excessive resource use will result in the containing project's termination (and possibly yours), so plan carefully. 
This is the guidance the monitoring team is willing to share: unless directed, use `e2-micro` for small Linux VMs, and use `e2-medium` for Windows or other applications, such as Kubernetes nodes.

#### Your challenge
As soon as you sit down at your desk and open your new laptop, you receive several requests from the Nucleus team. Read through each description, and then create the resources.

### Tasks
- Create a jumphost instance
- Configure an http load balancer

## TASK 1: Create a jumphost instance

You will use this instance to perform maintenance for the project.

Requirements:

Name the instance **nucleus-jumphost-376**.
Create the instance in the **europe-west4-a**  zone.
Use an `e2-micro` machine type.
Use the default image type (Debian Linux).

# My answer
A jumphost also called a bastion host. It provides an external point of entry into a Virtual Private Cloud (VPC) network that contains VMs that don't have external IP addresses. When using a bastion host, connect to the bastion host first, and then connect to the target VM. From the jumphost, we can SSH into the target machines.



1. In the Google Cloud console, activate Cloud Shell. First,set the environment variables of the region and zone as follows:
##
    export REGION=europe-west4; export ZONE=europe-west4-a

Then set the default region and zone.
##
    gcloud config set compute/region $REGION;gcloud config set compute/zone $ZONE    

2. Create a compute engine VM named `nucleus-jumphost-376`:

##
    gcloud compute instances create nucleus-jumphost-376 \
    --zone=$ZONE \
    --machine-type=e2-micro 
       
        
        

    

## TASK 2: Set up an HTTP Load balancer

You will serve the site via nginx web servers, but you want to ensure that the environment is fault-tolerant. Create an HTTP load balancer with a managed instance group of 2 nginx web servers. Use the following code to configure the web servers; the team will replace this with their own configuration later.

##
        cat << EOF > startup.sh
        #! /bin/bash
        apt-get update
        apt-get install -y nginx
        service nginx start
        sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /       var/www/html/index.nginx-debian.html
        EOF  


You need to:

- Create an instance template. Don't use the default machine type. 
- Make sure you specify e2-medium as the machine type.
- Create a managed instance group based on the template.
- Create a firewall rule named as **Firewall rule** to **allow traffic (80/tcp)**.
- Create a health check.
- Create a backend service and add your instance group as the backend to the backend service group with named **port (http:80)**.
- Create a URL map, and target the HTTP proxy to route the incoming requests to the default backend service.
- Create a target HTTP proxy to route requests to your URL map
- Create a forwarding rule.

# My Answers
1. First, We will create a load balancer template named `nucleus-lb-template`.

##
    gcloud compute instance-templates create nucleus-lb-template \
    --region=$REGION \
    --network=default \
    --subnet=default \
    --tags=allow-health-check \
    --machine-type=e2-medium \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script=cat << EOF > startup.sh
    #! /bin/bash
    apt-get update
    apt-get install -y nginx
    service nginx start
    sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
    EOF


The script will update the vm, install nginx and replace nginx with 'Google Cloud Platform- $HOSTNAME


2. We will create a managed-instance group named `nucleus-webserver`:
##
    gcloud compute instance-groups managed create nucleus-webserver --template=nucleus-lb-template --size=2 --zone=$ZONE

Set the named-port with this command:
##
    gcloud compute instance-groups set-named-ports nucleus-webserver --named-ports http:80 --zone=$ZONE

The managed instance group will provision two VMs as stated by the size.

3. We will create a firewall rule named `allow-tcp-rule-969`
##
    gcloud compute firewall-rules create allow-tcp-rule-969 \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80
          

4. We will create a global static ip for the load balancer named `nucleus-lb-ip`:
##
    gcloud compute addresses create nucleus-lb-ip \
    --ip-version=IPV4 \
    --global



To verify the presence of the global static ip run:

##
    gcloud compute addresses describe nucleus-lb-ip \
    --format="get(address)" \
    --global



5. We will create a health check name `nucleus-health-check`:
##
    gcloud compute  health-checks create http nucleus-health-check \
    --port 80

6. We will create a backend service named `nucleus-backend-service` and add our managed instance group to it:
##
    gcloud compute backend-services create nucleus-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=nucleus-health-check \
    --global

To add our managed instance group to the backend service run:
##
    gcloud compute backend-services add-backend nucleus-backend-service \
    --instance-group=nucleus-webserver \
    --instance-group-zone=$ZONE \
    --global

7. We will create a URL map named `nucleus-map-http`, and target the HTTP proxy named `nucleus-http-lb-proxy` to route the incoming requests to the default backend service.
##
    gcloud compute url-maps create nucleus-map-http \
    --default-service nucleus-backend-service


8. We will create a target HTTP proxy to route request to the URL map run:
##
    gcloud compute target-http-proxies create nucleus-http-lb-proxy \
    --url-map nucleus-map-http

9. We create a forwarding rule named `nucleus-http-rule` to route incoming requests to the HTTP proxy:
##
    gcloud compute forwarding-rules create nucleus-http-rule \
    --address=nucleus-lb-ip \
    --global \
    --target-http-proxy=nucleus-http-lb-proxy \
    --ports=80


10. We will test the traffic sent to our instances from the console by navigating to **Network services**. There, we will click on **Load balancing** and check state of our instances whether they are healthy. When they become healthy, we will visit the static ip of the load balancer from our browser.

