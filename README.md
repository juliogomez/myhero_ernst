# MyHero Ernst Service

This is the Vote Processing Service for a basic microservice demo application.
This service subscribes to an MQTT Server where votes are waiting for processing.

Details on deploying the entire demo to a Kubernetes cluster can be found at

* DevOps tutorial - [juliogomez/devops](https://github.com/juliogomez/devops)

The application was designed to provide a simple demo for Kubernetes. It is written as a simple Python Flask application and deployed as a docker container.

Other services are:

* Data - [juliogomez/myhero_data](https://github.com/juliogomez/myhero_data)
* App - [juliogomez/myhero_app](https://github.com/juliogomez/myhero_app)
* UI - [juliogomez/myhero_ui](https://github.com/juliogomez/myhero_ui)
* Ernst - [juliogomez/myhero_ernst](https://github.com/juliogomez/myhero_ernst)
  * Optional Service used along with an MQTT server when App is in "queue" mode
* Spark Bot - [juliogomez/myhero_spark](https://github.com/juliogomez/myhero_spark)
  * Optional Service that allows voting through IM/Chat with a Cisco Spark Bot


The docker containers are available at

* Data - [juliocisco/myhero_data](https://hub.docker.com/r/juliocisco/myhero-data/)
* App - [juliocisco/myhero_app](https://hub.docker.com/r/juliocisco/myhero-app)
* UI - [juliocisco/myhero_ui](https://hub.docker.com/r/juliocisco/myhero-ui)
* Ernst - [juliocisco/myhero_ernst](https://hub.docker.com/r/juliocisco/myhero-ernst)
  * Optional Service used along with an MQTT server when App is in "queue" mode
* Spark Bot - [juliocisco/myhero_spark](https://hub.docker.com/r/juliocisco/myhero-spark)
  * Optional Service that allows voting through IM/Chat with a Cisco Spark Bot

## Environmental Requirement

This service requires an MQTT server or broker to be available that can be subscribed, where votes will be published by the APP service.  The larger [MyHero Demo](https://github.com/juliogomez/devops) application leverages [Mosca](https://hub.docker.com/r/matteocollina/mosca/) as a MQTT server for this purpose, though any MQTT server or broker should work.

The Vagrant environment for local development of this service deploys a Mosca container along with the ernst service itself and leverages it for local development work.

 **NOT DONE YET**

## Basic Application Details

Required

* flask
* ArgumentParser
* requests
* paho-mqtt
* dnspython

# Environment Installation

    pip install -r requirements.txt

# Basic Usage

In order to run, the service needs 3 pieces of information to be provided:

* Data Server Address
* Data Server Authentication Key to Use
* MQTT Server IP and Port
  * This can be determined in 2 different ways
    * Explicitly Set
    * Dynamically learned by doing an DNS Lookup

These details can be provided in one of three ways.

* As a command line argument
  - `python myhero_ernst/myhero_ernst.py --dataserver "http://myhero-data.server.com" --datakey "DATA AUTH KEY" --appsecret "APP AUTH KEY" --mqtthost "192.168.1.10" --mqttport "1883" --mqttserver "mqtt.service.consul" `
* As environment variables
  - `export myhero_data_server="http://myhero-data.server.com"`
  - `export myhero_data_key="DATA AUTH KEY"`
  - `export myhero_mqtt_host=192.168.1.10`
  - `export myhero_mqtt_port=1883`
  - `export myhero_mqtt_server=mqtt.service.consul`
  - `python myhero_ernst/myhero_ernst.py`
* As raw input when the application is run
  - `python myhero_ernst/myhero_ernst.py`
  - `What is the data server address? http://myhero-data.server.com`
  - `Data Server Key: DATA AUTH KEY`
  - `App Server Key: APP AUTH KEY`

A command line argument overrides an environment variable, and raw input is only used if neither of the other two options provide needed details.

**For determining the MQTT Server IP and Port, explicitly setting them through Command Line Arguements or through ENV variables will be preferred over a DNS lookup**

# Alternate and Advanced Configurations

## Finding Data Server Details with SRV Lookup

If in your deployment, the myhero_data microservice is deployed in a way that the data server address (ie IP and Port) are dynamic, there is support for querying an SRV record to determine the details.

An example of this type of setup would be deploying MyHero to a Kubernetes cluster.  Rather than hard code in the address of the data server, you would query Kubernetes name resolution service for the address infromation.

To use this method, you will provide a different argument or environment variable to the program.

* As a command line argument
  - `python myhero_ernst/myhero_ernst.py --datasrv "data-myhero.service.consul" --datakey "DATA AUTH KEY" --appsecret "APP AUTH KEY" --mqtthost "192.168.1.10" --mqttport "1883" --mqttserver "mqtt.service.consul" `
* As environment variables
  - `export myhero_data_srv"data-myhero.service.consul"`
  - `export myhero_data_key="DATA AUTH KEY"`
  - `export myhero_mqtt_host=192.168.1.10`
  - `export myhero_mqtt_port=1883`
  - `export myhero_mqtt_server=mqtt.service.consul`
  - `python myhero_ernst/myhero_ernst.py`


# Accessing

Initial and Basic APIs.
These are v1 APIs that require no authentication and will eventually be removed

* Basic List of Hero Choices
  * `curl http://localhost:5000/hero_list`
* Current results calculations
  * `curl http://localhost:5000/results`
* Place a vote for an option
  * `curl http://localhost:5000/vote/<HERO>`

New v2 APIs
These newer APIs require authentication as well as support more features

* Get the current list of options for voting
  * `curl -X GET -H "key: APP AUTH KEY" http://localhost:5000/options`
* Add a new option to the list
  * `curl -X PUT -H "key: APP AUTH KEY" http://localhost:5000/options -d '{"option":"Deadpool"}'`
* Replace the entire options list
  * `curl-X POST -H "key: APP AUTH KEY" http://localhost:5000/options -d @sample_post.json`
  * Data should be of same format as a GET request
* Delete a single option from the list
  * `curl -X DELETE -H "key: APP AUTH KEY" http://localhost:5000/options/Deadpool`
* Place a Vote for an option
  * `curl -X POST -H "key: APP AUTH KEY" http://localhost:5000/vote/Deadpool`
* Get current results
  * `curl -X GET -H "key: APP AUTH KEY" http://localhost:5000/results`

# Local Development with docker-compose

I've included the configuration files needed to do local development with docker-compose in the repo.  docker-compose will still use Docker for local development and requires the following be installed on your laptop: 

* [Docker](https://www.docker.com/community-edition)

To start local development run:

* `docker-compose up`
* Now you can interact with the API or interface at localhost:15001 (configured in docker-compose.yml)
  - example:  from your local machine `curl -H "key: DevApp" http://localhost:15001/options`
  - Environment Variables are configured in .env for development

Each of the services in the application (i.e. myhero_web, myhero_app, and myhero_data) include docker-compose support to allow working locally on all three simultaneously.

# Local Development with Vagrant

I've included the configuration files needed to do local development with Vagrant in the repo.  Vagrant will still use Docker for local development and requires the following be installed on your laptop: 

* [Vagrant 2.0.1 or higher](https://www.vagrantup.com/downloads.html)
* [Docker](https://www.docker.com/community-edition)

To start local development run:

* `vagrant up`
* Now you can interact with the API or interface at localhost:15001 (configured in Vagrantfile)
  - example:  from your local machine `curl -H "key: DevApp" http://localhost:15001/options`
  - Environment Variables are configured in Vagrantfile for development

Each of the services in the application (i.e. myhero_web, myhero_app, and myhero_data) include Vagrant support to allow working locally on all three simultaneously.