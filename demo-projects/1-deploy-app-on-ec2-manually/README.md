## Demo Project - Manually Deploy App on EC2

### Topics of the Demo Project
Deploy Web Application on EC2 Instance (manually)

### Technologies Used
- AWS
- Docker
- Linux

### Project Description
- Create and configure an EC2 Instance on AWS
- Install Docker on remote EC2 Instance
- Deploy Docker image from private Docker repository on EC2 Instance

#### Steps to Create and Configure an EC2 Instance on AWS
**Step 1:** Login as admin user to the AWS management web console. Go to "Services" > "Compute" > "EC2". Scroll down to the "Launch instance" section, press the "Launch instance" button and select "Launch instance". This leads you to a page where you can configure the new instance.

**Step2:** Enter a name (e.g. web-server) and the "Add additional tags" link. Press the "Add tag" button and enter the key-value-pair "Type" -> "web-server-with-docker".

**Step 3:** Scroll down to "Application and OS Images (Amazon Machine Image)" and select the machine image "Amazon Linux". Next you can select from a large list of instance types. Select the free tier eligible "t2.micro".

**Step 4:** To be able to ssh into the EC2 server we have to generate a key pair. We create the key pair on AWS and download the private key. The public key is automatically stored in the right place.\
Scroll down to "Key pair (login)" and click the "Create new key pair" link. Enter the name 'docker-server', select "RSA" and ".pem" and press the "Create key pair" button. The `docker-server.pem` file holding the private key is automatically downloaded.

**Step 5:** In the "Network settings" section we could choose a VPC and a subnet (availability zone), but we leave the defaults unchanged. Make sure "Auto-assign public IP" is enabled. Select "Create security group", change the security group name to 'security-group-docker-server' and the description accordingly, leave the ssh firewall rule unchanged but modify the source type from "Anywhere" to "My IP".

**Step 6:** Leave the defaults in the "Configure storage section" unchanged.

**Step 7:** In the summary column on the right you could change the number of instances to be created, but we leave it at the default value of 1. Press the "Launch instance" button.

#### Steps to Install Docker on the EC2 Instance
**Step 1:** SSH into the EC2 instance\
Move the downloaded 'docker-server.pem' file into the ssh folder `~/.ssh` and restrict the file permissions 'read for you only': `chmod 400 ~/.ssh/docker-server.pem`.\
Open the AWS web console, go to "EC2 Dashboard" > "Instances" and check the 'web-server' instance. Select the "Networking" tab below and copy the public IPv4 address. Now open a terminal on your local machine and ssh into the EC2 server as 'ec2-user':\
`ssh -i ~/.ssh/docker-server.pem ec2-user@35.156.226.244`

**Step 2:** Install Docker on EC2
Execute the following commands on the EC2 terminal:
```sh
sudo yum update
sudo yum install docker
sudo service docker start
# add the ec2-user to the docker group 
# to avoid having to use sudo for every docker command
sudo usermod -aG docker ec2-user
# the last command will be effective only after a re-login
exit
```

#### Steps to Deploy Docker Image from Private Docker Repository on EC2 Instance
**Step 1:** Clone the 'react-nodejs-example' application from [GitHub](https://github.com/nanuchi/react-nodejs-example). Build a Docker image and push the image to your private Docker registry. Because my local machine has an Apple M2 processor (arm64) and the EC2 virtual machine we just created has an amd64 processor, the usual 'docker build' command would create an image runnable on arm64 only. So I would either have to slightly modify the Dockerfile and replace 'FROM node:10' with 'FROM --platform=linux/amd64 node:10' or I can use [docker buildx](https://docs.docker.com/engine/reference/commandline/buildx/) to build images for specific platforms:
```sh
docker buildx create --use
docker login
docker buildx build --platform linux/amd64,linux/arm64 -t fsiegrist/fesi-repo:devops-bootcamp-react-nodjs-1.0 --push .
```

**Step 2:** Now switch back to the EC2 terminal, login to DockerHub, pull the image and start a container from it:
```sh
ssh -i ~/.ssh/docker-server.pem ec2-user@35.156.226.244

docker login
docker pull --platform linux/amd64 fsiegrist/fesi-repo:devops-bootcamp-react-nodjs-1.0
docker run -d -p 3000:3080 fsiegrist/fesi-repo:devops-bootcamp-react-nodjs-1.0
```

**Step 3:** Make the app accessible from the browser\
Open the AWS web console, go to "EC2 Dashboard" > "Instances" and check the 'web-server' instance. Select the "Security" tab below and click on the link for the 'security-group-docker-server'. Open the "Inbound rules" tab and press the "Edit inbound rules" button. Press "Add rule" and enter a rule of type "Custom TCP" for port 3000 with source "Anywhere-IPv4". Press "Save rules".

Now open the browser and navigate to `http://35.156.226.244:3000` to see the application in action.
