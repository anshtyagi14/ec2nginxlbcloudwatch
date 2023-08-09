# Setting up an EC2 Ubuntu instance, installing NGINX, and setting up an Application Load Balancer

-----BLANK-----

## Requirements

-----BLANK-----

## Initial setup

### Step 1: Create an IAM Role

1. Go to the IAM console in the AWS Management Console.
2. In the navigation pane, select Roles and then Create Role.
3. For the trusted entity type, select AWS service and choose EC2.
4. Attach the CloudWatchFullAccessv2 policy.
5. (Optional) Add tags for easier management.
6. Name your role and create it.

### Step 2: Configure Security Groups

### a. ALB Security Group:

- Click ‘Create security group’.
- Name: alb-sg. Description: ALB Security Group. Select your VPC.
- Inbound rule: Allow HTTP from 0.0.0.0/0.

### b. EC2 Security Group:

- Click ‘Create security group’.
- Name: nginx-ec2-sg. Description: EC2 Security Group for Nginx. Select your VPC.
- Inbound rules:
	- Allow SSH from My IP.
	- Allow HTTP from the ALB's security group.

### Step 3: Launch Ubuntu EC2 Instance

1. Navigate to the EC2 dashboard.
2. Click Launch instance.
3. Select the Ubuntu Server AMI.
4. Configure instance details, ensuring you attach the IAM role created earlier.
5. Select or create a new key pair.
6. Attach the nginx-ec2-sg security group.
7. Launch the instance.

### Step 4: Set Up NGINX and CloudWatch on the EC2 Instance

SSH into your EC2 instance and execute the following:

```console
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install nginx -y
$ wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
$ sudo dpkg -i amazon-cloudwatch-agent.deb 
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

- Follow the CloudWatch Agent Configuration Manager prompt. Specify /var/log/nginx/access.log and /var/log/nginx/error.log for monitoring.
- Apply the CloudWatch configuration:

```console
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```

### Step 5: Set Up the Application Load Balancer

1. Go to the EC2 Dashboard and select Load Balancers.
2. Click Create Load Balancer and choose Application Load Balancer.
3. Configure the ALB settings and attach the alb-sg security group.
4. Configure the routing to point to a new target group. Do not add instances yet.
5. Complete the creation of the ALB.

### Step 6: Register EC2 Instance to the Target Group
1. Navigate to Target Groups under Load Balancers.
2. Select the target group created in the previous step.
3. Under the Targets tab, click Edit.
4. Add the EC2 instance. Ensure that you select the same availability zone as your EC2 instance and ALB.

### Step 7: Access NGINX via the Application Load Balancer

- Once the instance is registered and passes health checks, access the NGINX webpage through the Application Load Balancer DNS name.

### Step 8: Monitor with CloudWatch

1. Navigate to CloudWatch.
2. Under Metrics, view All Metrics.
3. Look under Logs to find and explore your log group metrics for NGINX.

Now, you've successfully set up an EC2 instance with NGINX, configured an Application Load Balancer to distribute incoming traffic, and set up monitoring using CloudWatch.
