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

### Create Docker image

We will use multistage Docker in our project

```
FROM openjdk:8 AS BUILD_IMAGE
RUN apt update && apt install maven -y
RUN git clone -b vp-docker https://github.com/imranvisualpath/vprofile-repo.git
RUN cd vprofile-repo && mvn install

FROM tomcat:8-jre11

RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=BUILD_IMAGE vprofile-repo/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
```
We will create artifact with maven then copy this artifact to another tomcat server and run it.


### Update Jenkinsfile

We will need to add 3 more variable to in order to upload Container to ECS and following commands in order to upload artifacts to ECR.

```
registryCredentials = "ecr-us-east-1:cicdjenkins"
appRegistry = "866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops"
applicationRegistry = "https://866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops"

```

```
        stage("Build Docker Image"){
            steps{
                script{
                    dockerImage = docker.build(imagename + ":$BUILD_ID" + "_$BUILD_TIMESTAMP","./Docker-files/app/multistage/")
                }
            }
        }
        stage("Upload Docker to ECR"){
            steps{
                script{
                    docker.withRegistry (applicationRegistry,registryCredentials){
                        dockerImage.push("$BUILD_ID")
                        // dockerImage.push("$BUILD_TIMESTAMP")
                        dockerImage.push("latest")
                    }
                }
            }
        }

```
As a result following logs will be generated after successfull build.

```
$ docker login -u AWS -p ******** https://866308211434.dkr.ecr.us-east-1.amazonaws.com
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /var/lib/jenkins/workspace/CI_CD_PIPELINE@tmp/40547163-9e27-4671-8cc4-cded0493e5a7/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[Pipeline] {
[Pipeline] isUnix
[Pipeline] withEnv
[Pipeline] {
[Pipeline] sh
+ docker tag 866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops:36_2022-11-17_13_06 866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops:36
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] isUnix
[Pipeline] withEnv
[Pipeline] {
[Pipeline] sh
+ docker push 866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops:36
The push refers to repository [866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops]
aa2e51f5ab8a: Preparing
f4a670ac65b6: Preparing
9a797133ad85: Waiting
13ec7f04879a: Waiting
76d6ce51a35d: Layer already exists
8b4173f33a28: Layer already exists
0439bc492ca9: Layer already exists
36: digest: sha256:9e3b339ae352612fb1059e1977909a0bcfd4c0e13f2fb750feabf31d9797f1a3 size: 2209
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] isUnix
[Pipeline] withEnv
[Pipeline] {
[Pipeline] sh
+ docker tag 866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops:36_2022-11-17_13_06 866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops:latest
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] isUnix
[Pipeline] withEnv
[Pipeline] {
[Pipeline] sh
+ docker push 866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops:latest
The push refers to repository [866308211434.dkr.ecr.us-east-1.amazonaws.com/zhajili_devops]

```
We can see that **BUILD_ID** and **BUILD_TIMESTAMP** is appended to docker image as tag ,we can see that our **latest** tag is also added to docker image.






