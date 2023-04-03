## Demo Project - Deploy App on EC2 from Jenkins Pipeline (using Docker Compose)

### Topics of the Demo Project
CD - Deploy Application from Jenkins Pipeline on EC2 Instance (automatically with docker-compose)

### Technologies Used
- AWS
- Jenkins
- Docker
- Linux
- Git
- Java
- Maven
- Docker Hub

### Project Description
- Install Docker Compose on AWS EC2 Instance
- Create docker-compose.yml file that deploys our web application image
- Configure Jenkins pipeline to deploy newly built image using Docker Compose on EC2 server
- Improvement: Extract multiple Linux commands that are executed on remote server into a separate shell script and execute the script from Jenkinsfile

#### Steps to install Docker Compose on AWS EC2 instance
**Step 1:** Download Docker Compose:\
```sh
# ssh into the ec2 server
ssh -i ~/.ssh/docker-server.pem ec2-user@35.156.226.244

# download docker-compose into /usr/local/bin
sudo curl -SL https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

# make the docker-compose command executable
sudo chmod +x /usr/local/bin/docker-compose

# test the installation:\
docker-compose --version
```

#### Steps to areate a docker-compose.yml file that deploys our web application image
**Step 1:** Add a file called `docker-compose.yaml` with the following content to the 'java-maven-app' project:
```yaml
version: '3.9'
services:
  java-maven-app:
    image: fsiegrist/fesi-repo:devops-bootcamp-java-maven-app-${IMAGE_TAG}
    ports:
      - 8000:8080
```
Note that the image tag value is taken from an environment variable which must be set before calling docker-compose. Of course the real benefit of using docker-compose over simple 'docker run' commands would manifest only when starting more than just one container.

#### Steps to configure Jenkins pipeline to deploy newly built image using Docker Compose on EC2 server
**Step 1:** Adjust the deploy stage of the application's Jenkinsfile to the following content:
```groovy
stage('Deploy Application') {
    steps {
        script {
            echo 'deploying Docker image to EC2 server...'
            def dockerComposeCmd = "IMAGE_TAG=${IMAGE_TAG} docker-compose -f docker-compose.yaml up -d"
            sshagent(['ec2-server-key']) {
                sh 'scp -o StrictHostKeyChecking=no docker-compose.yaml ec2-user@${ec2-ip-address}:/home/ec2-user'
                sh "ssh -o StrictHostKeyChecking=no ec2-user@35.156.226.244 ${dockerComposeCmd}"
            }
        }
    }
}
```
Note that the env variable `IMAGE_TAG` must be set earlier in the Jenkinsfile.\
Commit and push the changes to the Git repository (on a new branch 'feature/docker-compose'). This will trigger the multibranch build pipeline on Jenkins.

#### Steps to extract multiple Linux commands that are executed on remote server into a separate shell script
**Step 1:** Add a shell script called `server-cmds.sh` with the following content to the application project:
```sh
#!/usr/bin/env/ bash

export IMAGE_TAG=$1
docker-compose -f docker-compose.yaml up -d
echo "successfully started the container using docker-compose"
```
The value to be exported as environment variable `IMAGE_TAG` must be past as first parameter to the script. 

**Step 2:** Adjust the deploy stage of the application's Jenkinsfile to the following content:
```groovy
stage('Deploy Application') {
    steps {
        script {
            echo 'deploying Docker image to EC2 server...'
            def shellCmd = "bash ./server-cmds.sh ${IMAGE_TAG}"
            sshagent(['ec2-server-key']) {
                sh 'scp -o StrictHostKeyChecking=no server-cmds.sh docker-compose.yaml ec2-user@35.156.226.244:/home/ec2-user'
                sh "ssh -o StrictHostKeyChecking=no ec2-user@35.156.226.244 ${shellCmd}"
            }
        }
    }
}
```
Commit and push the changes to the Git repository to start the build pipeline on Jenkins.
