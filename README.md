<!-- <p align="center">
  <img src="https://crazymax.dev/diun/assets/meta/card.png" alt="Diun Logo" height="200"/>
</p> -->

# <img src="https://drive.usercontent.google.com/download?id=1KbYhPopR37y50wHRMne7FRKLLUN-usi1" height="25"> Tyzen9's InfluDB and Grafana
This compose stack provides InfluDB and Grafana support.  This documentation assumes that these will be used with Home Assistant.

## Prerequisites
Install [Docker Engine](https://docs.docker.com/get-docker/) or [Docker Desktop](https://docs.docker.com/desktop/) if you require the Docker user interface.  In production it's generally best to use [Docker Engine](https://docs.docker.com/get-docker/) on a Linux host operating system, and a lightweight service delivery platform designed for managing containerized applications such as [Portainer](https://www.portainer.io/)

This documentation assumes you have a working knowledge of [Docker](https://www.docker.com/), and [Home Assistant](https://www.home-assistant.io/).

## Configuration
This `docker-compose` implementation is configured using the `environment` section of the `docker-compose.yaml` file.  


## Start the Container (Development/Testing)
1. Clone this repository: `git clone https://github.com/tyzen9/docker-diun.git`
1. Set the configuration options as desired in the `docker-compose.yaml` and `.env` files.
1. Navigate to the project's root directory and run the following command:

```
docker compose up
```

# InfluxDB Configuration

## Preparing for HA Connection
1. Log into InfluxDB at http://localhost:8086, using the username and password as defined in the `.env` file
1. Click the `Load Data` option on the left, and choose `API Tokens`
1. Click `Generate API Token` and choose `All Access API Token`
1. Give the token a description, and click `SAVE`
1. Copy the resulting `API token string` someplace safe, as you will not be able to access it again

## Connecting HA to InfluxDB
1. Open the `configuration.yaml` file in HA
1. Using the [InfluxDB Ingration guide](https://www.home-assistant.io/integrations/influxdb/), enter the following as a starting point for the InfluxDB 2.0 configuration. Make sure to substitute the appropriate values in for the square brackets.

```yaml
influxdb:
  api_version: 2
  ssl: false
  host: [InfluxDB Host Name]
  port: [InfluxDB Port]
  token: [API token string]
  organization: [InfluxDB Org]
  bucket: [InfluxDB Bucket]
  tags:
    source: HA
  tags_attributes:
    - friendly_name
  default_measurement: units
  exclude:
    entities:
      - zone.home
    domains:
      - persistent_notification
      - person
  include:
    domains:
      - sensor
      - binary_sensor
      - sun
    entities:
      - weather.home
```
### Finite control
It is possible to be very specific about individual entities that pushed to InfluxDB.  This may be desireable should you wish to control the size and content of your influxDB.  In the example below, I am only allowing for the dryer's exhaust temperature to be pushed to InfluxDB, and will need to add other sensors and restart HA as needed.  The absense of an `exclude` option, tells Home Assistant to ONLY send the items listed in the `include` configuration.

```yaml
influxdb:
  api_version: 2
  ssl: false
  host: [InfluxDB Host Name]
  port: [InfluxDB Port]
  token: [API token string]
  organization: [InfluxDB Org]
  bucket: [InfluxDB Bucket]
  tags:
    source: HA
  tags_attributes:
    - friendly_name
  default_measurement: units
  include:
    entities:
      - sensor.sensor_tumbledryer_dryer_exhaust_temperature
```

# Connecting Grafana

1. Visit Grafana at http://localhost:3000
1. Login with the username: `admin` and password: `admin`
1. If this is your first login, set the new `admin` password and click `submit`
1. In the left menu select `Connections` and `Add new connection`
1. Choose `InfluxDB` from the extensive list to the right
1. In the upper right choose `Add new data source`
1. At the top, give this connection a name
1. In the `Query language` section: choose `Flux` as the query language
1. In the `HTTP` Section: Populate the URL to InfluxDB (in this example this is http://localhost:8086/)
1. In the `InfluxDB Details` setion, enter the `Organization` the `Token` and the `Default Bucket` name.
1. Click `Save & test` and hopefully all is well


## Production Deployment using Portainer
I use [Portainer](https://www.portainer.io/) to manage and orchestrate my Docker resources in my Production environments. To deploy this into your Portainer environment:

1. In Portainer, choose the Environment to deploy to
1. In the left menu choose Stacks, then click `+ Add stack`
1. Choose `Repository` and fill in the required fields (don't forget name at the top)
1. At the bottom, there is an option to `Load variables from .env file`. Click this button and provide the prepared `.env` file for this environment.
1. Click `Deploy the stack`

Once successfully deployed, it would be a good idea to test the notification (See above) as there is no Diun dashboard.