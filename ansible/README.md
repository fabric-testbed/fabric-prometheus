# Installing Fabric Prometheus System on a Rack

The install consists of 2 Ansible playbooks. The first playbook installs the main components needed on the head node. The second playbook installs the rack's node exporters.

## Needed Information
You will need several bits of information to setup the monitoring on a rack.
* Rack short name. This can be found at 

## Install Ansible
[Installing Ansible Details](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
TL;DR `sudo dnf install ansible`

## Install Needed Ansible role for Node Exporter
`ansible-galaxy install cloudalchemy.node-exporter`

## SSH Config
Since access is limited to the racks, it maybe easiest to setup an ssh conf file to access your rack.
There are many variations on this. Here is an example that uses a rack called "samp".
```Host samp-hn
	ProxyCommand ssh -W %h:%p your-jbox
	HostName samp-hn.fabric-testbed.net
	IdentityFile ~/.ssh/my_key

Host samp-w1
	ProxyCommand ssh -W %h:%p samp-hn
	HostName samp-w1.fabric-testbed.net
	IdentityFile ~/.ssh/my_key

Host samp-w2
	ProxyCommand ssh -W %h:%p samp-hn
	HostName samp-w2.fabric-testbed.net
	IdentityFile ~/.ssh/my_key

Host samp-w2
	ProxyCommand ssh -W %h:%p samp-hn
	HostName samp-w2.fabric-testbed.net
	IdentityFile ~/.ssh/my_key
```

## Create the Host File with Variables
All of the values needed to setup the rack are contained in the `hosts.yml` file. To create this file copy the `sample_hosts.yml` file and set all values contained in `<>` for your rack. The `sample_hosts.yml` file has more details to help you.

## Run Playbooks
Below are the commands to run the ansible playbooks to install the main system and the node_exporters on the head node and worker nodes. You may optionally add these flags.
* Add `--check` to run without actually making changes on remote host.  
* Add `--ask-pass` if password needed to login to node.  
* Add `--ask-become-pass` if sudo password needed on node.  

### Main System
Run the playbook using `ansible-playbook -i hosts.yml main_playbook.yml`.  

### Node Exporters
Run the playbook using `ansible-playbook -i hosts.yml node_exporter_playbook.yml`.  
