# GCP Practice Project
## Course: Essential Google Cloud Infrastructure: Foundation

### Module: Virtual Networks
    LAB: VPC Networking
<br/>
<br/>


### Objectives of this lab
<ul>
   <li> Explore the default VPC network </li>
   <li> Create an auto mode network with firewall rules
    <li>Convert an auto mode network to a custom mode network</li>
    <li>Create custom mode VPC networks with firewall rules</li>
    <li>Create VM instances using Compute Engine
    <li>Explore the connectivity for VM instances across VPC networks</li>
</ul>


### Task 1. Explore the default network
### Viewing the subnets

`gcloud compute networks list`

### View the routes

`gcloud compute routes list`

### Viewing the firewall rules

`gcloud compute firewall-rules  list`

### Task 2. Create an auto mode network
<br/>

### Create an auto mode VPC network with firewall rules

`gcloud compute networks create mynetwork --project=qwiklabs-gcp-01-607b9cd16417 --subnet-mode=auto --bgp-routing-mode=regional`

`gcloud compute firewall-rules create mynetwork-allow-icmp --project=qwiklabs-gcp-01-607b9cd16417 --network=projects/qwiklabs-gcp-01-607b9cd16417/global/networks/mynetwork --description=Allows\ ICMP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp`

`gcloud compute firewall-rules create mynetwork-allow-internal --project=qwiklabs-gcp-01-607b9cd16417 --network=projects/qwiklabs-gcp-01-607b9cd16417/global/networks/mynetwork --description=Allows\ connections\ from\ any\ source\ in\ the\ network\ IP\ range\ to\ any\ instance\ on\ the\ network\ using\ all\ protocols. --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all`

`gcloud compute firewall-rules create mynetwork-allow-rdp --project=qwiklabs-gcp-01-607b9cd16417 --network=projects/qwiklabs-gcp-01-607b9cd16417/global/networks/mynetwork --description=Allows\ RDP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 3389. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389`

`gcloud compute firewall-rules create mynetwork-allow-ssh --project=qwiklabs-gcp-01-607b9cd16417 --network=projects/qwiklabs-gcp-01-607b9cd16417/global/networks/mynetwork --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22`

### Create a VM instance in us-central1

`gcloud beta compute --project=qwiklabs-gcp-01-607b9cd16417 instances create mynet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=297531843720-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mynet-us-vm --reservation-affinity=any`

### Create a VM instance in europe-west1

`gcloud beta compute --project=qwiklabs-gcp-01-607b9cd16417 instances create mynet-eu-vm --zone=europe-west1-c --machine-type=n1-standard-1 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=297531843720-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mynet-eu-vm --reservation-affinity=any`

#### Verify connectivity for the VM instances

#### In US VM reach EU VM using internal IP

`ping -c 3 10.132.0.2`


### Task 3. Create custom mode networks
<br/>

### Create the managementnet network

`gcloud compute networks create managementnet --project=qwiklabs-gcp-01-607b9cd16417 --subnet-mode=custom --bgp-routing-mode=regional`


`gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-01-607b9cd16417 --range=10.130.0.0/20 --network=managementnet --region=us-central1`

### Create the privatenet network

`gcloud compute networks create privatenet --subnet-mode=custom`

`gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24`

`gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20`

#### Listing the networks
`gcloud compute networks list`

#### Listing the networks (sorted order)
`gcloud compute networks subnets list --sort-by=NETWORK`

### Create the firewall rules for managementnet

`gcloud compute --project=qwiklabs-gcp-01-607b9cd16417 firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0`


### Create the firewall rules for privatenet

`gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0`

`gcloud compute firewall-rules list --sort-by=NETWORK`

### Create the managementnet-us-vm instance

`gcloud compute instances create managementnet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=privatesubnet-us`

### Create the privatenet-us-vm instance

`gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=f1-micro --subnet=privatesubnet-us`

`gcloud compute instances list --sort-by=ZONE`

#Testing the connections from `mynet-us-vm` using external IPs

`ping -c 3 104.154.243.118 `

`ping -c 3 34.78.146.224`

`ping -c 3 34.68.10.158`

<br/><br/><br/>

## Course: Essential Google Cloud Infrastructure: Foundation
### Module: Virtual Machines
    LAB:  Working with Virtual Machines 
<br/>
<br/>

### Objectives of this lab
<ul>
    <li>Customize an application server</li>
    <li>Install and configure necessary software</li>
    <li>Configure network access</li>
    <li>Schedule regular backups</li>
</ul>

### Task 1: Create the VM

