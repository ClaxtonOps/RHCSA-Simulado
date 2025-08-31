- Install a RHEL 9 virtual machine that meets the following requirements:
    
    - 2 GB of RAM
        
    - 20 GB of disk space using default partitioning
        
    - One additional 20-GB disk that does not have any partitions installed
        
    - Server with GUI installation pattern
        
- Create user **student** with password **password**, and user **root** with password **password**.
    
- Configure your system to automatically mount the ISO of the installation disk on the directory **/repo**. Configure your system to remove this loop-mounted ISO as the only repository that is used for installation. Do _not_ register your system with **subscription-manager**, and remove all references to external repositories that may already exist.
```
lsblk
sr0            11:0    1 11.9G  0 rom 
mkdir -p /repo
mount -o loop /dev/sr0 /repo
vim /etc/yum.repos.d/local.repo

[local-baseos]
name=Local BaseOS
baseurl=file:///repo/BaseOS
enabled=1
gpgcheck=0

[local-appstream]
name=Local AppStream
baseurl=file:///repo/AppStream
enabled=1
gpgcheck=0
```




- Create a 500-MiB partition on your second hard disk, and format it with the Ext4 file system. Mount it persistently on the directory /mydata, using the label **mydata**.
```
fdisk /dev/sdb1
500MiB
mkfs.ext4 /dev/sdb1
e2label /dev/sdb1 mydata
blkid /dev/sdb1

/dev/sdb1: LABEL="mydata" UUID="21a3666d-d28b-4641-b3bf-57345bfbcc77" TYPE="ext4"

echo "LABEL="mydata" /mydata ext4 defaults 0 0" >> /etc/fstab
```

- Set default values for new users. A user should get a warning three days before expiration of the current password. Also, new passwords should have a maximum lifetime of 120 days.
```
PASS_MAX_DAYS   120
PASS_WARN_AGE   3
```
    
- Create users **lori** and **laura** and make them members of the secondary group sales. Ensure that user lori uses UID 2000 and user laura uses UID 2001.
```
useradd --uid 2000 lori
useradd --uid 2001 laura
groupadd sales
usermod -aG sales lori
usermod -aG sales laura


```

- Create shared group directories **/groups/sales** and **/groups/data**, and make sure the groups meet the following requirements:
    
    - Members of the group sales have full access to their directory.
    
    - Members of the group data have full access to their directory.
        
    - Others has no access to any of the directories.
        
    - Alex is general manager, so user alex has read access to all files in both directories and has permissions to delete all files that are created in both directories.
```
mkdir -p /groups/sales; mkdir -p /groups/data
chown :sales /groups/sales
chmod 2770 /groups/sales

chown :data /groups/data
chmod 2770 /groups/data

setfacl -m u:alex:rwx /groups/sales
setfacl -m u:alex:rwx /groups/data

setfacl -d -m u:alex:rwx /groups/sales
setfacl -d -m u:alex:rwx /groups/data
```
        
- Create a 1-GiB swap partition and mount it persistently.
```
fdisk /dev/sdc
mkswap /dev/sdc1
swapon /dev/sdc1
echo "/dev/sdc1 none swap defaults 0 0" >> /etc/fstab
```
    
- Find all files that have the SUID permission set, and write the result to the file /root/suidfiles.
    
- Create a 1-GiB LVM volume group. In this volume group, create a 512-MiB swap volume and mount it persistently.
```
fdisk /dev/sdd
sdd                 8:48   0   10G  0 disk 
└─sdd1              8:49   0    1G  0 part 
vgcreate 1G vgswap /dev/sdd1
lvcreate -L 500MiB -n lvswap vgswap
mkswap /dev/vgswap/lvswap
swapon /dev/vgswap/lvswap
echo "/dev/vgswap/lvswap none swap defaults 0 0" >> /etc/fstab
```



- Add a 10-GiB disk to your virtual machine. On this disk, create a Stratis pool and volume. Use the name **stratisvol** for the volume, and mount it persistently on the directory /stratis.
    
- Install an HTTP web server and configure it to listen on port 8080.
```
vim /etc/httpd/conf/httpd.conf
Change Port 80 for Port 8080
firewall-cmd --add-port=8080/tcp --permanent 
firewall-cmd --reload
systemctl restart httpd
curl localhost:8080
```

- Create a configuration that allows user laura to run all administrative commands using **sudo**.
```
echo "laura	ALL=(ALL) 	ALL" >> /etc/sudoers.d/laura
chmod 440 /etc/sudoers.d/laura
```
	
- Create a directory with the name **/users** and ensure it contains the subdirectories **linda** and **anna**. Export this directory by using an NFS server.
```
mkdir -p /users/linda; mkdir -p /users/anna
dnf install -y nfs-utils
echo "/users *(rw,no_root_squash)" >> /etc/exports
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload
systemctl enable --now nfs-server.service
exportfs -rav
```
    
- Create users **linda** and **anna** and set their home directories to /home/users/linda and /home/users/anna. Make sure that while these users access their home directory, autofs is used to mount the NFS shares /users/linda and /users/anna from the same server.
```
useradd -d /home/users/linda linda
useradd -d /home/users/anna anna

```