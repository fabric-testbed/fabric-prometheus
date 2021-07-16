# Ansible Role
The ansible fabric_prometheus role can install the reqired components for Prometheus monitoring of various systems.
* Fabric Central Infrastructure (aka global metics)
* Fabric Rack
* Fabric Slice
* GENI Slice

These systems all have similar & overlapping features but differ in which components are added and how they are configured.
The `install_type` determines which system is being installed. The options are:
* fabric_rack
* geni_slice
* fabric_slice
* global

Each type of install requires one or more variable files.  
You can use the playbooks included or alter them for special cases.
* playbook_role_fabric_rack_install.yml
* playbook_role_geni_slice_install.yml
* playbook_role_fabric_global_install.yml





# Install on a GENI slice

To install to a geni slice you will need to run the following playbook:
* playbook_role_geni_slice_install

For the vars file see the the vars/example directory for examples. 
* geni_example_vars.yml
* geni_private_example_vars.yml
* node_exporter_example_vars.yml
Copy these files and set the values in <>. Then pass them to the playbook using `--extra-vars "@your_var_file.yml"`. You may pass them using 3 --extra-vars args, or you may combine them all into one file.

To run a playbook use something similar to:

### Install Complete system.
`ansible-playbook -i hosts.ini --key-file "~/.ssh/id_geni_ssh_rsa" playbook_role_geni_slice_install.yml --extra-vars "@your_vars.yml" --diff`

### Install just the monitoring software on the monitor node.
`ansible-playbook -i hosts.ini --key-file "~/.ssh/id_geni_ssh_rsa" playbook_role_geni_slice_install.yml --extra-vars "@your_vars.yml" --diff --tags monitor`

### Install just the node exporters.
`ansible-playbook -i hosts.ini --key-file "~/.ssh/id_geni_ssh_rsa" playbook_role_geni_slice_install.yml --extra-vars "@your_vars.yml" --diff --tags exporters`


# GENI test after ansible scripts are run.
## Prometheus
Go to the monitor node's Prometheus web UI at https://<monitor_node_ip>:9090  
You will have to type in the username and password that you set in the variable file.
Click on the "Status" drop down and choose "Targets".  Assuming the system has been up for a few minutes all of the target's state should be a green UP.  
Click on the "Graph" link and type "up" in the search box. Click "Execute". You should see a ping job for each node, a node jobe for each node and then the docker adn prometheus jobs.

## Grafana
Go to the Grafana web UI at https://<monitor_node_ip>/grafana  
Type in "admin" for the user and the password you set in the variables file.  
Click on the compass icon to the left to open the Explore page. Type "up" for the metrics query the click "Run Query". You should see the same list you saw in the Prometheus test.  
Hover over the four squares icon and click Manage.  Click Import. Type in "1860" and click "Load". Keep defaults but Choose "Prometheus" as the datascource in the bottom drop down. Click "Import".  
Now in your Dashboards list you will See "Node Exporter Full". Open the dashboard and see if the drop down for the "Hosts" has all of the nodes for the slice.

## Check status on Monitor node
SSH into the monitor node. Type `docker ps` to see if all the containers are running. Depending on your install variable you should have: 
* prometheus
* grafana
* nginx
* blackbox
* docker_exporter

This should match what results when you run the query "docker_container_running_state" on the Explore page in Grafana.  



# Misc Notes
Ansible control node must have  passlib[bcrypt] installed.  
Galaxy role installs 
* geerlinguy.docker sp?
* cloudalchemy.node_exporter sp?
