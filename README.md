# Jenkins in Docker with Nexus and DockerHub Integration

This repository documents the setup and configuration of a Jenkins instance running inside a Docker container. The Jenkins instance is configured to build and push Docker images to Nexus and DockerHub.

## Prerequisites

- Docker installed on your host machine
- An active Nexus repository (for storing Docker images)
- DockerHub account

## Running Jenkins in Docker

To start a Jenkins container, run:

sh
docker run -d --name jenkins \
  -p 8088:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:latest


Once the container is running, access Jenkins at:  
[http://localhost:8088](http://localhost:8088)

## Configuring Jenkins

### 1. Install Jenkins
- Open http://localhost:8088
- Complete the installation using the UI.

### 2. Install Docker Inside the Jenkins Container

sh
docker exec -it jenkins bash
curl -fsSL https://get.docker.com -o dockerinstall
chmod 777 dockerinstall
./dockerinstall
chmod 666 /var/run/docker.sock


### 3. Configure Docker Daemon
- Enter the container as root:

sh
docker exec -u root -it jenkins bash


- Edit the Docker daemon configuration:

sh
vim /etc/docker/daemon.json


- Add the following:

json
{
  "insecure-registries": ["localhost:8083"]
}


- Restart Docker:

sh
systemctl restart docker


### 4. Install JDK and Maven

Inside the container:

sh
apt update && apt install -y openjdk-11-jdk maven


### 5. Create Jenkins Credentials
- *DockerHub Credentials*:  
  - ID: docker-hub
  - Username: <your-dockerhub-username>
  - Password: <your-dockerhub-password>

- *Nexus Credentials*:  
  - ID: nexus
  - Username: <your-nexus-username>
  - Password: <your-nexus-password>

## Creating a Freestyle Pipeline

1. Open Jenkins UI → Create a new Freestyle Project
2. Configure *Source Code Management*:
   - Repository URL: https://gitlab.com/......
   - Branch: */master

3. Configure *Build Triggers*:
   - Enable SCM Polling (optional)

4. Configure *Build Steps*:
   - Add a *Maven build step*:
     - Goals: package
   - Add a *Shell script* to build and push the Docker image:

sh
docker build -t abmrv/demo-java-app:1.5 .
docker login -u $USER_NAME -p $USER_PASSWORD
docker push abmrv/demo-java-app:1.5


5. Configure *Post-Build Actions*:
   - Add another *Shell script* to push the image to Nexus:

sh
docker build -t localhost:8083/java-maven:1.5 .
docker login -u $USER -p $PASSWORD localhost
docker push localhost:8083/java-maven:1.5


6. Save and Run the Pipeline.

## Access Jenkins

Jenkins UI: [http://localhost:8088](http://localhost:8088)

Environment Variables: [http://localhost:8088/env-vars.html](http://localhost:8088/env-vars.html)

---
