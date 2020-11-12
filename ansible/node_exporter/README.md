# Installing Node Exporter

## Install Ansible
[Installing Ansible Details](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
TLDR `sudo dnf install ansible`

## Install Needed Ansible role
`ansible-galaxy install cloudalchemy.node-exporter`

## Set user username/pasword for client auth.

Edit file `users.yml` to contain the username:password for the rack's Prometheus client. 
Use bycrypt (found in apache2-utils or httpd-tools package) to generate the pasword hash. 
Optionally, you can also add any other users that may need access to the exporter's metrics. 

## Set Host
Edit the `hosts` file. Add the host(s) onto which  you will be installing the node exporter. See the file for examples.

## Run Playbook
Run the playbook using `ansible-playbook -i hosts node_exporter_self_signed_cert_playbook.yml`
