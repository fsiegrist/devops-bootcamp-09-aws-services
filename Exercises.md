## Exercises
<br />

Your company decided that they will use AWS as a cloud provider to deploy their applications. It's too much overhead to manage multiple platforms, including the billing etc.

So you need to deploy the previous NodeJS application on an EC2 instance now. This means you need to create and prepare an EC2 server with the AWS Command Line Tool to run your NodeJS app container on it.

You know there are many steps to set this up, so you go through it with step by step exercises.

<details>
<summary>Exercise 1: Create IAM user</summary>
<br />

**Tasks:**

First of all, you need an IAM user with correct permissions to execute the tasks below.
- Create a new IAM user "your name" with "devops" user-group
- Give the "devops" group all needed permissions to execute the tasks below - with login and CLI credentials

Note: Do that using the AWS UI with Admin User

**Steps to solve the tasks:**\


</details>

******

<details>
<summary>Exercise 2: Configure AWS CLI</summary>
<br />

**Tasks:**

You want to use the AWS CLI for the following tasks. So, to be able to interact with the AWS account from the AWS Command Line tool you need to configure it correctly:
- Set credentials for that user for AWS CLI
- Configure correct region for your AWS CLI

**Steps to solve the tasks:**\


</details>

******

<details>
<summary>Exercise 3: Create VPC</summary>
<br />

**Tasks:**

You want to create the EC2 Instance in a dedicated VPC, instead of using the default one. So you:
- create a new VPC with 1 subnet and
- create a security group in the VPC that will allow you access on ssh port 22 and will allow browser access to your Node application (using the AWS CLI)

**Steps to solve the tasks:**\


</details>

******


<details>
<summary>Exercise 4: Create EC2 Instance</summary>
<br />

**Tasks:**

Once the VPC is created, you:
- Create an EC2 instance in that VPC
- with the security group you just created and ssh key file (using the AWS CLI)

**Steps to solve the tasks:**\


</details>

******


<details>
<summary>Exercise 5: SSH into the server and install Docker on it</summary>
<br />

**Tasks:**

Once the EC2 instance is created successfully, you want to prepare the server to run Docker containers. So you:
- ssh into the server and
- install Docker on it to run the dockerized application later

**Steps to solve the tasks:**\


</details>

******

### Set up Continuous Deployment

Now you don't want to deploy manually to the server all the time, because it's time-consuming and also sometimes you miss it, when changes are made and the new docker image is built by the pipeline. When you forget to check the pipeline, your team members need to write you and ask you to deploy the new version.

As a solution, you want to **automate** this thing to save you and your team members time and energy.

<details>
<summary>Exercise 6: Add docker-compose for deployment</summary>
<br />

**Tasks:**

First:
- add docker-compose to your NodeJS application

The reason is you want to have the whole configuration for starting the docker container in a file, in case you need to make changes to that, instead of a plain docker command with parameters. Also, in case you add a database later.

Use repository: https://gitlab.com/devops-bootcamp3/node-project

**Steps to solve the tasks:**\


</details>

******

<details>
<summary>Exercise 7: Add "deploy to EC2" step to your pipeline</summary>
<br />

**Tasks:**

- Complete the previous pipeline by adding a deployment step for your previous NodeJS project with docker-compose.

**Steps to solve the tasks:**\


</details>

******

<details>
<summary>Exercise 8: Configure access from browser (EC2 Security Group)</summary>
<br />

**Tasks:**

After executing the Jenkins pipeline successfully, the application is deployed, but you still can't access it from the browser. Again, you need to open the correct port on the server. For that you:
- Configure EC2 security group to access your application from browser (using AWS CLI)

**Steps to solve the tasks:**\


</details>

******

<details>
<summary>Exercise 9: Configure automatic triggering of multi-branch pipeline</summary>
<br />

**Tasks:**

Your team members are creating branches to add new features to the application or fix stuff, so you don't want to build and deploy all these half-done features or bug fixes. You want to build and deploy only the master branch. All other branches should only run tests. Add this logic to the Jenkinsfile.

- Add branch based logic to Jenkinsfile
- Add webhook to trigger pipeline automatically

**Steps to solve the tasks:**\


</details>

******
