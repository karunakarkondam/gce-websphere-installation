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


**Prepare Ubuntu for WebSpere Application Server**

Install required packages

$ sudo apt-get install libxtst6 libgtk2.0-bin libxft2

Download IBM Installation Manager

https://www.ibm.com/support/fixcentral/swg/selectFixes?parent=ibm%7ERational&product=ibm/Rational/IBM+Installation+Manager&release=All&platform=Linux&function=all

•	1.9.1.5-IBMIM-LINUX-X86_64-20210309_1755
Install IBM Installation Manager
$ sudo ./installc -acceptLicense -showProgress
Or
Install IBM Installation Manager as Non-Root user userinstc
$ sudo ./userinstc -acceptLicense -showProgress
Verify IIM Installation
$ sudo /opt/IBM/InstallationManager/eclipse/tools/imcl listInstalledPackages
Change Directry to IBM Installation Manager
$ cd /opt/IBM/InstallationManager/eclipse/tools
Create Credential File
$ sudo ./imutilsc saveCredential \
    -secureStorageFile cred.file \
    -userName USER_ID \
    -userPassword USER_PASSWORD \
    -url http://www.ibm.com/software/repositorymanager/V9WASBase
List Offering Package List
$ sudo ./imcl listAvailablePackages \
    -repositories http://www.ibm.com/software/repositorymanager/V9WASBase \
    -secureStorageFile cred.file
Install WebSphere Application Server
$ sudo ./imcl install com.ibm.websphere.BASE.v90_9.0.5006.20201109_1605 com.ibm.java.jdk.v8 \
    -repositories http://www.ibm.com/software/repositorymanager/V9WASBase \
    -installationDirectory /opt/IBM/WebSphere/AppServer \
    -sharedResourcesDirectory /opt/IBM/IMShared \
    -showProgress \
    -secureStorageFile cred.file \
    -acceptLicense
Verify WAS Installation
$ /opt/IBM/WebSphere/AppServer/bin/versionInfo.sh
Create default profile
$ sudo ./manageprofiles.sh -create -templatePath /opt/IBM/WebSphere/AppServer/profileTemplates/default
List Profiles
$ sudo ./manageprofiles.sh -listProfiles
Start App Server
$ sudo ./startServer.sh server1 -profileName AppSrv01
Confirm Server Port
$ grep WC /opt/IBM/WebSphere/AppServer/profiles/AppSrv01/properties/portdef.props
Access to Admin Console (UserID: anything)

 ![image](https://github.com/user-attachments/assets/ada365a1-35e7-47e2-8194-95eca38606d3)


•	WC_adminhost=9060: http://<HOSTNAME> or <IP-ADDRESS>:9060/ibm/console/
•	WC_adminhost_secure=9043: https://<HOSTNAME> or <IP-ADDRESS>:9060/ibm/console/

Deploy Sample App
$ wget http://public.dhe.ibm.com/software/websphere/appserv/library/v85/samples/JaxWSServicesSamples-8_5.zip
$ unzip JaxWSServicesSamples-8_5.zip
$ sudo mkdir /opt/IBM/WebSphere/AppServer/profiles/AppSrv01/apps
$ sudo mv JaxWSServicesSamples-8.5/installableApps/JaxWSServicesSamples.ear /opt/IBM/WebSphere/AppServer/profiles/AppSrv01/apps
Install on the Admin Console with default settigs
•	http://<IP-ADDRESS>:9080/wssamplesei/demo
Stop App Server
$ sudo ./stopServer.sh server1 -profileName AppSrv01

