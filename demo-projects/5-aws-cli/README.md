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

#### Steps to install and configure AWS CLI
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

#### Steps to create an SSH key pair
```sh
aws ec2 create-key-pair \
  --key-name MyKpCli \
  --query 'KeyMaterial' \
  --output text > MyKpCli.pem
```
The first two lines would create a key pair and return a JSON object describing it. The private key is stored in the "KeyMaterial" attribute, so we can directly query the returned JSON object to just output the value of the "KeyMaterial" attribute and redirect it into a file called "MyKpCli.pem".

#### Steps to create an EC2 instance
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

#### Steps to create IAM resources like User, Group, Policy
```sh
# create a user group
aws iam create-group --group-name MyGroupCli

# create a user
aws iam create-user --user-name MyUserCli

# add the created user to the group
aws iam add-user-to-group --user-name MyUserCli --group-name MyGroupCli

# get the ARN (Amazon Resource Name) of the AmazonEC2FullAccess policy
aws iam list-policies --query 'Policies[?PolicyName==`AmazonEC2FullAccess`].Arn' --output text
# => arn:aws:iam::aws:policy/AmazonEC2FullAccess

# attach that policy to the new group
aws iam attach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess

# create a password for the new user to login on the UI web console
aws iam create-login-profile \
  --user-name MyUserCli \
  --password InitialPassword123 \
  --password-reset-required
```

In order to login as the new 'MyUserCli' user we need the account ID. This number is part of the 'MyUserCli' user's ARN. `aws iam get-user --user-name MyUserCli --query "User.Arn" --output text` outputs `arn:aws:iam::369076538622:user/MyUserCli`. The required account ID is the number 369076538622 within that ARN.

To change the initial password on first login, the user needs the permission to do that. This permission is not part of the 'AmazonEC2FullAccess' policy. We also have to add the 'IAMUserChangePassword' policy:
```sh
aws iam list-policies --query 'Policies[?PolicyName==`IAMUserChangePassword`].Arn' --output text
# => arn:aws:iam::aws:policy/IAMUserChangePassword

aws iam attach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/IAMUserChangePassword
```

Or we could create our own policy containing the required permission and assign that policy to the user group.

**Create a policy:**\
First create a JSON file with the following content (copied from the 'IAMUserChangePassword' policy looked up in the management web console under "IAM" > "Access management" > "Policies" > "IAMUserChangePassword", restricted to the user of our account only):
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:ChangePassword"
            ],
            "Resource": [
                "arn:aws:iam::369076538622:user/${aws:username}"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:GetAccountPasswordPolicy"
            ],
            "Resource": "*"
        }
    ]
}
```
Save the file as 'changePwdPolicy.json'. Now create the policy:
```sh
aws iam create-policy \
  --policy-name changePwd \
  --policy-document file://changePwdPolicy.json
```

Copy the policy ARN from the output of the last command and attach it to the user-group as before:
```sh
aws iam attach-group-policy \
  --group-name MyGroupCli \
  --policy-arn arn:aws:iam::369076538622:policy/changePwd
```

Now we can login with the account ID '369076538622' as user 'MyUserCli' and the initial password 'InitialPassword123'. And we can change the initial password as required.

**Create access key and access secret key for console login:**
Let's also create an access key id and a secret access key for the new user, which he can use for login from the command line (e.g. to execute AWS CLI commands). The following command creates both an access key id and a secret access key and stores them in a local file called 'MyUserCli_AccessKey.json':
```sh
aws iam create-access-key \
  --user-name MyUserCli \
  --query "AccessKey.{ACCESS_KEY_ID:AccessKeyId, SECRET_ACCESS_KEY:SecretAccessKey}" > MyUserCli_AccessKey.json
```

**Switch user for executing AWS CLI commands:**\
If you want to switch the user for executing AWS CLI commands, you can
- execute `aws configure` again (which also overwrites the region and output format)
- execute `aws configure set aws-access-key-id ...` and `aws configure set aws-secret_access-key ...` only
- just temporarily set the environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` (not changing the default user set by `aws configure`)

#### AWS CLI commands to list and browse AWS resources
To get a list of all ec2 'describe' sub-commands execute
```sh
aws ec2 help | grep describe
# get more information on a specific sub-command
aws ec2 describe-instances help
```

For the iam service the listing/browsing sub-commands start with 'list-' or 'get-':
```sh
aws iam help | grep list
aws iam list-policies help

aws iam help | grep get
aws iam get-user help
```

When executing aws sub-commands that display information on components (e.g. describe sub-commands), we can add a 
- `--filters` option to pick only certain components (name-values, where name is any attribute name), and a
- `--query` option to pick specific attributes of the components.

Note that the attribute name used in the `--filtering` option is written in lower case with hyphens instead of camel-case, so InstanceType is written as instance-type.

Examples:
```sh
aws ec2 describe-instances \
  --filters "Name=instance-type,Values=t2.micro" \
  --query "Reservations[].Instances[].InstanceId"

# query multiple attributes
aws ec2 describe-instances \
  --filters "Name=instance-type,Values=t2.micro" \
  --query "Reservations[].Instances[].{ID:InstanceId, ImageId:ImageId}"

# just using query (with pattern matching)
aws ec2 describe-instances \
  --query 'Reservations[].Instances[?InstanceType==`t2.micro`].InstanceId'

# filtering multiple values
aws ec2 describe-instances \
  --filters "Name=instance-id,Values=ami-x0123456,ami-y0123456,ami-z0123456" \
  --query "Reservations[].Instances[].InstanceId"

# filtering for certain tags
aws ec2 describe-instances \
  --filters "Name=tag:Type,Values=web-server-with-docker" \
  --query "Reservations[].Instances[].InstanceId"
```

More examples of 'describe' and 'get' sub-commands can be found in the above steps of creating an EC2 instance and a user, group, policy and credentials.

#### Optional: Delete all the components and resources created above
```sh
aws iam list-access-keys --user-name MyUserCli --query "AccessKeyMetadata[].AccessKeyId" --output text
# => AKIAVL3VNBT7H3WTEBPQ
aws iam delete-access-key --user-name MyUserCli --access-key-id AKIAVL3VNBT7H3WTEBPQ

aws iam list-policies --query 'Policies[?PolicyName==`changePwd`].Arn' --output text
# => arn:aws:iam::369076538622:policy/changePwd
aws iam delete-policy --policy-arn arn:aws:iam::369076538622:policy/changePwd

aws iam delete-login-profile --user-name MyUserCli

aws iam remove-user-from-group --user-name MyUserCli --group-name MyGroupCli
aws iam delete-user --user-name MyUserCli

aws iam detach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/IAMUserChangePassword
aws iam detach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
aws iam delete-group --group-name MyGroupCli

# ---

aws ec2 describe-instances --filter "Name=key-name,Values=MyKpCli" --query "Reservations[].Instances[].InstanceId" --output text
# => i-076b3a9afd65e9c42
aws ec2 terminate-instances --instance-ids i-076b3a9afd65e9c42

aws ec2 describe-security-groups --filters "Name=group-name,Values=devops-bootcamp-sg" --query "SecurityGroups[].GroupId" --output text
# => sg-058bdb4fd0f2399ea
aws ec2 delete-security-group --group-id sg-058bdb4fd0f2399ea

aws ec2 describe-key-pairs --filters "Name=key-name,Values=MyKpCli" --query "KeyPairs[].KeyPairId" --output text
# => key-0fc2afb29f74b70b9
aws ec2 delete-key-pair --key-pair-id key-0fc2afb29f74b70b9
```
