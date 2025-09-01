Task 1: Create web data and sales data directories in /export/web_data and /export/sales_data on node 1. Persistently mount web data and sales data on node2 under /mnt/nfs_web_data and /mnt/nfs_sales_data

Task 2: Create a 2 GB gpt partition on node 1 and format the partition with xfs and mount the device persistently

Task 3: On Node1 Initialise one partition on sdb1 (1GB) and one on sdc (500MB) for use in LVM. Create a volume group called vg_group and add both physical volumes to it. Use the PE size of 16MB. Create two logical volumes, lv1 and lv2, in the vg_group volume group. Use 500MB for lv1 and 300MB for lv2. Display the details of the volume group and the logical volumes. Add another partition sdb2 of size 500MB to vg_group to increase the pool of allocatable space. Initialise the new partition prior to adding it to the volume group. Increase the size of lv1 to 600MB. Display the basic information for the physical volumes, volume group, and logical volume. Rename lv1 to lv0. Decrease the size of lv0 to 200MB. Remove both logical volumes. Display the summary for the volume groups, logical volumes, and physical volumes

Task 4:  
Uninitialize all three physical volumes by deleting the LVM structural information from them. Remove the partitions from the sdd disk and verify that all disks are now in their original raw state

Task 5:  
Create a new logical volume (LV-A) with a size of 30 extents that belongs to the volume group VG-A (with a PE size of 32M). After creating the volume, configure the server to mount it persistently on /mnt/share_drive/

Task 6:  
Create 1 swap of 400MB on sdc. Create another swap area in a 400MB logical volume called swapvol in vgfs of 1GB. Add their entries to the /etc/fstab file for persistence. Use the UUID and priority 1 for the partition swap and the device file and priority 2 for the logical volume swap. Activate them and use appropriate tools to validate the activation

Task 7:  
Create 2x500MB partitions on the /dev/sdb disk, initialise them separately with the Ext4 and XFS file system types

- create mount points called /ext4fs and /xfs1
    
- attach them to the directory structure, verify their availability and usage
    
- mount them persistently using their labels
    

Task 8:  
Create a volume group called vgfs comprised of a 1.6GB physical volume created in a partition on the /dev/sdc disk. The PE size for the volume group should be set at 160MB. Create 2 logical volumes called ext4vol and xfsvol of size 800MB each and initialise them with the Ext4 and XFS file system types. Ensure that both file systems are persistently defined using their logical volume device filenames. Create mount points /ext4fs2 and /xfsfs2, mount the file systems, and verify their availability and usage

Task 9:  
Grow the size of the vgfs volume group that was created above by adding another disk sdb to it. Extend the ext4vol logical volume along with the file system it contains by 400MB

Task 10:  
Configure a direct map to automount the NFS share /common that is available from server2. Install the relevant software, create a local mount point /autodir, and set up AutoFS maps to support the automatic mounting. Note that /common is already mounted on the /local mount point on server1 via fstab. Ensure there is no conflict in configuration or functionality between the 2

Task 11:  
On node1 (NFS server), create a user account called user30 with UID 3000. Add the /home directory to the list of NFS shares so that it becomes available for remote mount. On node2 (NFS client), create a user account called user30 with UID 3000, base directory /nfshome, and no user home directory. establish an indirect map to automount the remote home directory of user30 under /nfshome. Observe that the home directory of user30 is automounted under /nfshome when you sign in as user30

Task 12: Use NFS to export home directories for all users (user100, user200, and user300) on node1 so that their home directories become available automatically under /home1 when they log on to node2. Create user100, user200, and user300. Ensure the UUIDs on both servers for these users are the same.

Task 13: On node1 create a disk partition of size 1GiB on sda1 disk and format it with Ext4 file system structures. Assign label stdlabel to the file system. Mount the file system on /mnt/stdfs1 persistently using the label. Create file stdfile1 in the mount point.

Task 14: On node2 create a logical volume called lv1 of size equal to 10 LEs in vg1 volume group (create vg1 with PE size 8MB in a partition on the secondary disk). Initialize the logical volume with XFS type and mount it on /mnt/lvfs1. Create a file called lv1file1 in the mount point. Set the file system to automatically mount at each system reboot.

Task 15: On node2 extend the file system in the logical volume lv1 by 64MB without unmounting it and without losing any data. Confirm the new size for the logical volume and the file system.

Task 16: On Node1 Create a logical volume of name "wshare" from a volume group name "wgroup" physical extends of 16M and logical volume should have size of 50extents. Format with ext4 filesystem and permanently mounted to /mnt/wshare

Task 17: On Node2 Set Up a Volume Group with 7GB Disk Space, create a 4GB Logical Volume, and Expand It by 1GB, Shrink the LVM by 3GB, Ensuring No Data Loss.