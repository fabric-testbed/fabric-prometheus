To install to a geni slice you will need to run the following playbooks:
* playbook_install_docker_and_python_sdk.yml - needs to be run first
* playbook_geni_slice_install.yml
* playbook_install_node_exporter.yml


To run a playbook use something similar to:

`ansible-playbook -i Hosts/hosts.ini --key-file "~/.ssh/most_used_keys/id_geni_ssh_rsa" playbook_install_docker_and_python_sdk.yml --diff --check`

`ansible-playbook -i Hosts/hosts.ini --key-file "~/.ssh/most_used_keys/id_geni_ssh_rsa" playbook_geni_slice_install.yml --diff --check`

`ansible-playbook -i Hosts/hosts.ini --key-file "~/.ssh/most_used_keys/id_geni_ssh_rsa" playbook_install_node_exporter.yml --diff --check`

