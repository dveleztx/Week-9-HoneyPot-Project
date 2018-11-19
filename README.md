# Week-9-HoneyPot-Project
## Objective
*Setup a honeypot and intercept some attacks in the wild using Modern Honey Network (MHN)*

## Time Spent
*15 hours*

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
- Full documentation [here](https://cloud.google.com/sdk/install)

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

Steps:
- Conduct an update on the OS
  - ```sudo apt-get update```
- Download git
  - ```sudo apt-get install git -y```
- cd into /opt/ and clone MHN repo
  - ```cd /opt/; sudo git clone https://github.com/RedolentSun/mhn.git```
- Edit scripts/install_hpfeeds.sh within repo
  - From ```pip install -e git+https://github.com/HurricaneLabs/pyev.git#egg=pyev``` to: ```pip install -e git+https://github.com/couozu/pyev.git#egg=pyev```
- Install using script
  - ```sudo ./install.sh```
- Make sure to verify supervisorctl status, that all services are running, per the following:
  - <img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/supervisorctl_status.png" width="600">

Issues:
- Admin console did not come up, suspected firewall issues
  - To fix, create a firewall rule to open http as demonstrated next
  - Once rule is complete, make sure MHN VM has port 80 open (UFW/iptables)
  - ```sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT```
  
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
- Next, create the VM for the honeypot, we'll call it *mhn-honeypot-1*
  - ```gcloud compute instances create "mhn-honeypot-1" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-honeypot","http-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-honeypot-1"```
- On the **admin console**, go to the *Deploy* tab and select "Ubuntu - Dionaea with HTTP"
  - Use the script to on the Honeypot VM to deploy it and it'll add to the list of sensors on the admin console (shown above on the gif) 

## Attack Reporting
### Analyze Attacks and Payloads on Dionaea Honeypot

Demonstration:
<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/dionaea_attack_report_with_payloads.gif" width="800">

Secondary Attack using [Sn1per](https://github.com/1N3/Sn1per)
- The report from the recon scan is [here](https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/dionaea-sniper-report.txt)
- The gif of the attack is [here](https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/sniper_attack_dionaea_honeypot.gif)
- A brief summary of the report above:
```
Starting Nmap 7.70 ( https://nmap.org ) at 2018-11-19 03:55 CST
Nmap scan report for 200.222.226.35.bc.googleusercontent.com (35.226.222.200)
Host is up (1.2s latency).
Not shown: 258 closed ports, 204 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
42/tcp    open  nameserver
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
443/tcp   open  https
1433/tcp  open  ms-sql-s
1723/tcp  open  pptp
3306/tcp  open  mysql
5060/tcp  open  sip
11211/tcp open  memcache
27017/tcp open  mongod
[+] URL's Discovered: 
/usr/share/blackwidow/35.226.222.200_80/35.226.222.200_80-urls-sorted.txt
__________________________________________________________________________________________________

[+] Dynamic URL's Discovered: 
/usr/share/blackwidow/35.226.222.200_80/35.226.222.200_80-dynamic-sorted.txt
__________________________________________________________________________________________________

[+] Form URL's Discovered: 
/usr/share/blackwidow/35.226.222.200_80/35.226.222.200_80-forms-sorted.txt
__________________________________________________________________________________________________

[+] Unique Dynamic Parameters Discovered: 
/usr/share/blackwidow/35.226.222.200_80/35.226.222.200_80-dynamic-unique.txt
__________________________________________________________________________________________________

[+] Sub-domains Discovered: 
/usr/share/blackwidow/35.226.222.200_80/35.226.222.200_80-subdomains-sorted.txt
__________________________________________________________________________________________________

[+] Emails Discovered: 
/usr/share/blackwidow/35.226.222.200_80/35.226.222.200_80-emails-sorted.txt
__________________________________________________________________________________________________

[+] Phones Discovered: 
/usr/share/blackwidow/35.226.222.200_80/35.226.222.200_80-phones-sorted.txt
__________________________________________________________________________________________________

[+] Loot Saved To: 
/usr/share/blackwidow/35.226.222.200_80/
```

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

<img src="https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/Attack%20Stats.png" width="800">

**NOTE:** 
- Each specific log is listed above in the chart under "Attacks" per honeypot. 
- Also, the glastopf VM didn't seem to be completely working, although it did have a page standing, which does log if any attempts are made against it, [this](https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/imgs/glastopf-%20admin.php.png) was the front page.
- It should be known that payloads had to be dug up for the Dionaea machine, as I had noted above. The logs for dionaea will show the payloads logged into a sqlite
- It should also be known that conpot has 0 on my attack chart on my MHN Admin Console, but looking at the auth.log in /var/log on the VM determined that there were numerous hits of attackers trying to get in *and succeeding*. The charts above also support this claim.
- I lost access to kippo within hours, I was not able to pull any logs from the VM itself, but the admin console did capture traffic going through ssh (port 22), the charts also support this claim.

### Conclusion
In my findings, I pulled out logs from each VM to get a better idea what specific transactions were taken place per honeypot. A more broader summary of data collected from the honeypots are listed here: [session.json](https://github.com/dveleztx/Week-9-HoneyPot-Project/blob/master/mhn-admin/session.json).

Standing up these honeypots indicate one thing, that the cyber world of exploitation and attacking is very real and alive. My data was collected for the course of 3 days, and the amount of attacks were staggering. With over **20,000** attacks, with close to **7,000** attacks committed per day, it's hard not to phathom a world without strong security practices. These honeypots prove that the existence of vulnerabilities whether having real threat surfaces or not, in our case in honeypot snares, that nobody is absolutely safe. This is why cyber security must be taken seriously in every field, in every industry, and coming up with solutions based off data collected in instances like this help us understand our enemies better.
