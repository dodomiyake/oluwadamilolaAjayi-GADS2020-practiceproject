# LAB: VPC Networking Fundamentals

## OBJECTIVES:

In this lab, you learn how to perform the following tasks:
    - Explore the default VPC network
    - Create an auto mode network with firewall rules
    - Create VM instances using Compute Engine
    - Explore the connectivity for VM instances

## STEPS:

1.  Explore the default network:

   1.   View the subnets:

            gcloud compute networks subnets list

   2.   View the routes:

            gcloud compute routes list

   3.   View the firewall rules:

            gcloud compute firewall-rules list

   4.   Delete the default network:

            gcloud compute networks delete default

   5. Try to create a VM instance:

            gcloud compute instances create myvm-1



2. Create a VPC network and VM instances:

   1.  Create an auto mode VPC network with Firewall rules:

            gcloud compute networks create mynetwork --subnet-mode=auto --bgp-routing-mode=regional

        firewall rules:

            gcloud compute firewall-rules create mynetwork-allow-icmp --direction=INGRESS --network=mynetwork --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp

            gcloud compute firewall-rules create mynetwork-allow-internal --direction=INGRESS --network=mynetwork --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all

            gcloud compute firewall-rules create mynetwork-allow-rdp --direction=INGRESS --network=mynetwork --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389

            gcloud compute firewall-rules create mynetwork-allow-ssh --direction=INGRESS --network=mynetwork --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22


   2. Create a VM instance in us-central1:

            gcloud compute instances create mynet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=mynetwork

   3.  Create a VM instance in europe-west1:

            gcloud compute instances create mynet-eu-vm --zone=europe-west1-c --machine-type=n1-standard-1 --subnet=mynetwork



3. Explore the connectivity for VM instances:
   1.   Verify connectivity for the VM instances
        A. Connect to mynet-us-vm:

            gcloud compute ssh mynet-us-vm

        B. To test connectivity to mynet-eu-vm's internal IP, run the following command using mynet-eu-vm's internal IP:

            ping -c 3 <Enter mynet-eu-vm's internal IP here>

        C. Repeat the same test, this time using mynet-eu-vm's name:

            ping -c 3 mynet-eu-vm

        D. To test connectivity to mynet-eu-vm's external IP, run the following command using mynet-eu-vm's external IP:

            ping -c 3 <Enter mynet-eu-vm's external IP here>

        E. Remove the allow-icmp firewall rules:

            gcloud compute firewall-rules delete mynetwork-allow-icmp

        F. Return to the mynet-us-vm SSH terminal:

            gcloud compute ssh mynet-us-vm

        G. To test connectivity to mynet-eu-vm's internal IP, run the following command using mynet-eu-vm's internal IP:

            ping -c 3 <Enter mynet-eu-vm's internal IP here>

        H. To test connectivity to mynet-eu-vm's external IP, run the following command using mynet-eu-vm's external IP:

            ping -c 3 <Enter mynet-eu-vm's external IP here>

        I. Remove the allow-internal firewall rules:

            gcloud compute firewall-rules delete mynetwork-allow-internal

        J. Return to the mynet-us-vm SSH terminal:

            gcloud compute ssh mynet-us-vm

        K. To test connectivity to mynet-eu-vm's internal IP, run the following command using mynet-eu-vm's internal IP:

            ping -c 3 <Enter mynet-eu-vm's internal IP here>

        L. Close the SSH terminal:

            exit

        M. Remove the allow-ssh firewall rules:

            gcloud compute firewall-rules delete mynetwork-allow-ssh

        N. For mynet-us-vm, SSH to launch a terminal and connect:

            gcloud compute ssh mynet-us-vm

            "The Connection failed message indicates that you are unable to SSH to mynet-us-vm because you deleted the allow-ssh firewall rule!"
