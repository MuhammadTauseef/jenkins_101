# Installation on Windows 10

## Docker installation on windows step by step guide
https://www.youtube.com/watch?v=aCRMnDLnWmU

## Create bridge network in docker
```
docker network create jenkins
```
## Create Dockerfile

```
FROM jenkins/jenkins:2.414.2-jdk17
USER root
RUN apt-get update && apt-get install -y lsb-release python3
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```

## Build new docker image

```
docker build -t myjenkins-blueocean:2.414.2-1 .
```

## Run the container

```
docker run --name jenkins-blueocean --restart=on-failure --detach `
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 `
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 `
  --volume jenkins-data:/var/jenkins_home `
  --volume jenkins-docker-certs:/certs/client:ro `
  --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.414.2-1
```

## Get the Password
```
docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword
```

## Connect to the Jenkins
Open below link and paste the password obtained from previous step
```
http://localhost:8080/
```

## Freestyle project

Dashboard->New item->Freestyle project

## Jenkins filesystem and workspace

```
docker exec jenkins-blueocean ls a/var/jenkins_home
```
Important folders are nodes, plugins, workspace, users, updates, secrets etc...

## Executing python script from build
```
Dashboard->New item->Freestyle project
Source code management->Git (provide public git repo URL)
Build steps->Execute shell and enter below
python3 hello_world.py
```

## Configure Cloud agent (Docker)

Dashboard->Manage Jenkins->Cloud->Plugins and select Docker then Install

Restart Jenkins when installation is complete

### alpine/socat container to forward traffic from Jenkins to Docker Desktop on Host Machine

```
docker run -d --restart=always -p 127.0.0.1:2376:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
docker ps -n 1 --format "{{.ID}}"
docker inspect <container id from above step> 
```

Save the IPAddress from last command

Dashboard->Manage Jenkins->Cloud->New cloud->Docker cloud details

Enter below value into Docker Host URI
```
tcp://<IPADDRESS VALUE>:2375
```
Test Connection to verify if the connection is fine and then hit save

## Docker Agent template

Dashboard->Manage Jenkins->Cloud->docker->Configure->Docker Agent templates->Add Docker Template

Labels=docker-agent-alpine

Enabled=Selected

Name=docker-agent-alpine

Docker Image=jenkins/agent:alpine-jdk17

Instance capacity=2

Remote File System=/home/jenkins

Then click save

## Running configured job with selected agent
Dashboard->build project->configuration->Restrict where this project can be run
Enter labels of the desired agent
Save



# References
https://github.com/devopsjourney1/jenkins-101

https://stackoverflow.com/questions/47709208/how-to-find-docker-host-uri-to-be-used-in-jenkins-docker-plugin
