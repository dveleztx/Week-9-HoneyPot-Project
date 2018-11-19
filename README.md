# Week-9-HoneyPot-Project
## Objective
*Setup a honeypot and intercept some attacks in the wild using Modern Honey Network (MHN)*

## Time Spent
*8 hours*

## Report
### Create and Install MHN Admin Application

Details:
- Create the MHN Admin VM on Google Cloud Platform
- SSH into the MNH Admin VM, update it, install git, and clone MHN Repo
- Edit **scripts/install_hpfeeds.sh** file within repo
  - Instead of using HurricaneLabs, use [couozu](https://github.com/couozu/pyev.git#egg=pyev) repo
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
- On the **admin console**, go to the *Deploy* tab and select "Ubuntu - Dionaea with HTTP"
  - Use the script to on the Honeypot VM to deploy it and it'll add to the list of sensors on the admin console (shown above on the gif) 

#### Analyze Attacks and Payloads on Dionaea Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/dionaea_attack_report_with_payloads.gif" width="800">

Issues:
- Payloads were not registered on the MHN Admin Console
  - Decided to look on the backend and look at sql logs and came up with some good data and captures
  - SQLITE file was easy readable using SQLITE3

#### Analyze Attack and Payloads on SNORT Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/snort_attack_report_with_payloads.gif" width="800">

#### Analyze Attack and Payloads on Suricata Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/suricata_attack_report_with_payload.gif" width="800">

#### Analyze Attack and Payloads on p0f Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/p0f_attack_report_with_payloads.gif" width="800">

#### Analyze Attack and Paylods on kippo Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/kippo_attack_report_with_payloads.gif" width="800">


### Demonstrations of Attack in Progress (on Dionaea) with Interaction HoneyMap

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

### Charts of Kippo/Cowrie Top Users/Passwords and Attackers

#### Top Attackers
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/kippo-cowrie-charts/Kippo_Cowrie%20Top%20Attackers.png" width="800">

#### Top User/Passwords
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/kippo-cowrie-charts/Kippo_Cowrie%20Top%20User_Passwords.png" width="800">
