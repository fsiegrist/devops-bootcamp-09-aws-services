## Demo Project - Complete CI/CD Pipeline

### Topics of the Demo Project
Complete the CI/CD Pipeline (Docker-Compose, Dynamic versioning)

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
- CI step: Increment version
- CI step: Build artifact for Java Maven application
- CI step: Build and push Docker image to Docker Hub 
- CD step: Deploy new application version with Docker Compose
- CD step: Commit the version update

#### Steps to implement the complete CI/CD Pipeline
**Step 1:** Compared to the pipeline of the previous demo project 3 only the first stage (incrementing the version) and the last stage (committing the version update) are missing. So we copy these two stages from the final pipeline in module 08 (Build Automation & CI/CD with Jenkins). This results in the following Jenkinsfile for the complete pipeline:

```groovy
#!/usr/bin/env groovy

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage("Increment Version") {
            steps {
                script {
                    echo 'incrementing the bugfix version of the application...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
      
                    def version = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
                    env.IMAGE_TAG = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage("Build Application JAR") {
            steps {
                script {
                    echo "building the application..."
                    sh 'mvn clean package'
                }
            }
        }
        stage("Build and Publish Docker Image") {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                        echo "building the docker image..."
                        sh "docker build -t fsiegrist/fesi-repo:devops-bootcamp-java-maven-app-${IMAGE_TAG} ."
                        
                        echo "publishing the docker image..."
                        sh "echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin"
                        sh "docker push fsiegrist/fesi-repo:devops-bootcamp-java-maven-app-${IMAGE_TAG}"
                    }
                }
            }
        }
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
        stage('Commit Version Update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'GitHub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "git remote set-url origin https://${USERNAME}:${PASSWORD}@github.com/fsiegrist/devops-bootcamp-java-maven-app.git"
                        sh 'git add .'
                        sh 'git commit -m "jenkins: version bump"'
                        sh 'git push origin HEAD:feature/docker-compose'
                    }
                }
            }
        }
    }   
}
```
Of course we could extract all the logic into Jenkins Shared Library functions, but the above form provides a better overview over the tasks to be done in the several stages.

The shell script, called in the stage "Deploy Application" just contains the following lines:

```sh
#!/usr/bin/env/ bash

export IMAGE_TAG=$1
docker-compose -f docker-compose.yaml up -d
echo "successfully started the container using docker-compose"
```
