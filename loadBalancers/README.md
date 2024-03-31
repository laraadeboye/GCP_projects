# HOW TO SET UP NETWORK AND HTTP LOAD BALANCERS ON VIRTUAL MACHINES IN GOOGLE CLOUD

The purpose of this hands-on is to learn how to set up network and http load balancers on google gloud from the google cloudshell using the gcloud CLI.

 &nbsp;




## Overview
A Load balancer helps to spread traffic accross multiple instances of your application. This helps to reduce the risk that a single server will be overloaded and thereby improving performance.

#### Types of Load balancers

- Network Load balancer
- Application Load balancer

To choose a load balancer, you must determine the type of traffic expected for your application. Application Load balancers support HTTP/HTTPS traffic while Network Load Balancers are for TLS offloading at scale, UDP traffic or if you need to expose your client IP addresses to your applications.
## Learning outcomes
In this walk-through, we will:
- set up a network load balancer
- set up an HTTP load balancer
- understand the differences between network load balancers and HTTP load balancers. 
## Outline
Step 0: Create a project

Step 1: Set the default region and zone for all resources

Step 2: Create multiple instances of webservers

Step 3: Configure the network load balancer

Step 4: Send traffic to your instances to verify

Step 5: Create an HTTP load balancer

Step6: Test the traffic sent to your instances
## Step 0: Create a project and activate google cloud shell
  1. You can create a project from the console, command line or API.
Open your google cloud account. 
Navigate to your cloudshell and launch a cloudshell instance
Create a project by running the following command in the cloudshell terminal:

##
        gcloud projects create PROJECT_ID

or
##
        gcloud projects create PROJECT_ID --organization=ORGANIZATION_ID


  2. On your google cloudshell,
optionally list the project ID  to verify an existing project by running the following command:
##
        gcloud config list project

Click **authorize** if prompted.


 &nbsp;


## Step 1: Set the default region and zone for all resources
We will be using the following details:

- region: **us-east4**
- zone: **us-east4-a**
_(Be free to use any region or zone you desire)_

  1. Set the default region:
##
        gcloud config set compute/region us-east4
  2. Set the default zone:
##
        gcloud config set compute/zone us-east4-a


 &nbsp;
 
   _Tip_:
  _You can run both commands in a single line by separating them with a semi-colon:_
##
        gcloud config set compute/region us-east4;gcloud config set compute/zone us-east4-a
 &nbsp;

## Step 2: Create multiple instances of webservers
We would create 3 instances of VMs and install Apache on them. We will also add a firewall rule that allows HTTP traffic to reach the instances.
We will use the following parameters:


- zone: **us-east4-a**
- tags: **network-vm-tag**
- machine-type: **e2-small**
- image-family: **debian-11**
- image-project: **debian-cloud**
- firewall: **vm-firewall-network**
- metadata: 
``` #!/bin/bash 
          apt-get update
          apt-get install apache2 -y
          service apache2 restart
          echo "<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
  ```



1. Run the following code to create three VM instances. Replace the `[name]` with vm1, vm2, vm3 respectively:
##
    gcloud compute instances create [name] \
    --zone=us-east4-a \
    --tags=network-vm-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "<h3>Web Server: [name]</h3>" | tee /var/www/html/index.html'

2. Create a firewall rule to allow external traffic to the VM instances:
##
        gcloud compute firewall-rules create vm-firewall-network \
        --target-tags network-lb-tag --allow tcp:80

3. Verify that your instances exist. Copy their external IP in a notepad on your system:
##
        gcloud compute instances list
You can verify the running state of your VMs by running the following command, replace the <ip_address> with the external Ips you copied:
##
        curl http://<ip_address>

&nbsp;


## Step 3:  Configure the network load balancer
1. Reserve a static external IP address for your load balancer:
##
        gcloud compute addresses create network-lb-ip \
        --region us-east4

2. Create a HTTP health check:
##
        gcloud compute http-health-checks create http-hlt-check

3. Create a target pool. (A target pool is a group of backend instances that receive incoming traffic from external passthrough Network Load Balancers. The backend instances of a target pool must be in the same region.):
##
        gcloud compute target-pools create vm-pool \
        --region us-east4 --http-health-check http-hlt-check

4. Add the instances to the pool:
##
        gcloud compute target-pools add-instances vm-pool \
        --instances vm1,vm2,vm3
5. Create a forwarding rule:
##
        gcloud compute forwarding-rules create vm-rule \
        --region  us-east4 \
        --ports 80 \
        --address network-lb-ip \
        --target-pool vm-pool

&nbsp;





