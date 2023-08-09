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
5. Name your role and create it.

### Step 2: Configure Security Groups

### a. ALB Security Group:

1. Go to AWS Management Console and navigate to the EC2 dashboard.
2. On the left sidebar, under ‘Network & Security’, click ‘Security Groups’.
3. Click ‘Create security group’.
4. Name it 'alb-sg' and provide a description as 'ALB Security Group'.
5. In the ‘Inbound rules’ tab:
	- Click ‘Add rule’.
	- Choose ‘HTTP’ for Type and set the source to ‘0.0.0.0/0’. This means the ALB will accept HTTP traffic from anywhere.
6. Click ‘Create security group’.

### b. EC2 Security Group:

1. Again, on the left sidebar under ‘Network & Security’, click ‘Security Groups’.
2. Click ‘Create security group’.
3. Name it 'nginx-ec2-sg' and provide a description as 'EC2 Security Group for Nginx'.
4. In the ‘Inbound rules’ tab:
	- Click ‘Add rule’.
	- Choose ‘SSH’ for Type and source as ‘My IP’ to allow only your IP to SSH into the instance.
	- Add another rule. Choose ‘HTTP’ for Type and for the source, specify the security group of the ALB. This means traffic from the ALB will be allowed.
5. Click ‘Create security group’.

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
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install nginx
$ wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
$ sudo dpkg -i amazon-cloudwatch-agent.deb 
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

- Follow the CloudWatch Agent Configuration Manager prompt. Specify /var/log/nginx/access.log and /var/log/nginx/error.log for monitoring.
- Apply the CloudWatch configuration:

```console
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```

```console
================================================================
= Welcome to the Amazon CloudWatch Agent Configuration Manager =
=                                                              =
= CloudWatch Agent allows you to collect metrics and logs from =
= your host and send them to CloudWatch. Additional CloudWatch =
= charges may apply.                                           =
================================================================
```

```console
On which OS are you planning to use the agent?
1. linux
2. windows
3. darwin
default choice: [1]:
1
```

```console
Trying to fetch the default region based on ec2 metadata...
Are you using EC2 or On-Premises hosts?
1. EC2
2. On-Premises
default choice: [1]:
1
```

```console
Which user are you planning to run the agent?
1. root
2. cwagent
3. others
default choice: [1]:
1
```

```console
Do you want to turn on StatsD daemon?
1. yes
2. no
default choice: [1]:
2
```

```console
Do you want to monitor metrics from CollectD? WARNING: CollectD must be installed or the Agent will fail to start
1. yes
2. no
default choice: [1]:
2
```

```console
Do you want to monitor any host metrics? e.g. CPU, memory, etc.
1. yes
2. no
default choice: [1]:
2
```

```console
Do you have any existing CloudWatch Log Agent (http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AgentReference.html) configuration file to import for migration?
1. yes
2. no
default choice: [2]:
2
```

```console
Do you want to monitor any log files?
1. yes
2. no
default choice: [1]:
1
```

```console
Log file path:
/var/log/nginx/access.log
```

```console
Log group name:
default choice: [access.log]
```

```console
Log stream name:
default choice: [{instance_id}]
```

```console
Log Group Retention in days
1. -1
2. 1
3. 3
...
default choice: [1]:
1
```

```console
Do you want to specify any additional log files to monitor?
1. yes
2. no
default choice: [1]:
1
```

```console
Log file path:
/var/log/nginx/error.log
```

```console
Log group name:
default choice: [error.log]
```

```console
Log stream name:
default choice: [{instance_id}]
```

```console
Log Group Retention in days
1. -1
2. 1
3. 3
...
default choice: [1]:
```

```console
Do you want to specify any additional log files to monitor?
1. yes
2. no
default choice: [1]:
2
```

```console
Saved config file to /opt/aws/amazon-cloudwatch-agent/bin/config.json successfully.
Current config as follows:
{
        "agent": {
                "run_as_user": "root"
        },
        "logs": {
                "logs_collected": {
                        "files": {
                                "collect_list": [
                                        {
                                                "file_path": "/var/log/nginx/access.log",
                                                "log_group_name": "access.log",
                                                "log_stream_name": "{instance_id}",
                                                "retention_in_days": -1
                                        },
                                        {
                                                "file_path": "/var/log/nginx/error.log",
                                                "log_group_name": "error.log",
                                                "log_stream_name": "{instance_id}",
                                                "retention_in_days": -1
                                        }
                                ]
                        }
                }
        }
}

Please check the above content of the config.
The config file is also located at /opt/aws/amazon-cloudwatch-agent/bin/config.json.
Edit it manually if needed.
```

```console
Do you want to store the config in the SSM parameter store?
1. yes
2. no
default choice: [1]:
2
Program exits now.
```

### Step 5: Create a Target Group

1. Navigate to the EC2 dashboard in the AWS Console.
2. On the left sidebar, under ‘Load Balancing’, click on ‘Target Groups’.
3. Click ‘Create target group’.
4. Give it a name, and ensure the target type is set to ‘Instances’. Set the protocol to ‘HTTP’ and the port to 80.
5. Set the health check to ‘HTTP’ on the default port and click ‘Next’
6. Do not add instances yet and click ‘Create target group’.

### Step 6: Set up an Application Load Balancer (ALB)

1. On the EC2 dashboard, select ‘Load Balancers’ from the sidebar.
2. Click on ‘Create Load Balancer’.
3. Choose ‘Application Load Balancer’ and click ‘Create’.
4. In the ‘Network mapping’ step, select at least two Availability Zones and one must be the Availability Zones in which the EC2 instance is present.
5. Attach the 'alb-sg' security group.
6. In the ‘Listeners and routing’ step, select the target group we created earlier.
7. Review and create the load balancer.

### Step 7: Register EC2 Instance to the Target Group
1. Wait until the ALB provisioning completes and becomes active.
2. Again, on the left sidebar, under ‘Load Balancing’, click on ‘Target Groups’.
3. Navigate to Target Groups under Load Balancers.
4. Select the target group created in the previous step.
5. Under the Targets tab, click Register targets.
6. Select the earlier created EC2 instance, click on 'Include as pending below' and click on 'Register pending targets'

### Step 7: Access NGINX via the Application Load Balancer

- Once the instance is registered and passes health checks, access the NGINX webpage through the Application Load Balancer DNS name.

### Step 8: Monitor with CloudWatch

1. Navigate to CloudWatch.
2. Under Metrics, view All Metrics.
3. Look under Logs to find and explore your log group metrics for NGINX.

Now, you've successfully set up an EC2 instance with NGINX, configured an Application Load Balancer to distribute incoming traffic, and set up monitoring using CloudWatch.
