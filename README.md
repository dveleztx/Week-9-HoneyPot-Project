# Week-9-HoneyPot-Project
## Objective
*Setup a honeypot and intercept some attacks in the wild using Modern Honey Network (MHN)*

## Time Spent
*6 hours

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
