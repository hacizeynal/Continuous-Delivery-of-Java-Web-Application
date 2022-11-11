# Continuous Delivery of Java Web Application

In the previous [project](https://github.com/hacizeynal/Continuous-Integration-Using-Jenkins-Nexus-Sonarqube-Slack)
, we built a Continuous Integration pipeline on Jenkins .In this project we will continue to build Continuous Delivery project ,which means that we will need to deploy code automatically . We will use following tools for the project 

* Jenkins
* Maven
* Checkstyle
* Slack 
* SonaType Nexus
* SonarQube
* Docker
* ECR
* ECS
* AWS CLI

High level overview for the project is described below

[![Screenshot-2022-11-09-at-09-08-59.png](https://i.postimg.cc/pTQQxK6L/Screenshot-2022-11-09-at-09-08-59.png)](https://postimg.cc/47dHQHGD)

## STEPS

[![Screenshot-2022-11-09-at-20-38-21.png](https://i.postimg.cc/MTQ503xG/Screenshot-2022-11-09-at-20-38-21.png)](https://postimg.cc/YGtQM3vc)

[![Screenshot-2022-11-09-at-20-39-27.png](https://i.postimg.cc/sgMhZQ3j/Screenshot-2022-11-09-at-20-39-27.png)](https://postimg.cc/Xr0JRvZm)

### Create IAM User

We will create new IAM user **cicdjenkins** ,this user will access to ECR repository and ECS service.
Jenkins will run AWS CLI from pipeline and this user will execute commands for ECR and ECS.

### Configure Elastic Container Registry

We will create private artifact repository on AWS and upload our generated artifacts from Jenkins pipeline to ECR.

[![Screenshot-2022-11-11-at-20-22-36.png](https://i.postimg.cc/50QWsDgJ/Screenshot-2022-11-11-at-20-22-36.png)](https://postimg.cc/dkFf1HgN)

### Configure Additional Settings on Jenkins

We will configure additional configuration in our previous Jenkins server

* Install necessary plugins (Docker Pipeline ,CloudBees Docker Build and publish,AWS ECR,Pipeline: AWS Steps)
* Store new credentials on it (credentials for cicdjenkins)
* Install Docker Engine

Since we will install correct plugins in Jenkins ,we will see **AWS Credentials** type of credentials in Jenkins.


