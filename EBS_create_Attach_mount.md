**How to create, attach and mount EBS volumes to a running Linux EC2 instance**? 

These changes can done on the fly without stopping the EC2 instance.

**Commands Used**: 

df -h

**Lists all the block devices in the Linux Machine:**

lsblk  

**Check if there is any file system on new EBS Volume:**

file -s /dev/xvdf
(If you see "Data", meaning you need to setup file system for this block device.)

**Create a file system on volume to mount it to EC2:**

mkfs -t xfs /dev/xvdf

**Create new directory:**

mkdir -p /apps/my-data/apps/volume/new-volume
cd /apps/my-data/

**Mount volume to EC2 Instance:**

mount /dev/xvdf /apps/my-data/apps/volume/new-volume 