`gcloud beta compute --project=qwiklabs-gcp-04-a4e1aabdb53f instances create mc-server --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default --address=34.67.9.37 --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=506215754375-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/trace.append,https://www.googleapis.com/auth/devstorage.read_write --tags=minecraft-server --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mc-server --create-disk=mode=rw,size=50,type=projects/qwiklabs-gcp-04-a4e1aabdb53f/zones/us-central1-a/diskTypes/pd-ssd,name=minecraft-disk,device-name=minecraft-disk --reservation-affinity=any`

### Task 2: Preparing the data disk
#Create a directory and format and mount the disk
#Formarting the disk
#mount the disk,

`sudo mkdir -p /home/minecraft`

`sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk `

`sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft`


### Task 3: Installing and running the application
#Install the Java Runtime Environment (JRE) and the Minecraft server

`sudo apt-get update` 

`sudo apt-get install -y default-jre-headless`

`cd /home/minecraft `

`sudo apt-get install wget`

`sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar`


#Initializing the Minecraft server

`sudo java -Xmx1024M -Xms1024M -jar server.jar nogui #Errors- Have to accept EULA Agreement (End User Licensing Agreement) first`

`sudo ls -l`

`sudo nano eula.txt #Edit the file from eula=false to eula=true`

#Create a virtual terminal screen to start the Minecraft server

`sudo apt-get install -y screen`

`sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui`


screen terminal, press Ctrl+A, Ctrl+D.(To detach the screen, created above.)
<br>
`sudo screen -r mcs`  #To retach the screen, created above.
<br/>
### Task 4: Allow client traffic

`gcloud compute --project=qwiklabs-gcp-04-a4e1aabdb53f firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server`

### Task 5: Schedule regular backups

#Create a globally unique bucket name, and store it in the environment variable YOUR_BUCKET_NAME<br/>
#Verify<br>
#create the bucket using the gsutil tool, part of the Cloud SDK
export YOUR_BUCKET_NAME=qwiklabs-gcp-04-a4e1aabdb53f-bucket

`echo $YOUR_BUCKET_NAME` 

`gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup`

#Creating a backup script

`cd /home/minecraft` #(SSH in the Terminal)

#Commands to copy into the script
```
#!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'

/usr/bin/gsutil 

cp -R ${BASH_SOURCE%/*}/world 

gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world

screen -r mcs -X stuff '/save-on\n'
```


`sudo chmod 755 /home/minecraft/backup.sh` #To make the script executable


#Test the backup script and schedule a cron job
`. /home/minecraft/backup.sh` #Run in mc-server SSH terminal
<br> `sudo crontab -e`  #open the cron table for editing
#At the bottom of the cron table, paste the following line
0 */4 * * * /home/minecraft/backup.sh

### Task 6: Server maintenance
#Stop mc-server

#Automate server maintenance with <br> startup and shutdown scripts

startup-script-url     `https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh`

shutdown-script-url 	`https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh`

<br/><br/><br/>
## Course: Essential Google Cloud Infrastructure: Foundation
### Module: Virtual Machines
    LAB: Creating Virtual Machines 
<br/>
<br/>

### Objectives of this lab
<ul>
  <li>Create several standard VMs</li>
  <li>Create advanced VMs</li>
</ul>


### Task 1: Create a utility virtual machine
```gcloud beta compute --project=qwiklabs-gcp-00-411b0dad38d4 instances create gcp-instance-1 --zone=us-central1-a --machine-type=n1-standard-1 --subnet=default --no-address --maintenance-policy=MIGRATE --service-account=519350998772-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=gcp-instance-1 --reservation-affinity=any```

### Task 2: Create a Windows virtual machine & creating firewall rules.
```gcloud beta compute --project=qwiklabs-gcp-00-411b0dad38d4 instances create instance-2 --zone=europe-west2-a --machine-type=n1-standard-2 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=519350998772-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --image=windows-server-2016-dc-core-v20200813 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd --boot-disk-device-name=instance-2 --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any```

```gcloud compute --project=qwiklabs-gcp-00-411b0dad38d4 firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server```

```gcloud compute --project=qwiklabs-gcp-00-411b0dad38d4 firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server```

### Task 3: Create a custom virtual machine
```gcloud beta compute --project=qwiklabs-gcp-00-411b0dad38d4 instances create instance-3 --zone=us-west1-b --machine-type=custom-6-32768 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=519350998772-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-9-stretch-v20200805 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-3 --reservation-affinity=any```

### Connecting via SSH to custom VM ( running the commands in VM SSH session)

`free` #To check information about unused and used memory and swap space on your custom VM

`sudo dmidecode -t 17` #Details of RAM

`lscpu` #To see details about the CPUs installed on your VM

`exit` #To exit the SHH session.