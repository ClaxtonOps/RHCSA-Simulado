Task1: 
	1 Find the string "Listen" in /etc/httpd/conf/httpd.conf and save the output to /root/web.txt
	1.2 Create a gzip-compressed tar archive of /etc named etc_vault.tar.gz in /vaults directory.
		Use man pages for gzip option
```
cat /etc/httpd/conf/httpd.conf | grep Listen > /root/web.txt
man tar | grep gzip
mkdir /vaults
tar -czvf /vaults/etc_vault.tar.gz /etc
```

Task2:
	2.1 In shorts directory:
	- Create a file file_a
	- Create a soft link file_b pointing to file_a
	- create hard link file_c point to file_a
	- verify all links works
```
mkdir -p /shorts
cd /shorts
touch file_a

```

Task 3:
	3.1 Find files in /usr that are greater than 3MB but < 10MB and copy them to /bigfiles directory
	3.2 Find files in /etc modified more than 120 days ago and copy them to /var/tmp/twenty
	3.3 Find all files owned by user hadoga and copy them to /root/h-files
	 3.4 find a file named "httpd.conf" and save the absolute paths to root/httpd-path.txt
```
3.1:
mkdir -p /bigfiles
find /usr -type f -size +3M -size -10M -exec cp {} /bigfiles \;

3.2:
mkdir -p /var/tmp/twenty
find /etc -type f -mtime +120 -exec cp -a {} /var/tmp/twenty/ \;

3.3:
mkdir -p /root/hfiles
find / -type f -user hadoga -exec cp -a {} /root/hfiles \; 

3.4
find / -name "httpd.conf" >> /root/httpd.path.txt 
```

Task 4:
From a node1, ssh into node2 as user hadoga and:
Copy the contents of /etc/fstab to /var/tmp
Set the file ownership to root
ensure no execute permission for anione
```
cp /etc/fstab /var/tmp
su root
chgrp root /var/tmp/fstab
chmod 776 /var/tmp/fstab
```

Task 5:
Create a shell script that:
Finds files in /usr sized >30kb but <50kb
outputs results to /root/sized_files.txt
```
#!/bin/bash
echo "------------------- Shellscript for size +30 but -50 Kbs -----------------"

if [ -f /root/sized_files.txt ]; then
	echo "arquivo . txt ja criado."
	exit 0
else
	echo " . txt criado com a lista de arquivos 30-50kb"
	find /usr -size +30k -size -50k >> /root/sized_files.txt
	exit 0
fi

exit 0
```

```
chmod +x size.sh
./size.sh
```

Task 5: Create /root/career.sh that:
Outputs: Yes, iam a System Engineer when run ./caree.sh me
Outputs: Okay, they do cloud engineering when run ./career.sh they
Ouputs: "Usage: ./career.sh me|they" for invalid/empty arguemnts 
```
#!/bin/bash
if [ "$1" == "me" ]; then
	echo "Yes, iam a System Engineer"
	exit 0
elif [ "$1" == "they" ]; then
	echo "Okay, they do cloud engineering"
	exit 0
else
	echo "Usage: ./career.sh me|they"
	exit 1
fi
```

```
chmod +x career.sh
./career.sh me
Yes, Iam a System Engineer
./career.sh they
Okay, they do cloud engineering
./career.sh aa
Usage: ./career.sh me|they 
```

Task 6:
Write shell scripts on node1 that create users and groups according to that following parameters

maryam:2030:hpc_admin,hpc_managers
adam:2040:sysadmin
jacob:2050:hpc_admin

write a shellscript that sets the pas sword of the users maryam, adam, and jacob to Password@1.
```
MARYAM=maryam:2030
ADAM=adam:2040
JACOB=jacob:2050
USERS=$(echo $MARYAM, $ADAM, $JACOB)

for entry in $MARYAM $ADAM $JACOB; do
        user=$(echo $entry | cut -d: -f1)
        uid=$(echo $entry | cut -d: -f2)
        useradd -u $uid $user
        echo "Password@1" | passwd --stdin $user;
done
        echo "Users with UID custom $USERS created with password"

echo "------------------ Creat group & add user ------------"
groupadd hpc_admin
groupadd sysadmin
groupadd hpc_managers

usermod -aG hpc_admin maryam
usermod -aG sysadmin adam
usermod -aG hpc_admin jacob
usermod -aG hpc_managers maryam

exit 0
```

Task 7:
Break into node2 and set a new root password to hoppy.

Task 8:
Check the current recommended tuniung profile
Put SELinux in permissive mode on node2
On node1 ensure network is enabled and starts on boot.
```
tuned-adm recommend 
virtual-guest 
tuned-adm profile virtual-guest 
```

```
[root@node2 ~]# setenforce 0
[root@node2 ~]# getenforce 
Permissive
```

