To install the CloudWatch Agent on your EC2 instance and configure it to use SSM parameters to fetch instance values, follow these steps:

1. Install CloudWatch Agent
Log into your EC2 instance and run the following commands to install the CloudWatch Agent:

bash
Copy code
# Update package lists
sudo yum update -y  # For Amazon Linux 2 / CentOS
sudo apt-get update -y  # For Ubuntu

# Install CloudWatch Agent
sudo yum install amazon-cloudwatch-agent -y  # For Amazon Linux 2 / CentOS
sudo apt-get install amazon-cloudwatch-agent -y  # For Ubuntu
2. Create an IAM Role for EC2
Make sure your EC2 instance has an IAM role attached that has the required permissions to read SSM parameters and push metrics to CloudWatch. If you don’t have this role:

Go to the IAM console.
Create a new role with the following AWS managed policies:
AmazonSSMReadOnlyAccess
CloudWatchAgentServerPolicy
Attach the role to your EC2 instance.
3. Create SSM Parameters (Optional)
You can create parameters in SSM Parameter Store to hold specific configuration values that CloudWatch Agent will fetch.

Navigate to Systems Manager in the AWS console.
Go to Parameter Store and click Create parameter.
Enter the name, type (e.g., String, SecureString), and value.
For example, create a parameter to hold your CloudWatchAgentConfig:

Name: /EC2/CloudWatch/AgentConfig
Type: String
Value: The CloudWatch agent configuration in JSON format (more on this below).
4. Create CloudWatch Agent Configuration
You can create a JSON configuration file for the CloudWatch agent to specify which metrics and logs to collect. Here's a sample configuration file:

json
Copy code
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
Store this configuration in an SSM parameter like /EC2/CloudWatch/AgentConfig.

5. Configure the CloudWatch Agent to Fetch SSM Parameters
To configure the CloudWatch agent to use the configuration from SSM Parameter Store:

On your EC2 instance, run:

bash
Copy code
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config \
-m ec2 \
-c ssm:/EC2/CloudWatch/AgentConfig \
-s
This command tells the CloudWatch agent to fetch the configuration from the SSM parameter /EC2/CloudWatch/AgentConfig and start the agent.

6. Verify Installation
To check if the CloudWatch Agent is running:

bash
Copy code
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
You should see the agent's status and whether it is collecting metrics and logs.

This setup will allow your CloudWatch Agent to use SSM parameters to fetch configurations dynamically and report metrics and logs to CloudWatch. Let me know if you need more details on any of the steps!






