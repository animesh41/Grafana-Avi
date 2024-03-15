------------------------------------------------------------
##### Grafana Setup ########
------------------------------------------------------------

The Avi Metrics Collection script was built with the intent to pull metrics from one or more Avi controllers and send these values to a centralized time series database.
In this deployment we are using Grafana with InfluxDB installed  on the same VM. 

### Prerequisites

* Linux machine
* python 3.6+
* python3-yaml
* python3-requests
* The following TCP ports should not already be in use: 2003, 3000,8086 and 8008

### Services

The application stack is broken into three separate services:
* **Influxdb** &mdash; stores numeric time-series data
* **Grafana** &mdash; UI front-end to display the data from Influxdb
* **Avi Metrics Script** &mdash; Python script using Avi REST API to retrieve metrics and insert them to the database created in Influxdb


### Installation steps

## Installing InfluxDB - This is setup with InfluxDB version1

    ## sudo apt-get update && sudo apt-get install influxdb
    ## sudo systemctl unmask influxdb.service
    ## sudo systemctl start influxdb
    ## sudo service influxdb start¯
    ## sudo apt-get install influxdb
    ## sudo service influxdb start

## Install influx client: 
    ## sudo apt install influxdb-client

## Create an InfluxDB to receive data -- Check if all the metrics list

    ##   influx // Run influx client before creating the database 
        ##   CREATE DATABASE your_database_name;
        ##   CREATE USER your_username WITH PASSWORD 'your_password';
        ##   GRANT ALL ON your_database_name TO your_username;

## Install Grafana 

    # sudo apt-get update 
    # sudo apt-get install -y apt-transport-https software-properties-common wget
    # sudo apt-get install -y adduser libfontconfig1 musl
    # wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.2.2_amd64.deb
    # sudo dpkg -i grafana-enterprise_10.2.2_amd64.deb

    Linux
    If you installed Grafana using the deb or rpm packages, then your configuration file is located at /etc/grafana/grafana.ini and a separate custom.ini is not used. This path is specified in the Grafana init.d script using --config file parameter.

    # sudo service grafana-server start
    # sudo service grafana-server status¯
    # sudo systemctl enable grafana-server 

    # During First time login , Set password on the UI , if required can setup multiple profiles.

## Configure Grafana 

* Add the InfluxDB as the Datasource in Grafana
** Ensure the DB name matches of 'influxdb' as per the datasource name in grafana

 ## Avi Metrics Script

* Download the metricscollection.py script and configuration-example.yaml from github location : 
    - https://github.com/avinetworks/devops/tree/master/monitoring_tools/metrics%20collection

## Modify  the configuration.yaml

     # avi_cluster_name: clustername
     # avi_controller: controller ip address
     # avi_user: admin
     # comment: ACCEPTS PLAIN TEXT OR BASE64 ENCODED PASSWORD
     # avi_pass: password
     # tags:
      # environment: dev
      # location: datacenter1    

* configure the virtualservice name
* Add the metrics required : https://avinetworks.com/docs/22.1/metrics-list/
* configure the metrics_endpoint config to influx db
    
        # metrics_endpoint_config:
        # - type: influxdb
        # enable: True
        # server: ip address 
        # server_port: 8086
        # protocol: http
        # db: db name used during database creation within influxdb
        #_comment":"Change metric_prefix only if you need to define a prefix",
        # metric_prefix: ""
        #_comment":"If using auth on influxdb set auth-enabled to true and modify the credential values",
        # auth-enabled: True
        # username: admin
        # password: password


* Create a folder and move the metricscollection.py script and configuration.yaml to it
* Run metricscollection script - command Python3 metricscollection.py

* A sample configuration.yaml is provided

## Setup cronjob to run python script every minute
    ## sudo apt install cron
    ## sudo systemctl enable cron
    ## crontab -e 
        - choose editor 
        * * * * * /usr/bin/python3 /root/metricscollection/metricscollection.py


The job runs every minute to poll the metrics from the controller and put the same into the influxdb. If realtime metrics are enabled then controller collects then data granularity is 5 seconds, with retention period of 1 hour on the controller. 
Cronjob by default runs every minute, for 5 seconds interval a  shell script can be setup to run the python script every 5 seconds.

## Access the Grafana Dashboard

- Access the Grafana on the http://GrafanaVMIP:3000
- Import the dashboards as required  from Devops github : https://github.com/avinetworks/devops/tree/master/monitoring_tools/grafana/influxdb
- On Grafana UI, Select >  Dashboards > New > Import and select the json downloaded from the github to load the file. 
- Modify and configure accordingly 

## Appendix : 

- https://github.com/avinetworks/devops/tree/master/monitoring_tools/avimetrics%20script
- https://github.com/avinetworks/devops/tree/master/monitoring_tools/metrics%20collection
- https://github.com/avinetworks/devops/tree/master/monitoring_tools/grafana/influxdb
- https://avinetworks.com/docs/22.1/metrics-list/
