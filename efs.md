Building a Scalable Media Storage Solution with AWS EFS and EC2 Instances
Building a Scalable Media Storage Solution with AWS EFS and EC2 Instances

A Monk in Cloud's photo
A Monk in Cloud
·
Oct 10, 2024
·
4 min read

Business Scenario
A media production firm requires a centralised, scalable storage solution that can be accessed by multiple EC2 instances deployed across different Availability Zones (AZs). The company handles large volumes of video files and other media assets that need to be shared efficiently among several production teams. Their infrastructure demands high availability, seamless scalability, and top-tier performance, along with secure data transmission between servers.

To address these challenges, the company adopts AWS Elastic File System (EFS). EFS offers a fully managed, elastic, and highly available shared file system, which multiple EC2 instances can access simultaneously. This allows teams to collaborate seamlessly and work on shared assets without storage bottlenecks.

Solution:


1. Launching EC2 Instances in Multiple Availability Zones
Objective: Spin up two EC2 instances in different Availability Zones (AZs) to ensure high availability, redundancy, and performance. Both instances will share access to the same AWS EFS (Elastic File System).

Steps:

Create a Security Group: We’ll need to define a security group to keep things safe.
aws ec2 create-security-group --group-name StorageLabs --description "SG for EFS storage"

Add an Inbound SSH Rule: This allows you to log into your instances for some hands-on fun.
aws ec2 authorize-security-group-ingress --group-name StorageLabs --protocol tcp --port 22 --cidr 0.0.0.0/0

Launch EC2 in US-EAST-1A: Time to start the first instance in AZ 1A.

aws ec2 run-instances --image-id ami-0866a3c8686eaeeba --instance-type t2.micro --placement AvailabilityZone=us-east-1a --security-group-ids sg-00055e6ae2bc876ce

Launch EC2 in US-EAST-1B: Now, fire up the second instance in AZ 1B for redundancy and magic.

aws ec2 run-instances --image-id ami-0866a3c8686eaeeba --instance-type t2.micro --placement AvailabilityZone=us-east-1b --security-group-ids sg-00055e6ae2bc876ce

Now we have two EC2 buddies in different AZs, ready to share files like they’re best friends forever.

2. Creating and Configuring AWS EFS (Elastic File System)
Objective: Set up an AWS EFS that both EC2 instances can access, making it easy to share data between them.

Steps:

Create an EFS File System: Head to the AWS console, and with just a few clicks, boom—you’ve got an EFS. Associate it with the security group we created earlier. Keep all settings as default (it’s easy like Sunday morning).

Add NFS Protocol Rule: To make sure the EC2s and EFS play nicely, add an NFS rule to your security group.
aws ec2 authorize-security-group-ingress --group-id sg-00055e6ae2bc876ce --protocol tcp --port 2049 --source-group sg-00055e6ae2bc876ce

Mount the EFS: Jump into each EC2 instance, install the NFS client, and create a directory where we’ll mount the file system.

Install NFS Client:
sudo yum -y install nfs-utils

Create Mount Point:
mkdir ~/efs-mount-point

Mount the EFS:
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport your.efs.us-east-1.amazonaws.com:/ ~/efs-mount-point

Mount on Both Instances: Now, mount the EFS on both EC2s like a pro! Create a file in Instance A, and voilà—check out how it magically appears in Instance B. File sharing just became a breeze.

3. Enabling Encryption in Transit
Objective: Let’s step up the security game by enforcing encryption for all data transfers between the EC2 instances and the EFS.

Steps:

Create a File System Policy: Use the AWS console to set up a policy that enforces encryption in transit. This ensures all connections to the EFS are secured.

Unmount the File System: Before we proceed, unmount the EFS from your EC2 instance (and make sure you’re not in the mount directory when you do this—safety first).
sudo umount ~/efs-mount-point

Remount and What Happens?: After applying the encryption policy, when you try to remount using the standard NFS client, boom! Access denied! Why? Because the basic NFS client doesn’t support encryption in transit, which is now a must.
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport :/ ~/efs-mount-point

But don’t worry, there’s a solution.

Solution: Use EFS Utils: The secret sauce here is using Amazon’s EFS Utils, which supports the TLS encryption required by our new policy.

Install EFS Utils: Quickly install EFS Utils on your EC2s and you’re good to go!
sudo yum install -y amazon-efs-utils

Mount Again with Encryption: This time, mount your EFS with encryption enabled. Smooth sailing from here!
sudo mount -t efs -o tls :/ ~/efs-mount-point

Outcome:
Your teams can now access the same files across different AZs with practically zero latency.

The storage scales as your business grows, like magic.

Data is securely transferred with in-transit encryption, so you’re ticking all the boxes for security compliance.

With high availability, fault tolerance, and effortless collaboration, your production team is now unstoppable.

This project shows how AWS EFS provides a scalable, secure, and highly available storage solution that’s perfect for businesses needing seamless file sharing and collaboration across multiple AZs.
