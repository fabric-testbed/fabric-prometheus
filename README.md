
# Prometheus Monitoring Setup
This document covers the installation and configuration of the global metric collection via Prometheus and Thanos.
Each rack in the FABRIC Testbed has a VM running a set of docker containers (Prometheus, Thanos and supporting containers) responsible for collecting metrics from resources located at/near the rack's location. The collected metrics are perodically shipped to a global S3 compatable storage using Thanos.

Prometheus is intended to be a pull system where a Prometheus client will periodically poll resources listed in its config file. Each resource to be monitored hosts a server, the "exporter", which is responsible for gathering and returning data to the Prometheus client.

## Docker Containers

Each Fabric rack will have a VM running a docker-compose set containing Prometheus and its supporting containers. These include:
* **Prometheus**  
The [Prometheus](https://prometheus.io), [DockerHub](https://hub.docker.com/r/prom/prometheus), Client responsible for polling exporters for metrics.
* **Thanos**  
[Thanos](https://thanos.io), [Docker](https://quay.io/repository/thanos/thanos) consists of several components. For component information see [Thanos Quick Tutorial](https://thanos.io/tip/thanos/quick-tutorial.md/)
  * **Sidecar**  
  Reads the data from the Prometheus database and ships it to the global storage.
  * **Query**  
  Allows for queries to be run on the data via Sidecar (local data) and Gateway (remote S3 stored data)
  * **Gateway**  
  Connects to remote object stores such as the S3 stored data.
* **SNMP exporter**  
[snmp_exporter](https://github.com/prometheus/snmp_exporter) that transforms data from SNMP devices for the Prometheus Client.
* **Docker exporter**  
[prometheus-net/docker_exporter](https://github.com/prometheus-net/docker_exporter) for information about the running containers.
* **Grafana**  
[Grafana](https://grafana.com), [DockerHub](https://hub.docker.com/r/grafana/grafana/), Allows for web UI to review the locally collected metrics.
* **Nginx**  
[Nginx](https://www.nginx.com), [DockerHub](https://hub.docker.com/_/nginx) Reverse proxy to enable SSL for Grafana and limit access to web GUIs run by various components/


## Docker Volumes
The system includes several directories that will be mapped as Docker volumes. The repo directory `example_volumes` contains these directories and base configuration files. See below on which files need editing. 
The Prometheus container binds 2 volumes: 
* `prometheus/config` Contains the configuration file `prometheus_config.yml`. You will need to edit this file.
* `prometheus/data` Contains the database and supporting data files that will be created by Prometheus.

The Thanos sidecar container binds 2 volumes:
* `thanos/config` Contains the configuration file `object_store_config.yml` which contains the credentials for the S3 storage. You will need to edit this file.
* `prometheus/data` the database and supporting files for prometheus, see above.

The Thanos Store Gateway container binds 1 volume.
* `thanos/config` Contains the configuration file `object_store_config.yml` which contains the credentials for the S3 storage. You will need to edit this file. Same as above.

The Grafana container binds 3 volumes.
* `grafana/data` Contains grafana generated data.
* `grafana/provisioning` Contains initial settings for datasources and dashboards.
* `grafana/custom` Contains custom ini settings. You will need to edit this file.

The Nginx container binds 2 volumes.  
`nginx/config/nginx.conf` Contains the Nginx configuration file. You will need to edit this file.
`nginx/certs` Conatins the SSL certs. You will need to add or link the certs here.

The docker_exporter binds to the exisiting docker sock at `/var/run/docker.sock`


## Web UI
Nginx provides a reverse proxy to the componets that provide web interfaces but do not provide other forms of security such as SSL or user logins. Nginx will provide at least SSL.  
Accesible pages are:  
* Grafana, has log in methods `metrics.yoursite.fabric-testbed.net/grafana` 
* Prometheus info page `metrics.yoursite.fabric-testbed.net:9090`
* Thanos info page `metrics.yoursite.fabric-testbed.net:10902`  

To disable/enable these pages for public view edit the nginx configuration file. See below.

## Installation
Tested using CentOS 8
### Prerequisites
* git
* docker
* docker-compose
(TODO add or link to docker instructions for centos8)

### Steps
1) Clone this repo
1) Copy all the directories from `example_volumes` to the root directory where the `docker-compose.yml` file is located.
1) Edit the configuration files. See below for details.
1) Run `docker-compose up` in the root directory where the `docker-compose.yml` is located.
1) Goto your server url/grafana to log into Grafana. There will be a default dashboards loaded to monitor the installed system.
1) Add node_exporters to machines you whish to monitor. 
#### TODO add in options on how to install exporters

## Conguration Files That Need To Be Edited

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
* `grafana/env_file`
Add an admin password so you can login to Grafana. `GF_SECURITY_ADMIN_PASSWORD=yourpassword`


