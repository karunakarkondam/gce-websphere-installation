gcloud beta compute instances create my-instance \
  --zone us-central1-a \
  --machine-type e2-medium \
  --scopes cloud-platform \
  --tags http-server \
  --image=ubuntu-2004-focal-v20210325 \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=10GB

gcloud compute firewall-rules create default-allow-http \
  --direction=INGRESS \
  --priority=1000 \
  --network=default \
  --action=ALLOW \
  --rules=tcp:80 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server

gcloud compute firewall-rules create default-allow-ssh \
  --direction=INGRESS \
  --priority=1000 \
  --network=default \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server

gcloud compute firewall-rules create default-allow-was-admin \
  --direction=INGRESS \
  --priority=1000 \
  --network=default \
  --action=ALLOW \
  --rules=tcp:9060 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server

gcloud compute firewall-rules create default-allow-was-host \
  --direction=INGRESS \
  --priority=1000 \
  --network=default \
  --action=ALLOW \
  --rules=tcp:9080 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server

$ gcloud beta compute ssh my-instance --zone us-central1-a --project (gcloud config get-value project)
