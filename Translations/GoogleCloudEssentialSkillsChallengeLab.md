# LAB: Google Cloud Essential Skills: Challenge Lab

## OBJECTIVES:

To deploy the site in the public cloud

    -   Create a Linux virtual machine instance

    -   Enable public access to VM instance

    -   Running basic Apache Web Server

    -   Test your server


## STEPS:

1. Running a Basic Apache Web Server:
   
   1. Create a Linux VM Instance

        gcloud compute instances create apache --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default --tags=http-server

    2. Enable Public Access to VM Instance

        gcloud compute firewall-rules create my-rule-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

    3. Running a Basic Apache Web Server

        gcloud compute ssh apache

        1. Install Apache

            sudo apt update && sudo apt -y install apache2

        2. Overwrite the Apache web server default web page with the following command:
   
            echo '<!doctype html><html><body><h1>Hello World!</h1></body></html>' | sudo tee /var/www/html/index.html




2. Test that your instance is serving traffic on its external IP.

    1. To view instance external IP address
   
        gcloud compute instances list

    2. Copy the external IP for your instance under the External IP column. In a browser, navigate to;
   
        http://[EXTERNAL_IP]
