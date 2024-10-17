**CloudWatch Agent Setup on EC2 with SSM Parameters**


This document outlines the steps to install the Amazon CloudWatch Agent on an EC2 instance and configure it to fetch values from AWS Systems Manager (SSM) Parameter Store.
**Table of Contents**
•	Prerequisites
•	Step 1: Install CloudWatch Agent
•	Step 2: Create an IAM Role for EC2
•	Step 3: Create SSM Parameters
•	Step 4: Create CloudWatch Agent Configuration
•	Step 5: Configure CloudWatch Agent to Fetch SSM Parameters
•	Step 6: Verify Installation
________________________________________
**Prerequisites**
•	An EC2 instance running Amazon Linux 2, CentOS, or Ubuntu.
•	AWS CLI installed on the instance (optional but recommended).
•	Appropriate IAM permissions to create roles and access SSM Parameter Store.

**Step 1: Install CloudWatch Agent**

**Amazon Linux 2 / CentOS:**

sudo yum update -y
sudo yum install amazon-cloudwatch-agent -y

**Ubuntu:**

sudo apt-get update -y
sudo apt-get install amazon-cloudwatch-agent -y

**Step 2: Create an IAM Role for EC2**
1.	Go to the IAM console.
2.	Create a new role with the following AWS managed policies:
o	AmazonSSMReadOnlyAccess
o	CloudWatchAgentServerPolicy
3.	Attach the role to your EC2 instance.

**Step 3: Create SSM Parameters**
1.	Navigate to AWS Systems Manager in the AWS console.
2.	Go to Parameter Store and click Create parameter.
3.	Enter the following details:
o	Name: /EC2/CloudWatch/AgentConfig
o	Type: String
o	Value: Provide the CloudWatch agent configuration in JSON format (see below).

**Step 4: Create CloudWatch Agent Configuration**
Create a CloudWatch agent configuration to specify which metrics and logs to collect. Use the following sample JSON configuration:

{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/messages",
            "log_group_name": "EC2-LogGroup",
            "log_stream_name": "{instance_id}/messages"
          }
        ]
      }
    }
  },
  "metrics": {
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}",
      "InstanceType": "${aws:InstanceType}"
    },
    "aggregation_dimensions": [["InstanceId"]],
    "metrics_collected": {
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      },
      "cpu": {
        "measurement": ["cpu_usage_idle"],
        "metrics_collection_interval": 60
      }
    }
  }
}

Store this JSON file in SSM Parameter Store under the name /EC2/CloudWatch/AgentConfig.

**Step 5: Configure CloudWatch Agent to Fetch SSM Parameters**
On your EC2 instance, run the following command to fetch the configuration from SSM and start the CloudWatch Agent:

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c ssm:/EC2/CloudWatch/AgentConfig \
  -s

This command tells the CloudWatch agent to fetch the configuration from the SSM parameter /EC2/CloudWatch/AgentConfig and apply it.

**Step 6: Verify Installation**
Check the status of the CloudWatch Agent with the following command:

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status

This will display the status of the agent, including whether it is actively collecting logs and metrics.
________________________________________
By following these steps, your CloudWatch Agent will be configured to use SSM Parameter Store to dynamically fetch configuration settings for your EC2 instance.

