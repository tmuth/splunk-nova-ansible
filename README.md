# NOVA Custom Configuration
## Overview
This project uses ansible to customize a nova cluster (https://splunkit.io/splunk) after its been provisioned. It was created by Tyler Muth.

In short, you populate the nova-config/hosts file like this:
```
[cluster_master]
master1 ansible_host=10.202.13.162

[search_heads]
sh1 ansible_host=10.202.11.88

[indexers]
idx1 ansible_host=10.202.11.95
idx2 ansible_host=10.202.12.234
idx3 ansible_host=10.202.11.31

[license_master]
lm1 ansible_host=10.202.11.227
```

Run the playbook and it:
- Creates indexes you defined in cluster_indexes.yml
- Loads apps on search heads from a particular folder
- Loads apps on indexers from a particular folder
- oneshot loads files from a data dir that you also defined with attributes in load_data.yml
- Updates the /etc/hosts file on your local laptop to add entries for those in the nova-config/hosts files
- Sets up passwordless ssh with a public key from your machine
- Sets up some ease of use aliases, etc in bash_profile

## Conventions
- Any references to *nova-config/* in this guide refer to the root of cloned repo. The nova-config/ dir contains files like hosts, site.yml, and ansible.cfg

## Pre-Setup
NOVA uses passwords for ssh. To get this to work on Mac OS:
- Install sshpass: https://gist.github.com/arunoda/7790979
- Note that this also depends on ansible.cfg in the project directory to work


## Setup
- Install ansible on your local machine. Everything runs locally. 
- Clone this repo to your local machine and cd to the directory of the repo (it contains site.yml)
- Create a cluster on nova (https://splunkit.io/splunk) and wait about 15-30 minutes for it to complete provisioning.
- Edit the nova-config/hosts file in this repo with the IP addresses of each server listed in the web UI of nova. You can run the following command ```ansible all -m ping -i hosts``` to validate that ansible can connect to all of the hosts. 
- Run:
```ansible-playbook -i hosts site.yml```


## Utilities
### Update local hosts
This will add entry to the /etc/hosts file on your laptop based on the hostnames and IPs you configured in the hosts file (from nova). This allows you to reference each instance by name (sh1,idx1,idx2,master1,etc) instead of by IP address. 
- cd to util
- Run:
```ansible-playbook -i ../hosts update-local-hosts.yml```


### Delete all indexes
This play puts an empty indexes.conf file on the cluster master, pushes the bundle, then deletes the directories for the indexes enumerated in ../group_vars/cluster_indexes.yml. It allows your to delete the data so you can reload it. 
- cd to util
- Run:
```ansible-playbook -i ../hosts delete_all_indexes.yml```
- To reload everything, you could just run the following again from the top level directory:
```ansible-playbook -i hosts site.yml```

