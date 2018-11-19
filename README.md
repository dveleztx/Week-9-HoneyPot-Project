# Week-9-HoneyPot-Project
## Objective
*Setup a honeypot and intercept some attacks in the wild using Modern Honey Network (MHN)*

## Time Spent
*12 hours*

## Report

### Introduction - Resources and Tools

The OS I used to build all this was through [Ubuntu 16.04 Xenial](http://releases.ubuntu.com/16.04/).

We will be building an Admin VM and Honeypot VMs using the Modern Honey Network and Google Cloud Platform (GCP).

The first thing to do is to create a free account on [GCP's Free Tier](https://cloud.google.com/free/).

The VMs will be running Ubuntu 14.04 Trusty.

### Installing GCP SDK

- Create an environment variable for the correct distro
  - ```export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"```
- Add the Cloud SDK URL distro to sources.list
  - ```echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list```
- Import Google Cloud public key
  - ```curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -```
- Update and install the SDK
  - ```sudo apt-get update && sudo apt-get install google-cloud-sdk```
- Start GCloud
  - ```gcloud init```
- Set Respective Zone and Region
  - ```gcloud config set compute/region us-central1```
  - ```gcloud config set compute/zone us-central1-c```
- Verify GCloud config with the following command:
  - ```gcloud config list```

### Create and Install MHN Admin Application

Details:
- Create the MHN Admin VM on Google Cloud Platform
- SSH into the MNH Admin VM, update it, install git, and clone the [Modern Honey Network (MHN)](https://github.com/RedolentSun/mhn.git) from RedolentSun.
- Edit **scripts/install_hpfeeds.sh** file within repo
  - Instead of using HurricaneLabs, use [couozu](https://github.com/couozu/pyev.git#egg=pyev) repo
  - From ```pip install -e git+https://github.com/HurricaneLabs/pyev.git#egg=pyev``` to: ```pip install -e git+https://github.com/couozu/pyev.git#egg=pyev```
- Run install scripts

Demonstration:
##### Creation of MHN VM
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/create_mhn_admin.gif" width="800">

##### Installation of MHN Application
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/install_mhn_app_revised.gif" width="800">

Issues:
- Admin console did not come up, suspected firewall issues
  - To fix, create a firewall rule to open http as demonstrated next
  - Once rule is complete, make sure MHN VM has port 80 open (UFW/iptables)
  
##### Create Firewall Rule on Google Cloud Platform
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/firewall_rules_revised.gif" width="800">

#### Honeypots to use

The honeypot VMs are easily deployable, and will be shown below on how exactly to deploy/use them. We will be using the following:
 - Ubuntu - Dionaea with HTTP - [Documentation](https://dionaea.readthedocs.io/en/latest/)
 - Ubuntu - Snort - [Documentation](https://www.snort.org/documents)
 - Ubuntu - p0f - [Documentation](http://lcamtuf.coredump.cx/p0f3/)
 - Ubuntu - Glastopf - [Installation](https://github.com/mushorg/glastopf/blob/master/docs/source/installation/installation_ubuntu.rst) and [Documentation](https://glastopf.readthedocs.io/en/latest/)
 - Ubuntu - Suricata - [Documentation](https://suricata-ids.org/)
 - Ubuntu - Conpot - [Documentation](http://conpot.org/) and [Conpot GitHub](https://github.com/mushorg/conpot)
 - Ubuntu - Kippo as vulnerable Juniper Netscreen - [Kippo GitHub](https://github.com/desaster/kippo)

### Create and Deploy Dionaea VM

Details:
- Create the Dionaea Honeypot on Google Cloud Platform
- SSH into the Dionaea Honeypot and deploy script
- Verify on MHN Admin Console
- **NOTE**: These details will be the same for all the other honeypots following this

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/create_install_dionaea.gif" width="800">

Steps:
- First, create firewall rule to allow incomign traffic on all ports
  - ```gcloud beta compute firewall-rules create mhn-allow-honeypot --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=all --source-ranges=0.0.0.0/0 --target-tags=mhn-honeypot```
- Next, create teh VM for the honeypot, we'll call it *mhn-honeypot-1*
  - ```gcloud compute instances create "mhn-honeypot-1" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-honeypot","http-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-honeypot-1"```
- Make sure to verify supervisorctl status, that all services are running, per the following:
  - <img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/supervisorctl_status.png" width="600">
- On the **admin console**, go to the *Deploy* tab and select "Ubuntu - Dionaea with HTTP"
  - Use the script to on the Honeypot VM to deploy it and it'll add to the list of sensors on the admin console (shown above on the gif) 

## Attack Reporting
### Analyze Attacks and Payloads on Dionaea Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/dionaea_attack_report_with_payloads.gif" width="800">

Issues:
- Payloads were not registered on the MHN Admin Console
  - Decided to look on the backend and look at sql logs and came up with some good data and captures
  - SQLITE file was easy readable using SQLITE3, more logs attached in the Summary below

### Analyze Attack and Payloads on SNORT Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/snort_attack_report_with_payloads.gif" width="800">

### Analyze Attack and Payloads on Suricata Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/suricata_attack_report_with_payload.gif" width="800">

### Analyze Attack and Payloads on p0f Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/p0f_attack_report_with_payloads.gif" width="800">

### Analyze Attack and Paylods on kippo Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/kippo_attack_report_with_payloads.gif" width="800">

### Charts of Kippo/Cowrie Top Users/Passwords and Attackers

#### Top Attackers
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/kippo-cowrie-charts/Kippo_Cowrie%20Top%20Attackers.png" width="800">

#### Top User/Passwords
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/kippo-cowrie-charts/Kippo_Cowrie%20Top%20User_Passwords.png" width="800">


### Demonstrations of Attacks in Progress (on Dionaea) with Interaction HoneyMap

#### Demo 1:

<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/attack%20in%20progress.gif" width="800">

#### Demo 2 (using NMAP):

<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/my_attack.gif" width="800">


## Summary of Attacks

| Number | Hostname | IP | Honeypot | UUID | Attacks |
| --- | --- | --- | --- | --- | --- |
| 1 | mhn-honeypot-1 | 35.226.222.200 | dionaea | 4a3e4a5e-ea40-11e8-9599-42010a800002 | [10517](https://github.com/dveleztx/Week-9-HoneyPot-Project/tree/master/dionaea_logs) |
| 2 | mhn-honeypot-2 | 35.202.198.20 | snort | 	cd43e14c-ea4b-11e8-9599-42010a800002 | [2134](https://github.com/dveleztx/Week-9-HoneyPot-Project/tree/master/snort_logs) |
| 3 | mhn-honeypot-3 | 35.192.2.246 | p0f | 9784f9b8-eb92-11e8-9599-42010a800002 | [961](https://github.com/dveleztx/Week-9-HoneyPot-Project/tree/master/p0f_logs) |
| 4 | mhn-honeypot-4 | 35.225.72.209 | glastopf | 5286be14-ea52-11e8-9599-42010a800002 | [0 (reported)](https://github.com/dveleztx/Week-9-HoneyPot-Project/tree/master/glastopf_logs) |
| 5 | mhn-honeypot-5 | 35.184.249.204 | suricata | 4c91dbd6-ea54-11e8-9599-42010a800002 | [7193](https://github.com/dveleztx/Week-9-HoneyPot-Project/tree/master/suricata_logs) | 
| 6 | mhn-honeypot-6 | 35.238.38.58 | conpot | 903087dc-ea97-11e8-9599-42010a800002 | [300+](https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/conpot_logs/auth.log) |
| 7 | mhn-honeypot-7 | 35.238.9.117 | kippo | e0ddd6ee-ea56-11e8-9599-42010a800002 | 301 |

**NOTE:** Each specific log is listed above in the chart under "Attacks" per honeypot.

In my findings, I pulled out logs from each VM to get a better idea what specific transactions were taken place per honeypot. A more broader summary of data collected from the honeypots are listed here: [session.json](https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/mhn-admin/session.json).
