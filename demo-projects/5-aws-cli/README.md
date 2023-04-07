## Demo Project - AWS CLI

### Topics of the Demo Project
Interacting with AWS CLI

### Technologies Used
- AWS
- Linux

### Project Description
- Install and configure AWS CLI tool to connect to our AWS account
- Create SSH key pair
- Create EC2 Instance using the AWS CLI with all necessary configurations like Security Group
- Create IAM resources like User, Group, Policy using the AWS CLI
- List and browse AWS resources using the AWS CLI

#### Steps to Install and configure AWS CLI
**Step 1:** Install AWS CLI
```sh
brew update
brew install awscli
```

**Step 2:** Configure AWS CLI
```sh
aws configure
  AWS Access Key ID [None]: <access-key-id> # downloaded when creating the admin user
  AWS Secret Access Key [None]: <secret-access-key> # downloaded when creating the admin user
  Default region name [None]: eu-central-1 # Frankfurt
  Default output format [None]: json
```

#### Steps to Create SSH key pair
```sh
aws ec2 create-key-pair \
  --key-name MyKpCli \
  --query 'KeyMaterial' \
  --output text > MyKpCli.pem
```
The first two lines would create a key pair and return a JSON object describing it. The private key is stored in the "KeyMaterial" attribute, so we can directly query the returned JSON object to just output the value of the "KeyMaterial" attribute and redirect it into a file called "MyKpCli.pem".

#### Steps to Create EC2 Instance using the AWS CLI
**Step 1:** Create a security group
```sh
# get the VPC id
aws ec2 describe-vpcs --query "Vpcs[].VpcId" --output text # => vpc-04acd8f40d2f4b8e9

# create a security group
aws ec2 create-security-group \
  --group-name devops-bootcamp-sg \
  --description "Devops Bootcamp Security Group" \
  --vpc-id vpc-04acd8f40d2f4b8e9

# copy the id of the security group -> sg-058bdb4fd0f2399ea
```

**Step 2:** Create a Firewall Rule
To open port 22 for ssh-ing into the EC2 instance from my IP address, I add a firewall rule to the new security group:
```sh
aws ec2 authorize-security-group-ingress \
  --group-id sg-058bdb4fd0f2399ea \
  --protocol tcp \
  --port 22 \
  --cidr 31.10.151.111/32
  ```

**Step 3:** Get a subnet id\
We just take the subnet id of the first availability zone:
```sh
aws ec2 describe-subnets \
  --filter "Name=availability-zone,Values=eu-central-1a" \
  --query "Subnets[].SubnetId" \
  --output text
# => subnet-07b8c6436782bd3b8
```

**Step 4:** Get the Amazon image id\
Amazon image ids can be found by `aws ec2 describe-images` but it is probably faster to open the management web console, start launching a new instance, select the required image and copy the ID starting with 'ami-'. We choose the "Amazon Linux 2023 AMI 2023.0.20230329.0 x86_64 HVM kernel-6.1" image with the id "ami-0fa03365cde71e0ab".

**Step 5:** Create the EC2 instance\
Now we are ready to create a new EC2 instance:
```sh
aws ec2 run-instances \
  --image-id ami-0fa03365cde71e0ab \
  --count 1 \
  --instance-type t2.micro \
  --key-name MyKpCli \
  --security-group-ids sg-058bdb4fd0f2399ea \
  --subnet-id subnet-07b8c6436782bd3b8
```

**Step 6:** SSH into the new EC2 instance (optional)
```sh
# get the public ip address of the new ec2 instance
aws ec2 describe-instances \
  --filters "Name=key-name,Values=MyKpCli" \
  --query "Reservations[].Instances[].PublicIpAddress" \
  --output text 
# => 18.159.35.168

# restrict read-permissions of the saved .pem file
chmod 400 MyKpCli.pem

# ssh into the ec2 instance
ssh -i MyKpCli.pem ec2-user@18.159.35.168
```

#### Steps to Create IAM resources like User, Group, Policy using the AWS CLI
**Step 1:** 


#### Steps to List and browse AWS resources using the AWS CLI
**Step 1:** 