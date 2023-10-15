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
RUN apt-get update && apt-get install -y lsb-release
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

# Reference
https://github.com/devopsjourney1/jenkins-101
