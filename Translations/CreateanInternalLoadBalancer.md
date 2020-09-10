# LAB: Create an Internal Load Balancer

## OBJECTIVES:

In this lab you learn how to perform the following tasks:
    - Create HTTP and health check firewall rules
    - Configure two instance templates
    - Create two managed instance groups
    - Configure and test an internal load balancer

## STEPS:

1. Configure HTTP and health check firewall rules
   
    1. Explore the my-internal-app network:
    
        gcloud compute networks describe my-internal-app

    2. Create the HTTP firewall rule:
   
        gcloud compute firewall-rules create app-allow-http --direction=INGRESS --network=my-internal-app --priority=1000 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:80 --target-tags=lb-backend

    3. Create the health check firewall rules:
        
        gcloud compute firewall-rules create app-allow-health-check --direction=INGRESS --network=default --priority=1000 --source-ranges=130.211.0.0/22,35.191.0.0/16 --action=ALLOW --rules=tcp --target-tags=lb-backend

2. Configure instance templates and create instance groups

    1. Configure the instance templates:

        gcloud compute instance-templates create instance-template-1 --machine-type=n1-standard-1 --subnet=subnet-a --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --region=us-central1 --tags=lb-backend

    2. Configure the next instance template:
   
        gcloud compute instance-templates create instance-template-2 --machine-type=n1-standard-1 --subnet=subnet-b --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh --region=us-central1 --tags=lb-backend

    3. Create the managed instance groups
        
        Instance group 1:

        gcloud compute instance-groups managed create instance-group-1 --base-instance-name=instance-group-1 --template=instance-template-1 --size=1 --zone=us-central1-a

        gcloud compute instance-groups managed set-autoscaling "instance-group-1" --zone "us-central1-a" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"

        Instance group 2:
        
        gcloud compute instance-groups managed create instance-group-2 --base-instance-name=instance-group-2 --template=instance-template-2 --size=1 --zone=us-central1-b

        gcloud compute instance-groups managed set-autoscaling "instance-group-2" --zone "us-central1-b" --cool-down-period "45" --max-num-replicas "5" --min-num-replicas "1" --target-cpu-utilization "0.8" --mode "on"


    4. Verify the backends:
   
        Verify that VM instances are being created in both subnets and create a utility VM to access the backends' HTTP sites:

        gcloud compute instances create utility-vm --zone=us-central1-f --machine-type=f1-micro --subnet=subnet-a --private-network-ip=10.10.20.50

    5. For utility-vm, click SSH to launch a terminal and connect:

        gcloud compute ssh utility-vm

        - To verify the welcome page for instance-group-1-xxxx, run the following command:

            curl 10.10.20.2

        - To verify the welcome page for instance-group-2-xxxx, run the following command:

            curl 10.10.30.2

        - Close the SSH terminal to utility-vm:

            exit


3. Start the configuration:

    1. Create a Load Balancer

        1.  Regional TCP health check:
          
                gcloud compute health-checks create tcp my-ilb-health-check --region=us-central1 --port=80

        3. Create the backend service:
   
                gcloud compute backend-services create my-tcp-ilb --load-balancing-scheme=internal --protocol=tcp --region=us-central1   --health-checks=my-ilb-health-check --health-checks-region=us-central1  

        4. Add the two instance groups to the backend service:
   
                gcloud compute backend-services add-backend my-tcp-ilb --region=us-central1 --instance-group=instance-group-1 --instance-group-zone=us-central1-a

                gcloud compute backend-services add-backend my-tcp-ilb --region=us-central1 --instance-group=instance-group-2 --instance-group-zone=us-central1-b

        5.  Create a forwarding rule for the backend service:
   
                gcloud compute forwarding-rules create my-ilb-ip --region=us-central1 --load-balancing-scheme=internal --network=my-internal-app --subnet=subnet-b --address=10.10.30.5 --ip-protocol=TCP --ports=80 --backend-service=my-tcp-ilb --backend-service-region=us-central1

4. Test the Internal Load Balancer

    Verify that the my-ilb IP address forwards traffic to instance-group-1 in us-central1-a and instance-group-2 in us-central1-b.

    1. For utility-vm, click SSH to launch a terminal and connect:

        gcloud compute ssh utility-vm

    2. To verify that the Internal Load Balancer forwards traffic, run the following command:

        curl 10.10.30.5

    Run the same command a couple more times. You should be able to see responses from instance-group-1 in us-central1-a and instance-group-2 in us-central1-b.

