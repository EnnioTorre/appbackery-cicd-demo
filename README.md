# Appbackery-cicd-demo

This repository contains artifacts definition for continuous delivery using Jenkins, Openshift, Quay.io, Prometheus, Grafana. 

* [Introduction](#introduction)
* [Scenario](#scenario)
* [technology stack](#technologystack)
* [Automated Deploy on OpenShift](#automatic-deploy-on-openshift)
* [Demo Guide](#demo-guide)


## Introduction

![Diagram](https://github.com/EnnioTorre/appbackery-cicd-demo/blob/master/pipeline.drawio)

On every pipeline execution, the code goes through the following steps:

1. Code is cloned from GitHub
2. Application is built with maven and the WAR artifact is archived in Jenkins
3. Unit tests are executed and the results archived in Jenkins
4. Integration test are executed and the results archived in Jenkins
5. A container image (:dev) is built and the image is pushed to Quay.io image registry and a security scan is scheduled [Quay.io](https://quay.io/repository/enniotorre/demobackery?tab=tags)
6. Application WAR artifact is deployed on Tomcat in DEV project (pulled form Quay.io)
7. Scalability Test are executed against the deployed application and results are archived in Jenkins
5. If tests successful, the pipeline is paused for the release manager to approve the release to PROD
6. If approved, the DEV image is tagged as PROD in the Quay.io image repository using [Skopeo](https://github.com/containers/skopeo)
6. The PROD image is deployed on tomcat in the PROD project (pulled form Quay.io, tag PROD)

The application is a vaadin application forked form :
[https://github.com/igor-baiborodine/vaadin-demo-bakery-app.git](https://github.com/igor-baiborodine/vaadin-demo-bakery-app.git)

## Scenario
A development team is responsible of the development and deployment of a kubernetes native application, this CI/CD pipeline build test and deploy the application to PROD, and, if configured, automatically executed whenever a new commit hits the master branch.


## Technology stack
* Openshift: well that was an easy decision, it includes the tools needed to manage the application lifecycle and provides an end-to-end solution for building complete deployment  pipelines and monitoring.
* Jenkins: it is well integrated in Openshift and provides a easy way to publish all the pipeline artifacts and reports: application WAR, testing results etc... .
* Quay.io: actually I wanted to try it out, it provides a lot of cool features, among which the possibility to inspect and pull the images produced by appbackery pipeline.
* Headless Chrome: mandatory when it comes to run selenium tests, see the image stored in the docker folder for more details.
* Tomcat: straightforward choice with spring boot2.
* Spring Boot Actuator:  includes a number of endpoints to help you monitor your application: `/health`, `/prometheus`. the latest in particular exposes the JVM metrics which are scraped by Prometheus.
* Prometheus+Grafana: very powerful monitoring stack, both are integrated in Openshift4 by means of Operators. Moreover grafana provides a waste choice of dashboards available online [Dashboards](https://grafana.com/grafana/dashboards). 

## Automated Deploy on OpenShift 4
Use the `provisioning/deploy.sh` script provided to deploy the entire demo:

  ```
  ./deploy.sh --help
  ./deploy.sh deploy 
  ./deploy.sh delete 
  ```
Full deploymnet with monitoring:

  ```
  ./deploy.sh deploy --enable-monitoring
  ```

It creates and DEV and PROD project and deploy the monitoring stack. 

## Demo Guide

* A Jenkins pipeline is pre-configured which clones the application from [https://github.com/enniotorre/vaadin-demo-bakery-app.git](https://github.com/enniotorre/vaadin-demo-bakery-app.git).

    You can also explore the pipeline job in Jenkins by clicking on the Jenkins route url, logging in with the OpenShift credentials and clicking on _tasks-pipeline_ and _Configure_.

* Run an instance of the pipeline by starting the _tasks-pipeline_ in OpenShift or Jenkins.

* Inspect the logs by clicking on _Logs_

* Inspect the built images on  [Quay.io](https://quay.io/repository/enniotorre/demobackery) once that the built is finished

* Pipelines pauses at _Deploy to PROD_ step for approval in order to promote the DEV image to the PROD environment. Click on this step on the pipeline and then _Promote_.
![](images/pipeline.png?raw=true)

* After pipeline completion, you can :
  * Explore the _Unit test results_ 
  
  ![](images/junit-analysis.png?raw=true)

  * Explore Integration Tests results 
  
  ![](images/integration-analysis.png?raw=true)

  * Explore Stress Tests results
  
   ![](images/gateling-analysis.png?raw=true)


  * Explore _Demobackery - Dev_ project in OpenShift console and verify the application is deployed in the DEV environment
  * Explore _Demobackery - Prod_ project in OpenShift console and verify the application is deployed in the PROD environment  
  * Check Grafana to monitor the deployed application, in order to see some workload (due to the stress test) and only for demo purposes, the monitoring stack is deployed in  _Demobackery - Dev_ 
  
  ![](images/monitoring.png?raw=true).

