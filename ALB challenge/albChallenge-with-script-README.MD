# Project Title: Implement Load Balancing on Compute Engine: Challenge Lab


1. In the Google Cloud console, activate Cloud Shell. First,set the environment variables of the region and zone as follows. Export all the values carefully:

##
    export INSTANCE_NAME=nucleus-jumphost-271
 
    export ZONE=us-west1-b
 
    export PORT=8081
 
    export FIREWALL_NAME=allow-tcp-rule-596
   

2. Change the permissions of your script named **challengeScript** with the following command:
(if you cloned this repo, it will be found within the repo folder)

##
    sudo chmod +x challengeScript.sh

3. Execute the script:

##
    ./challengeScript.sh
 
 
The resources will take time to create. Patiently wait.