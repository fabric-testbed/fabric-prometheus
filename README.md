
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

### Steps
1) Clone this repo.
1) Edit the ansible variables.
1) Run Ansible scripts.
1) Run `docker-compose up` in the root directory where the `docker-compose.yml` is located.
1) Goto Prometheus status page....
1) Add node_exporters to head and worker nodes using the Ansible script `????`
1) Goto `metrics.fabric-testbed.net/grafana/explore` and use the PromQl query `up {rack="<hank-name>"}. You should get results.

### Conguration Files That Need To Be Edited

* `prometheus/config/prometheus_config.yml`  
Set the rack name to the site acronym found at [Fabric Sites](https://fabric-testbed.atlassian.net/wiki/spaces/FP/pages/168624158/FABRIC+Site+Documentation)
* `thanos/config/object_store_config.yml`  
Set your CEPH access key and secret.
* `nginx/config/nginx.conf`  
Set you VM url. Set cert filenames to match below certs.
* `nginx/certs`  
Add the ssl certs for nginx.
* `grafana/custom/custom.ini`  
Set your server url.  
If you are enabling CiLogon, set the client_id and client_secret. TBD TODO add links for details when system ready.
* `grafana/env_file`
Add an admin password so you can login to Grafana. `GF_SECURITY_ADMIN_PASSWORD=yourpassword`

## Node Exporter
There is now an Ansible script in the ansible/node_exporter directory that will perform the node_exporter install. See the ansible/node_exporter/README.md for details.  


## Global Instalation
TODO


