# Automation-Jenkins-Setup

Docker Setup to run UI automation in Jenkins.

[![Made with Docker](https://img.shields.io/badge/Made%20with-Docker-blue.svg)](https://www.ruby-lang.org/en/)
[![Made with Ruby](https://img.shields.io/badge/Made%20with-Ruby-red.svg)](https://www.ruby-lang.org/en/)
[![email me](https://img.shields.io/badge/Contact-Email-green.svg)](nareshnavinash@gmail.com)

![alt text](jenkins-master/Docker_Jenkins.png)

## Initiate:
To initiate the full setup,
```
docker-compose -p jenkins up -d nginx master proxy
```
## Log:
To check for the logs of all the images,
```
docker-compose -p jenkins exec master tail -f /var/log/jenkins/jenkins.log
```
## Kill:
To kill the whole setup,
```
docker-compose -p jenkins down -v
```
## High level Architecture:
* Master docker in which jenkins is installed.
* Two volumes are created for the master jenkins, so that even if the server gets down jenkins data will be retained.
* Have nginx server with jenkins configuration
* Have a proxy and network to link the jenkins with volumes and the child dockers
* Slave docker files to run the selenium tests with specific configuration, where chrome and firefox is installed with selenium standalone server and respective drivers.
