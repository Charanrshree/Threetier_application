Increase EBS Volume Size in AWS
===========================================================
Step 1: Take Snapshot of EBS Volume (to be Safe).
Step 2: Increase EBS Volume Size in AWS Console.
Step 3: Extend a Linux file system after resizing a volume.


Command to Extend the file system:
===========================================================
df -hT

**Check whether the volume has a partition**:

sudo lsblk

**Extend the partition**:

sudo growpart /dev/xvda 1

**Extend the file system on /:**    this path of disk her / means root and below filesystem and ext4 use one command

[XFS file system]: sudo xfs_growfs -d /
[Ext4 file system]: sudo resize2fs /dev/xvda1
