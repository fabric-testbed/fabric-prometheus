# This repo has been Deprecated
See the MeasurementFramework repo for the current monitoring repositories.  
This repo is being keep for reference purposes only.


# Prometheus Monitoring Setup
This document covers the installation and configuration of metric collection via Prometheus and Thanos.
Each head node in a FABRIC Testbed rack runs a set of docker containers (Prometheus, Thanos and supporting containers) responsible for collecting metrics from resources located at/near the rack's location. The collected metrics are perodically shipped to a global S3 compatable storage using Thanos.

Prometheus is a pull system where a Prometheus client will periodically poll resources listed in its config file. Each resource to be monitored hosts a server, the "exporter", which is responsible for gathering and returning data to the Prometheus client.

## Docker Containers

Each Fabric rack will have a docker-compose set containing Prometheus and its supporting containers. These include:
* **Prometheus**  
The [Prometheus](https://prometheus.io), [DockerHub Image](https://hub.docker.com/r/prom/prometheus), Client responsible for polling exporters for metrics.
* **Thanos**  
[Thanos](https://thanos.io), [Docker Image](https://quay.io/repository/thanos/thanos) consists of several components. For component information see [Thanos Quick Tutorial](https://thanos.io/tip/thanos/quick-tutorial.md/)
  * **Sidecar**  
  Reads the data from the Prometheus database and ships it to the global storage. Also provides interface for queries to the Prometheus database for the most recent data that may have not been shipped off yet.
  * **Query**  
  Allows for queries to be run on the data via Sidecar (local data) and Gateway (remote S3 stored data).
  * **Gateway**  
  Connects to remote object stores such as the S3 stored data.
* **SNMP exporter**  
[snmp_exporter](https://github.com/prometheus/snmp_exporter) that transforms data from SNMP devices for the Prometheus Client.
* **Docker exporter**  
[prometheus-net/docker_exporter](https://github.com/prometheus-net/docker_exporter) for information about the running containers.
* **Grafana**  
[Grafana](https://grafana.com), [DockerHub Image](https://hub.docker.com/r/grafana/grafana/), Allows for web UI to review the locally collected metrics.
* **Nginx**  
[Nginx](https://www.nginx.com), [DockerHub Image](https://hub.docker.com/_/nginx) Reverse proxy to enable SSL for Grafana and limit access to web GUIs run by various components/


## Docker Volumes
The system includes several directories that will be mapped as Docker volumes. 
These directories may already exist or will be automatically created by the Ansible playbooks. There are 2 ansible variables to set the location for the volumes:
1. Install directory `base_install_dir` contains containers and configuration files.
1. Data directory `base_data_dir` contains data files that will grow and can be large.

#### Prometheus
* `<base_install_dir>/prometheus/` Contains the configuration files including `prometheus_config.yml`.
* `<base_data_dir>/prometheus/` Contains the database and supporting data files that will be created by Prometheus.

#### Thanos Sidecar
* `<base_install_dir>/thanos/config` Contains the configuration file `object_store_config.yml` which contains the credentials for the S3 storage.
* `<base_data_dir>/prometheus/` the database and supporting files for prometheus, see above.  
* Additionally Thanos needs TLS certs so the node's TLS certs are mapped. Usually these will be:
  * `/etc/ssl/certs/<hank-name>-hn_fabric-testbed_net.pem`
  * `/etc/pki/tls/private/<hank_name>-hn_fabric-testbed_net.key`

#### Thanos Store Gateway
* `<base_install_dir>/thanos/config` Contains the configuration file `object_store_config.yml` which contains the credentials for the S3 storage. This is the same file as above.
* Additionally Thanos needs TLS certs so the node's TLS certs are mapped. Usually these will be:
  * `/etc/ssl/certs/<hank-name>-hn_fabric-testbed_net.pem`
  * `/etc/pki/tls/private/<hank_name>-hn_fabric-testbed_net.key`

#### Grafana
* `<base_install_dir>/grafana/provisioning` Contains initial settings for datasources and dashboards.
* `<base_install_dir>/grafana/custom` Contains custom ini settings.
* `<base_data_dir>/grafana` Contains grafana generated data.

#### Nginx
* `<base_install_dir>/nginx/config/nginx.conf` Contains the Nginx configuration file.
* `<base_install_dir>/nginx/config/ssl_snippet.conf` Contains the Nginx TLS/SSL snippet.
* `<base_install_dir>/nginx/config/htpasswd` Contains the user/passwords for HT access to non-login protected Thanos & Prometheus status pages.  
* Additionally, the node's TLS certs are mapped. Usually these will be:
  * `/etc/ssl/certs/<hank-name>-hn_fabric-testbed_net.pem`
  * `/etc/ssl/certs/<hank-name>-hn_fabric-testbed_net_interm.cer`
  * `/etc/pki/tls/private/<hank_name>-hn_fabric-testbed_net.key`

#### SNMP Exporter
* `<base_install_dir>/snmp/config/snmp.yml` Contains the generated snmp.yml file for specific MIBs.

#### Blackbox Exporter
 * `/opt/fabric_prometheus/blackbox/config/` Contains the configuration for blackbox probes.

#### Docker Exporter
* `/var/run/docker.sock` The exisiting docker sock.




## Web UI
Nginx provides a reverse proxy to the componets that provide web interfaces but do not provide other forms of security such as TLS or user logins. Nginx will provide TLS and htaccess if needed.
Accesible pages are:  
* Grafana`<hank-name>-hn.fabric-testbed.net:3000` Adds TLS 
* Prometheus info page `<hank-name>-hn.fabric-testbed.net:9090` Adds TLS and htaccess.
* Thanos info page `<hank-name>-hn.fabric-testbed.net:10922` Adds TLS and htaccess.

To make these pages visible and/or htaccess see the Ansible file variables `fab_componets`.
## Rack Installation
Tested using CentOS 8
### Prerequisites
* docker on head node
* docker-compose on head node
* Ansible on any machine that can access the rack's nodes
* The needed ansible role on the local ansible machine to install node-exporter. To install use 
`ansible-galaxy install cloudalchemy.node-exporter`

### Install Overview
1) Clone this repo.
1) Edit the ansible variables.
1) Run Ansible scripts.
1) Login to the rack ad start up the docker containers using `docker-compose up` in the root directory where the `docker-compose.yml` is located.
1) Goto Prometheus status page and check that the targets are being collected. TODO add links and example for this...
1) Add the rack to the global scrape file. TBD how to update this, for now contact Charles Carpenter.
1) Goto `metrics.fabric-testbed.net/grafana/explore` and use the PromQl query `up {rack="<hank-name>"}. You should get results.

### Ansible Variable Files That Need To Be Edited

In the ansible role `prometheus`:
##### `prometheus/vars/main.yml` 
Has variables that don't change often. The `base_install_dir` and `base_data_dir` can be changed if you need to install in a different location, however use the default when possible since some of the monitoring variables such as the docker container names are base on the directory names.  
Here you can also set specific values for controlling what data is collected in the `node_exporter_enabled_collectors` section. Be aware that some of the non-default values can increase the size of the collected data exponentially. See the notes in the file.

##### `prometheus/vars/sensitive.yml`
This is an example of sensitive variables such as passwords and keys that need to be set.  The file should be copied using ansible vault to encrypt your values.  Use `ansible-vault edit sensitive.yml` to create a new file and past in the data from the `sensitive_example.yml`. Edit as needed. See the file for details on needed variables. The ceph system secret_key and access_key will have to be aquired for the specific ceph system to which you will be connecting. The other variables can be created as you see fit.


## Run Playbooks
Below are the commands to run the ansible playbooks to install the main system and the node_exporters on the head node and worker nodes. You may optionally add these flags.
* Add `--check` to run without actually making changes on remote host.  
* Add `--ask-pass` if password needed to login to node.  
* Add `--ask-become-pass` if sudo password needed on node.  
* Add `--ask-vault-pass` if you have an encrypyted variable file.

### Installing Rack Systems
The rack system consists of the Prometheus and supporting containers installed on a head node. Each worker node will have a node_exporter installed.
To run the ansible scripts you need to create the `sensitive_vars.yml` file.

#### Running as a Fabric Rack Deployment
The prometheus ansible install uses hosts and vars contained in fabric-deployment/ansible github repo.  If you are not installing to the fabric system or you are testing then you will have to create your own set of hosts and var files. If you do have access, then follow these instructions.
* Clone this repo.
* Change to ansible directory in the repo.
* Place the `sensitive_vars.yml` file into the ansible directory.
* Download the deployment hosts and var files using `ansible-playbook download_hosts_and_vars_playbook.yml`
* Edit `prometheus_playbook.yml` to include the rack hosts needed in the `- hosts` line.
* Run `ansible-playbook -i tmp/working/fabric-hosts prometheus_playbook.yml --ask-vault-pass`
* You may cleanup the temporary files created by running `ansible-playbook remove_downloaded_hosts_and_vars_playbook.yml`


#### Running a Test or Other Deployment
If you do not have access to the fabric-deployment repo or you are otherwise doing your own install then the hosts and vars will need to be setup. 
* Create the needed hosts and vars files. TODO expand help here
* Clone this repo.
* Change to ansible directory in the repo.
* Place the `sensitive_vars.yml` file into the ansible directory.
* Edit `prometheus_playbook.yml` to include the rack hosts needed in the `- hosts` line.
* Run `ansible-playbook -i your-hosts-file prometheus_playbook.yml --ask-vault-pass`

### Global Metrics System
The global machine install does not need the fabric-deployment files.
* Clone this repo.
* Change to the ansible directory in the repo.
* Place the `global_sensitive_vars.yml` file into the ansible directory.
* Run `ansible-playbook -i global_metrics_hosts.yml global_metrics_playbook.yml --ask-vault-pass`
