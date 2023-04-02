## Demo Project - Deploy App on EC2 from Jenkins Pipeline (using Docker)

### Topics of the Demo Project
CD - Deploy Application from Jenkins Pipeline to EC2 Instance (automatically with docker)

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
- Prepare AWS EC2 Instance for deployment (Install Docker)
- Create ssh key credentials for EC2 server on Jenkins
- Extend the previous CI pipeline with deploy step to ssh into the remote EC2 instance and deploy newly built image from Jenkins server
- Configure security group on EC2 Instance to allow access to our web application

#### Steps to prepare AWS EC2 instance for deployment (install Docker)
We already installed Docker on EC2. See [Demo Project 1, Steps to Install Docker on the EC2 Instance](../1-deploy-app-on-ec2-manually/README.md#steps-to-install-docker-on-the-ec2-instance).

#### Steps to create ssh key credentials for EC2 server on Jenkins
**Step 1:** Login to the Jenkins management web console, go to "Dashboard" > "Manage Jenkins" > "Manage Plugins" and install the "SSH Agent" plugin.

**Step 2:** Then open the multibranch pipeline ("Dashboard" > "devops-bootcamp-multibranch-pipeline"), open the pipeline specific "Credentials", scroll down to "Stores scoped to devops-bootcamp-multibranch-pipeline" and click on the devops-bootcamp-multibranch-pipeline link and then on the "Global credentials (unrestricted)" link. Press the "Add credentials" button, select the kind "SSH Username with private key", enter an ID (e.g. ec2-server-key), the username 'ec2-user', select "Private Key" > "Enter directly", press the "Add" button and paste the content of the `~/.ssh/docker-server.pem` file you downloaded from the EC2 server. (To copy the content on a Mac without having to display it on the terminal, use `pbcopy < ~/.ssh/docker-server.pem`.) Press the "Create" button.

**Step 3:** To allow Jenkins to ssh into the EC2 server, we have to add the IP address of the Jenkins host (droplet) to the firewall rule restricting access via port 22. Login as admin user to the AWS management web console. Go to "EC2 Dashboard" > "Instances" and check the 'web-server' instance. Select the "Security" tab below and click on the link for the 'security-group-docker-server'. Open the "Inbound rules" tab and press the "Edit inbound rules" button. Press "Add rule" and enter a rule of type "SSH" for port 22 with source "Custom" and enter the IP address of the Jenkins server (droplet) followed by `/32` (`64.225.104.226/32`). Press "Save rules".

#### Steps to extend the CI pipeline with deploy step to deploy newly built image from Jenkins server
**Step 1:** Open the Jenkinsfile in the application project, which is built in the multibranch pipeline (java-maven-app) and add the following stage:
```groovy
stage('Deploy Application') {
    steps {
        script {
            echo 'deploying Docker image to EC2 server...'
            def dockerCmd = 'docker run -d -p 8000:8080 fsiegrist/fesi-repo:devops-bootcamp-java-maven-app-1.0.1'
            sshagent(['ec2-server-key']) {
                sh "ssh -o StrictHostKeyChecking=no ec2-user@35.156.226.244 ${dockerCmd}"
            }
        }
    }
}
```

The option `-o StrictHostKeyChecking=no` is necessary to avoid ssh asking whether the server should be added to the known hosts.

**Step 2:** To allow EC2 to pull a Docker image from our private repository on DockerHub, we have to login from EC2 to DockerHub once.
```sh
ssh -i ~/.ssh/docker-server.pem ec2-user@35.156.226.244
  docker login
```
This will create an entry in `/home/ec2-user/.docker/config.json` and keep the ec2-user logged in.


#### Steps to configure security group on EC2 instance to allow access to our web application
To allow accessing the application from the internet, we have to add a firewall rule of type "Custom TCP" opening the port 8000 from anywhere (source 'Anywhere-IPv4, `0.0.0.0/0`).

Now open the browser and navigate to `http://35.156.226.244:8000` to see the application in action.
