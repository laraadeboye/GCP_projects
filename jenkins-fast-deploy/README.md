# DEPLOY JENKINS CI ENVIRONMENT FROM GOOGLE MARKETPLACE IN MINUTES

The purpose of this hands-on lab is to practice building a deployment using Marketplace.

&nbsp; 

## Overview
Google Marketplace offers production-ready stacks , products, datasts and services to speed up development. It reduces the time spent installing software.

Jenkins is an open-source continuous integration environment. You can use it to define jobs in Jenkins that can perform tasks such as running a scheduled build of software and backing up data.

&nbsp; 
## Outline

Step 1: Use Marketplace to build a Jenkins Continuous Integration environment.

Step 2: Verify that you can manage the service from the Jenkins UI.

Step 3: Administer the service from the Virtual Machine host through SSH.
&nbsp; 

## Prerequisites
* You need an active google cloud account for this hands-on.
&nbsp; 

## Step 1: Use Marketplace to build a Jenkins Continuous Integration environment.



1. Navigate to Marketplace. 
* From google cloud console, on the **Navigation menu**, click **Marketplace**
 
* Search for **Bitnami package for Jenkins**

* Click on the deployment.

  [search for Bitnami package for Jenkins](jenkins-fast-deploy/jenkins-deploy-Images/Screenshot 2024-04-05 101519.png)
&nbsp; 

2. Launch Jenkins. 
* Click on **Get Started and verify the deployment. Accept the terms of service and click **Deploy**. You may need to click **Deploy** again.

Deployment Manager, agoogle cloud service uses templates witten in YAML, python and Jinja2 to automate the deployment of the resources. It may take a few minutes. Wait until **jenkins-1** has been deployed

3. Navigate to the compute engine on the console and click **jenkins-1 VM**, use the public IP to visit the jenkins UI on another tab

  [Jenkins deployment in progress](https://github.com/laraadeboye/GCP_projects/blob/main/jenkins-fast-deploy/jenkins-deploy-Images/Screenshot%202024-04-05%20101909.png)
  &nbsp; 
  [Click Jenkins-1 VM](https://github.com/laraadeboye/GCP_projects/blob/main/jenkins-fast-deploy/jenkins-deploy-Images/Screenshot%202024-04-05%20102221.png)
&nbsp; 


## Step 2: Verify that you can manage the service from the Jenkins UI.

1. Login to Jenkins.
* Note the **Admin user** and the **Admin password** values and add them to a text editor of your choice.
* Visit the site on another browser tab. Reload multiple times if you get an error. Proceed to login with the **Admin user** and **Admin password** values.
* If prompted, proceed to install *the suggested plugins* then click *Restart*

  [Jenkins login uI](https://github.com/laraadeboye/GCP_projects/blob/main/jenkins-fast-deploy/jenkins-deploy-Images/Screenshot%202024-04-05%20102452.png)
&nbsp;
  [Jenkins console](https://github.com/laraadeboye/GCP_projects/blob/main/jenkins-fast-deploy/jenkins-deploy-Images/Screenshot%202024-04-05%20102621.png)

## Step 3: Step 3: Administer the service from the Virtual Machine host through SSH.

1. Click the **Navigation menu** and choose **Deployment** on the cloud console.

2. Click **jenkins-1** and **SSH** to connect to the jenkins server

3. Verify the operation of jenkins server from the SSH window by running the following commands:

**Stop jenkins server**
#
    sudo /opt/bitnami/ctlscript.sh stop

**Start jenkins server**
#
    sudo /opt/bitnami/ctlscript.sh restart

 [Administer JEnkins from the SSH window](https://github.com/laraadeboye/GCP_projects/blob/main/jenkins-fast-deploy/jenkins-deploy-Images/Screenshot%202024-04-05%20103257.png)
&nbsp;

## Conclusion
We have been able to deploy a jenkins CI environment in minutes using solutions from Google Marketplace.