```
systemctl enable --now NetworkManager
systemctl status NetworkManager
```

Task 9:
Break root password node2
```
rd.break
mount -o remount,rw /sysroot
chroot /sysroot
echo "NewPassword" | passwd --stdin root
touch /.autorelabel
exit
exit
```

Task 9:
Configure persistent journaling on both servers to retain logs across reboots.

```
chown root:systemd-journal /var/log/journal/ && chmod 2755 /var/log/journal/
systemctl restart systemd-journal-flush.service 
```

Task 10:
Copy /etc/fstab to /var/tmp
Set the file owner to root
ensure /var/tmp/fstab is not executable by anyone
configure file ACLs on the copied file to"
- User adam: read e write.
- User Maryam: no access
- All other users: ready only
```
cp /etc/fstab /var/tmp/fstab 
chown root: /var/tmp/fstab 
chmod 644 /var/tmp/fstab 

setfacl -m u:adam:rw /var/tmp/fstab 
setfacl -m u:maryam:0 /var/tmp/fstab

getfacl /var/tmp/fstab 
# file: etc/fstab
# owner: root
# group: root
user::rw-
user:maryam:---
user:adam:rw-
group::r--
mask::r--
other::r--
```


Task 11:
On node1 create a file node1-file.ext and securely copy (scp) it to the home dir of user hadoga on node2.
```
touch node1-file.ext
scp /root/node1-file.ext hadoga@node2:/home/hadoga
```

Task 12 
Validate if file node1-file.ext exist on node2 with shellscript
```
#!/bin/bash
FILE=node1-file.ext
if [ -f $FILE ]; then
	echo "list the file node1"
       	cat $FILE
	exit 0
else
	echo "file dont exist"
fi
exit 0
```

Task 13:
Start a stress-ng process on node1 with a niceness value of 19
Ajust the niceness value of the running stress-ng process to 10
Terminate the stress-ng process
```
nice -n 19 stress-ng --cpu 2 &
renice -n 10 -g $(pidof stress-ng)
ps -eo comm,nice,pid | grep stress-ng
stress-ng        10    2797
stress-ng-cpu    10    2798
stress-ng-cpu    10    2799
kill $(pidof stress-ng)
```

Task 14:
Create a Logical volume named devops_lv with 32 extents using the /dev/vdb disk.

This should be created from a volume group named devops_vg with 20MB physical extents. Format LVM with an ext4 filesystem  and mount presistently at /mnt/devops_lv
```
fdisk /dev/vdb
vgcreate --physicalextentsize 20M devops_vg /dev/vdb1
lvcreate -l 32 -n devops_lv devops_vg
mkfs.ext4 /dev/devops_vg/devops_lv
mkdir -p /mnt/devops_lv
mount /dev/devops_vg/devops_lv /mnt/devops_lv/
echo "/dev/devops_vg/devops_lv /mnt/devops_lv ext4 defaults 0 0" >> /etc/fstab
systemctl daemon-reload
```

task 15:
From /dev/vdb, create a 800MB swap partition and configure to mount persistently

all your changes must persist after a reboot
```
fdisk /dev/vdb
mkswap /dev/vdb2
swapon /dev/vdb2
echo "/dev/vdb2 none swap defaults 0 0" >> /etc/fstab
```

Task 16:
On node1, resize the existing devops_lv logical volume to 250mb (a size between 225 270mb), while resizeing its filesystem accordingly.

```
lvresize -L 250M /dev/devops_vg/devops_lv -r
Y
```

Task 17:
Create a user named bobby with UID of 4000, with no home directory, and a base directory of /mnt/netdir and a password of hoppy.
Configure NFS autofs such that the home directory of user bob is automatically mounted at /mnt/netdir/bobby on login.
Not that bobby account has been configured on the NFS server exporting thei home directory at repo.rhcsa.home:/home/bobby.
Login as user bobby to verify your config.
```
vim /etc/export
/home/bobby *(rw,sync,no_root_squash)

vim /etc/auto.master
/mnt/netdir /etc/auto.bobby

vim /etc/auto.bobby
bobby -rw,soft,intr repo.rhcsa.home:/home/bobby

mkdir -p /home/bobby
chown bobby:bobby /home/bobby
chmod 700 /home/bobby

systemctl restart autofs
systemctl restart nfs-server
su - bobby
```

Task 18:
Create web data and sales data directories in /export/web_data and /export/sales_data on node 1. Persistently mount web data and sales data on node2 under /mnt/nfs_web_data and /mnt/nfs_sales_data
```
node1:
vim /etc/exports
/sales_data *(rw,sync,no_root_squash)
/web_data *(rw,sync,no_root_squash)

showmount -e
/sales_data *
/home/bobby *
```

