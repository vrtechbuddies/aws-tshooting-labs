<h1 class='block'>If you lose the private key for an EBS-backed instance, you can regain access to your instance.
 <span class='badge badge-flexible'></span></h1>

You must stop the instance, detach its root volume and attach it to another instance as a data volume, modify the authorized_keys file with a new public key, move the volume back to the original instance, and restart the instance.

Note: This procedure is only supported for instances with EBS root volumes. If the root device is an instance store volume, you cannot use this procedure to regain access to your instance; you must have the private key to connect to the instance.

Steps for connecting to an EBS-backed instance with a different key pair

<b> Step 1: Create a new key pair </b>

Create a new key pair using either the Amazon EC2 console or a third-party tool. If you want to name your new key pair exactly the same as the lost private key, you must first delete the existing key pair.

<b> Step 2: Get information about the original instance and its root volume </b>

Make note of the following information because you'll need it to complete this procedure.

To get information about your original instance

    Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

    Choose Instances in the navigation pane, and then select the instance that you'd like to connect to. (We'll refer to this as the original instance.)

    On the Details tab, make note of the instance ID and AMI ID.

    On the Networking tab, make note of the Availability Zone.

    On the Storage tab, under Root device name, make note of the device name for the root volume (for example, /dev/xvda). Then, under Block devices, find this device name and make note of the volume ID (for example, vol-0a1234b5678c910de).

<b>Step 3: Stop the original instance</b>

Choose Instance state, Stop instance. If this option is disabled, either the instance is already stopped or its root device is an instance store volume.

<b>Step 4: Launch a temporary instance</b>

Choose Launch instances, and then use the launch wizard to launch a temporary instance with the following options:

    On the Choose an AMI page, select the same AMI that you used to launch the original instance. If this AMI is unavailable, you can create an AMI that you can use from the stopped instance. For more information, see Create an Amazon EBS-backed Linux AMI.

    On the Choose an Instance Type page, leave the default instance type that the wizard selects for you.

    On the Configure Instance Details page, specify the same Availability Zone as the original instance. If you're launching an instance in a VPC, select a subnet in this Availability Zone.

    On the Add Tags page, add the tag Name=Temporary to the instance to indicate that this is a temporary instance.

    On the Review page, choose Launch. Choose the key pair that you created in Step 1, then choose Launch Instances.

<b>Step 5: Detach the root volume from the original instance and attach it to the temporary instance </b>

    In the navigation pane, choose Volumes and select the root device volume for the original instance (you made note of its volume ID in a previous step).
    Choose Actions, Detach Volume, and then select Yes, Detach.
    Wait for the state of the volume to become available. (You might need to choose the Refresh icon.)

    With the volume still selected, choose Actions, and then select Attach Volume.
    Select the instance ID of the temporary instance, make note of the device name specified under Device (for example, /dev/sdf), and then choose Attach.



<b>Step 6: Add the new public key to authorized_keys on the original volume mounted to the temporary instance </b>

1. Connect to the temporary instance
2. From the temporary instance, mount the volume that you attached to the instance so that you can access its file system.

    a. Use the lsblk command to determine if the volume is partitioned.

    [ec2-user ~]$ lsblk
    NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    xvda    202:0    0   8G  0 disk
    └─xvda1 202:1    0   8G  0 part /
    xvdf    202:80   0   8G  0 disk
    └─xvdf1 202:81   0   8G  0 part

    b. Create a temporary directory to mount the volume.

    [ec2-user ~]$ sudo mkdir /mnt/tempvol

    c. Mount the volume (or partition) at the temporary mount point, using the volume name or device name that you identified earlier.

    [ec2-user ~]$ sudo mount -o nouuid /dev/xvdf1 /mnt/tempvol

3. From the temporary instance, use the following command to update authorized_keys on the mounted volume with the new public key from the authorized_keys for the temporary instance.

    [ec2-user ~]$ cp .ssh/authorized_keys /mnt/tempvol/home/ec2-user/.ssh/authorized_keys

Step 7: Unmount and detach the original volume from the temporary instance, and reattach it to the original instance
    a. Use umount command to unmount the original drive

    [ec2-user ~]$ sudo umount /mnt/tempvol

    b. Goto AWS Console, Select the volume and detach it from the old server

    c. With the same volume selected, attach it to the original server. Note: The device name should be xvda, as its a root volume.

Step 8: Connect to the original instance using the new key pair
Step 9: Clean up
