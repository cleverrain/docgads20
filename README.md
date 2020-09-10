# docgads20
Documentation GADS 2020

CONSOLE AND CLOUD SHELL : https://googlepluralsight.qwiklabs.com/focuses/8832016?parent=lti_session
Create a bucket from the command line in Cloud Console
gsutil mb gs://<BUCKET_NAME>
Copy a file from Cloud console to a bucket from GCP Cloud Console
gsutil cp e1b877f241.png gs://qwiklabs-gcp-02-aerzr
From cloud console create a persistent state
nano .profile
source infraclass/config


Create an instance without external IP address under Debian 10
gcloud beta compute --project=qwiklabs-gcp-03-7439e018d3f9 instances create instance-1 --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --no-address --maintenance-policy=MIGRATE --service-account=135683109594-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

Create an instance with external IP address under Server 2016 Core with http and https allowed in firewall-rules
gcloud beta compute --project=qwiklabs-gcp-03-7439e018d3f9 instances create instance-2 --zone=europe-west2-a --machine-type=n1-standard-2 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=135683109594-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --image=windows-server-2016-dc-core-v20200813 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd --boot-disk-device-name=instance-2 --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

gcloud compute --project=qwiklabs-gcp-03-7439e018d3f9 firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

gcloud compute --project=qwiklabs-gcp-03-7439e018d3f9 firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server


Create a custom instance with external IP address under Debian 10 (6 CPU and 32 GB)
gcloud beta compute --project=qwiklabs-gcp-03-7439e018d3f9 instances create instance-4 --zone=us-west1-b --machine-type=custom-6-32768 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=135683109594-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-4 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any


CONFIGURING GCLB, CDN, TRAFFIC BLACKLISTING WITH CLOUD ARMOR : https://googlepluralsight.qwiklabs.com/focuses/10298334?parent=lti_session
# create a firewall rule to allow port 80 traffic
gcloud compute firewall-rules create \
   www-firewall-network-lb --target-tags network-lb-tag \
   --allow tcp:80

# create an instance template named web-template
gcloud compute instance-templates create web-template \
    --machine-type=n1-standard-4 \
    --image-family=debian-9 \
    --image-project=debian-cloud \
    --machine-type=n1-standard-1 \
    --tags=network-lb-tag \
    --metadata=startup-script=\#\!\ /bin/bash$'\n'apt-get\ update$'\n'apt-get\ install\ apache2\ -y$'\n'service\ apache2\ restart$'\n'ZONE=\$\(curl\ \"http://metadata.google.internal/computeMetadata/v1/instance/zone\"\ -H\ \"Metadata-Flavor:\ Google\"\)$'\n'echo\ \'\<\!doctype\ html\>\<html\>\<body\>\<h1\>Web\ server\</h1\>\<h2\>This\ server\ is\ in\ zone:\ ZONE_HERE\</h2\>\</body\>\</html\>\'\ \|\ tee\ /var/www/html/index.html$'\n'sed\ -i\ \"s\|ZONE_HERE\|\$ZONE\|\"\ /var/www/html/index.html
	
# create a basic http health check	
gcloud compute http-health-checks create basic-http-check

#  create a managed instance group with 3 instances
gcloud compute instance-groups managed create web-group \
   --template web-template --size 3 --zones \
   us-central1-a,us-central1-b,us-central1-c,us-central1-f

# create the load balancing service
gcloud compute instance-groups managed set-named-ports \
   web-group --named-ports http:80 --region us-central1
gcloud compute backend-services create web-backend \
   --global \
   --port-name=http \
   --protocol HTTP \
   --http-health-checks basic-http-check \
   --enable-logging
gcloud compute backend-services add-backend web-backend \
   --instance-group web-group \
   --global \
   --instance-group-region us-central1
gcloud compute url-maps create web-lb \
   --default-service web-backend
gcloud compute target-http-proxies create web-lb-proxy \
   --url-map web-lb
gcloud compute forwarding-rules create web-rule \
   --global \
   --target-http-proxy web-lb-proxy \
   --ports 80

# Check if the instances are registered in the load balancer
gcloud compute backend-services get-health web-backend --global

# Retrieve the load balancer IP
gcloud compute forwarding-rules describe web-rule --global

#Create traffic by sending requests to the server
while true; do curl -m1 34.107.186.225; done   

# Create an instance E2 type in Australia
gcloud beta compute --project=qwiklabs-gcp-03-dccb9c89b9c4 instances create access-test --zone=australia-southeast1-b --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=457738097566-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=access-test --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

# Send a curl request to the load balancer
curl -m1 34.107.186.225


