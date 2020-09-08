
# Prometheus Monitoring Setup
This document covers the installation and configuration of the global metric collection via Prometheus and Thanos.
Each rack in the FABRIC Testbed has a VM running a set of docker containers (Prometheus, Thanos and supporting containers) responsible for collecting metrics from resources located at/near the rack's location. The collected metrics are perodically shipped using Thanos to a global S3 compatable storage.

Prometheus is intended to be a pull system where a Prometheus client will periodically poll resources listed in its config file. Each resource to be monitored hosts a server, the "exporter", which is responsible for gathering and returning data to the Prometheus client.

## Docker Containers

Each Fabric rack will have a VM running a docker-compose set containing Prometheus and its supporting containers. These include:
* **Prometheus**  
The Prometheus Client responsible for polling exporters for metrics.
* **Thanos**  
Thanos consists of several components. For component information see [Thanos Quick Tutorial](https://thanos.io/tip/thanos/quick-tutorial.md/)
  * **Sidecar**  
  Reads the data from the Prometheus database and ships it to the global storage.
  * **Query**  
  Allows for queries to be run on the data via Sidecar (local data) and Gateway (remote S3 stored data)
  * **Gateway**  
  Connects to remote object stores such as the S3 stored data.
* **SNMP exporter**  
Exporter that transforms data from SNMP devices for the Prometheus Client.
* **Docker exporter**  
Exporter for information about the running containers.
* **Grafana**  
Allows for web UI to review the locally collected metrics.
* **Nginx**  
Reverse proxy to enable SSL for Grafana and limit access to web GUIs run by various components/
TODO
```
* **Alert manager**
? or is that being shifted to Thanos Rules?
* **Nginx exporter**
?needed?
```


The Prometheus container maps 2 volumes: 
* `prometheus` configuration files  (list config files and point to example)
* `prometheus_data` the database and supporting data files
https://fabric-testbed.atlassian.net/wiki/spaces/FP/pages/89325610/Ancillary+Infrastructure
The thanos sidecar container maps 2 volumes:
* `thanos` config files (list config files and point to example)
* `prometheus_data` the database and supporting files for prometheus (?problems with double mapping of volume? sharing data between containers?)

(other thanos containers...)

Grafana presents the timeseries data via a web interface. Defaults to port 3000? Is proxied via nginx

Nginx provides a reverse proxy to the componets that provide web interfaces but do not provide other forms of security such as SSL or user logins. Nginx will provide at least SSL. (will add other methods later for users etc...) Accesible pages are:
* Grafana, has log in methods 
* Prometheus info page
* Thanos info page

Docker monitor

## Files That Need To Be Edited
* prometheus/config/prometheus_config.yml  
Set the rack name to the site acronym found at [Fabric Sites](https://fabric-testbed.atlassian.net/wiki/spaces/FP/pages/168624158/FABRIC+Site+Documentation)
* thanos/config/object_store_config.yml  
Set your CEPH access key and secret.
* nginx/config/nginx.conf  
Set you VM url. Set cert filenames to match below certs.
* nginx/certs  
Add the ssl certs for nginx.
* grafana/custom/custom.ini  
Set your server url. 