## Step 4: Send traffic to your instances to verify
1. View the external IP address of the vm-rule forwarding rule used by the network load balancer:
##
        gcloud compute forwarding-rules describe vm-rule --region us-east4

2. Access the external IP address
##
        IPADDRESS=$(gcloud compute forwarding-rules describe vm-rule --region us-east4 --format="json" | jq -r .IPAddress)

3. Show the external IP address
##
        echo $IPADDRESS

4. Use curl to access the external IP address with a while loop. 
##
        while true; do curl -m1 $IPADDRESS; done
_(The above command pings the IP address every 1s. the command alternates randomly among the three instances in the target pool. You can use **Ctrl +C** to exit the loop)_

&nbsp;
## Step 5: Create an HTTP load balancer
To create an HTTP load balancer, your VMs must reside in a managed instance group. To  do this, you must first create the load balancer template:
1. Create the load balancer template named `lb-backend-template`:
##
        gcloud compute instance-templates create lb-backend-template \
        --region=Region \
        --network=default \
        --subnet=default \
        --tags=allow-health-check \
        --machine-type=e2-medium \
        --image-family=debian-11 \
        --image-project=debian-cloud \
        --metadata=startup-script='#!/bin/bash
         apt-get update
         apt-get install apache2 -y
         a2ensite default-ssl
         a2enmod ssl
         vm_hostname="$(curl -H "Metadata-Flavor:Google" \
         http://169.254.169.254/computeMetadata/v1/instance/name)"
         echo "Page served from: $vm_hostname" | \
         tee /var/www/html/index.html
         systemctl restart apache2'
   
   

The metadata installs apache2, enables ssl for the webpages, queries metadata (https://cloud.google.com/compute/docs/metadata/querying-metadata) and restarts apache

2. Create a managed instance group named `lb-backend-group` from the template you created in the previous step:
##
        gcloud compute instance-groups managed create lb-backend-group 
        --template=lb-backend-template --size=2 --zone=us-east4-a

The purpose of the managed instance group is to allow you to operate apps on multiple identical virtual machines. It is useful for autoscaling, autohealing, deployment of machines in multiple regions or zones and automatic updating of VMs.

3. Create the firewall rule named `fw-allow-health-check`:
##
        gcloud compute firewall-rules create fw-allow-health-check \
        --network=default \
        --action=allow \
        --direction=ingress \
        --source-ranges=130.211.0.0/22,35.191.0.0/16 \
        --target-tags=allow-health-check \
        --rules=tcp:80
The ingress rules allows traffic from Google cloud health checking systems in the above source ranges.

4. Create a global static external IP address named `lb-ipv4`
##
        gcloud compute addresses create lb-ipv4 \
        --ip-version=IPV4 \
        --global
5. Verify that the IPV4 address was reserved:
##
        gcloud compute addresses describe lb-ipv4 \
        --format="get(address)" \
        --global
6. Create a health check for the load balancer named `http-hlt-check`
##
        gcloud compute health-checks create http http-hlt-check \
        --port 80
7. Create a backend service named `web-backend-service`:
##
        gcloud compute backend-services create web-backend-service \
        --protocol=HTTP \
        --port-name=http \
        --health-checks=http-hlt-check \
        --global
8. Add your managed instance group as the backend to the backend service:
##
        gcloud compute backend-services add-backend web-backend-service \
        --instance-group=lb-backend-group \
        --instance-group-zone=us-east4-a \
        --global

9. Create a URL map named `web-map-http` to route the incoming request to the default backend service.:
##
        gcloud compute url-maps create web-map-http \
        --default-service web-backend-service
10. Create a target HTTP proxy named `http-lb-proxy` to route requests to your URL map:
##
        gcloud compute target-http-proxies create http-lb-proxy \
        --url-map web-map-http
11. Create a global forwarding rule named `http-content-rule` to route incoming requests to the proxy:
##
        gcloud compute forwarding-rules create http-content-rule \
        --address=lb-ipv4 \
        --global \
        --target-http-proxy=http-lb-proxy \
        --ports=80

&nbsp;
## Step 6: Test the traffic sent to your instances
1. Visit the console and search for **Network services**, then click on **Load balancing**
2. Click on the newly created load balancer, `web-map-http`
3. Confirm that the VMs are healthy from the **Backend** section.
4. Get the ip address of the load balancer visit a web-browser or run the following command in the terminal:
##
        curl http://IP_ADDRESS/, replacing IP_ADDRESS with the load balancer's address.

5. The web browser should render a content showing the name of the instance that served the page.

&nbsp;

## CONCLUSION
In conclusion, we have been able to set up a network load balancer and http load balancer for our three VMs using the gcloud CLI.