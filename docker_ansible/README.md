To install to a geni slice you will need to run the following playbooks:
* playbook_install_docker_and_python_sdk.yml - needs to be run first
* playbook_geni_slice_install.yml
* playbook_install_node_exporter.yml


To run a playbook use something similar to:

`ansible-playbook -i Hosts/hosts.ini --key-file "~/.ssh/most_used_keys/id_geni_ssh_rsa" playbook_install_docker_and_python_sdk.yml --diff --check`

`ansible-playbook -i Hosts/hosts.ini --key-file "~/.ssh/most_used_keys/id_geni_ssh_rsa" playbook_geni_slice_install.yml --diff --check`

`ansible-playbook -i Hosts/hosts.ini --key-file "~/.ssh/most_used_keys/id_geni_ssh_rsa" playbook_install_node_exporter.yml --diff --check`


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