```
node2:
mkdir -p /mnt/nfs_sales_data
mkdir -p /mnt/nfs_web_data
dnf install -y nfs-utils
echo "192.168.1.60:/sales_data /mnt/nfs_sales_data nfs defaults 0 0" >> /etc/fstab
echo "192.168.1.60:/web_data /mnt/nfs_web_data nfs defaults 0 0" >> /etc/fstab
```

Task 19:
Create a cron job for user hadoga that run logger "RHCSA PLaylist Now Available" every 2 minutes.
Use at to write "This task was easy" to /at-files/at.txt in 2 minutes.
```
crontab -e -u hadoga
*/2 * * * * logger "RHCSA Playlist Now Available"

mkdir -p /at-files

echo "This task was easy" >> /at-files/at.txt | at now + 2 minutes
```

Task 20:
NTP: Chrony configuration
Setup chrony time service to sync time with server.rhel.com
```
vim /etc/chrony.conf
pool server.rhel.com iburts

put # in front #pool 2.rhel.pool.ntp.org iburst
systemctl restart chronyd
chronyc sources
```

Task 22:
Set GRUB_TIMEOUT=10,
GRUB_TIMEOUT_STYLE=hidden
add quiet to GRUB_CMDLINE_LINUX
Apply your changes to the grub config file.
```
vim /etc/default/grub
GRUB_TIMEOUT=10
GRUB_TIMEOUT_STYLE=hidden
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true

grub2-mkconfig -o /boot/grub2/grub.cfg
```

Task 21 on node1:
Ensure network service starts at boot
Allow access ssh and http services using firewall-cmd
Static ip configuration
- IP 172.16.1.10/24, gateway 172.16.1.1, DNS 172.16.1.1
- Hostname node5.rhcsa
- Set DNS search domain to rhel.server.com 
```
systemctl status NetworkManager
● NetworkManager.service - Network Manager
     Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; preset: enabled)
     Active: active (running) since Sun 2025-08-31 21:36:41 -03; 4h 14min ago
       Docs: man:NetworkManager(8)
   Main PID: 882 (NetworkManager)
      Tasks: 3 (limit: 10921)
     Memory: 11.9M
        CPU: 3.905s
     CGroup: /system.slice/NetworkManager.service
             └─882 /usr/sbin/NetworkManager --no-daemon

```

```
firewall-cmd --add-service=ssh --permanent 
firewall-cmd --add-service=http --permanent 
firewall-cmd --reload
```

```
hostnamectl set-hostname node5.rhcsa

nmcli con add con-name rhcsa ifname ens18 type ethernet autoconnect yes ipv4.method manual ip4 172.16.1.10/24 gw4 172.16.1.1 ipv4.dns 172.16.1.1 ipv4.dns-search rhel.server.com

nmcli connection up rhcsa

systemctl daemon-reload
```

Task 23:
Create a group named sharegroup and the following users:
- haruna (with no login shell, not a member of sharegroup)
- umar (member of sharegoup)
- adoga (with uid 4444 member of sharegroup)
- all user should have a password: persward
- Change the password of user adoga to perfect
Enforce password to ahve a minimum length of 8 chars
Set the max password age to 30 days

Remove the user umar form sharegroup
Delete the  sharegroup
delete the user haruna with their home directory

```
groupadd sharegroup
for i in umar adoga; do useradd -m $i; done
useradd -m -s /sbin/nologin haruna
usermod -u 4444 adoga

for i in umar adoga; do 
usermod -aG sharegroup $i; done

for i in umar adoga haruna; do 
echo "persward" | passwd --stdin $i; done

echo "perfect" | passwd --stdin adoga
vim /etc/login.defs 

PASS_MIN_LEN    8
PASS_MAX_DAYS   30

userdel -r haruna
gpasswd -d umar sharegroup
groupdel sharegroup
```

Task : Generate SSH key on node1, and setup key-based authentication for root user 
Enable SSH root login and teste your configuration
```
ssh-keygen
ssh-copy-id root@localhost
vim /etc/ssh/sshd_config
PubkeyAuthentication yes
PermitRootLogin yes

systemctl restart sshd

ssh root@localhost
```

Task 
The web server on node1 is listening on port 85, ensure that the site is reachable via that port. 
FIX Selinux context if needed
Test using curl http://server-ip/ or use web browser.

```
[root@node2 ~]# cat /etc/httpd/conf/httpd.conf  | grep ^Listen
Listen 82
vim /etc/httpd/conf/httpd.conf
Listen 85
firewall-cmd --add-port=85/tcp --permanent
firewall-cmd --reload

semanage port -l | egrep http
http_port_t            tcp      82, 80, 81, 443, 488, 8008, 8009, 8443, 9000
semanage port -a -t http_port_t -p tcp 85
echo "RHCSA works!" >> /var/www/html/index.html

need to kill pid httpd:
kill $(pidof httpd)

curl localhost:85
WELCOME HELLO WORLD
RHCSA works!
```
