Q-1 Configure hostname and IP Address on this machine server:

Server A
Ip Address: 192.168.1.50
Netmask: 255.255.255.0
Gateway: 192.168.1.1
DNS: 192.168.1.1
Domain: lab.example.com
Hostname: severa.lab.example.com

```
echo "severa.lab.example.com" >> /etc/hostname
```

```
vim /etc/resolv.conf
search lab.example.com
nameserver 192.168.1.1
nameserver fe80::82ea:bff:fedc:8230%ens18
```

```
nmcli con del ens18
nmcli con add con-name ens18 ifname ens18 type ethernet autoconnect yes ip4 192.168.1.50/24 gw4 192.168.1.1 ipv4.dns 192.168.1.1 ipv4.method manual ipv4.dns-search lab.example.com
```

Server b
Ip Address: 192.168.1.60
Netmask: 255.255.255.0
Gateway: 192.168.1.1
DNS: 192.168.1.1
Domain: lab.example.com
Hostname: severb.lab.example.com
```
hostnamectl hostname severb.lab.example.com
```

```
nmcli con del ens18
nmcli con add con-name ens18 ifname ens18 type ethernet autoconnect yes ip4 192.168.1.60/24 gw4 192.168.1.1 ipv4.dns 192.168.1.1 ipv4.method manual ipv4.dns-search lab.example.com
```


Q-2 Configure YUM using the following URLs:
http://classroom.example.com/pub/rhel9/BaseOS
http://classroom.example.com/pub/rhel9/AppStream

```
vim /etc/yum.repos.d/classroom.repo
```

```
[classroom-BaseOS]
name=classroom BaseOS
baseurl=http://classroom.example.com/pub/rhel9/BaseOS
enabled=1
gpgcheck=0

[classroom-AppStream]
name=classroom AppStream
baseurl=http://classroom.example.com/pub/rhel9/AppStream
enabled=1
gpgcheck=0

dnf repolist
```

Q3 Add an additional SWAp partition of 512 MiB to your system. The SWAP Partition should be mounted automatically at the time of the system booting.

```
fdisk /dev/sdb
+512M
t
82
w

mkswap /dev/sdb1
swapon /dev/sdb1
echo "/dev/sdb1 none swap defaults 0 0" >> /etc/fstab
```

Q4 A group named sales
A user nancy who belongs to sales as a secondary groups
A user mike whos belongs to sales as a secondary group
A user suny who does not have access to an interactive shell on the system, and not a member of sales group
Nancy, Mikle, and Sunny all have the password redhat
```
groupadd sales

for user in nancy mike; do useradd $user; usermod -aG sales $user; done

useradd sunny -s /sbin/nologin

for user in nancy mike sunny; do echo "redhat" | passwd --stdin $user;done
```
OR
```
useradd sunny -s /sbin/nologin 
useradd nancy 
useradd mike 
groupadd sales 
usermod -aG sales nancy 
usermod -aG sales mike 
passwd sunny 
passwd nancy 
passwd mike
```

Q4 Create a new logical volume according to the following requirements:
	1. LVM names wshare which belongs to the wgroup volume group and has size of 100 extents
	2. LVM in the WGROUP should have an extent size of 8MiB
	3. FOrmat the new logical volume with vfat file system.
NOTE: The logical volume should mount automatically on /mnt/wshare the time of the system booting.

```
fdisk /dev/sdc
vgcreate --physicalextentsize 8MiB wgroup /dev/sdc1
lvcreate -l 100 -n wshare wgroup 
mkdir -p /mnt/wshare
mkfs.vfat /dev/wgroup/wshare 
mount /dev/wgroup/wshare /mnt/wshare
vim /etc/fstab 
/dev/wgroup/wshare /mnt/wshare vfat defaults 0 0
systemctl daemon-reload 
```

Q5 Create a collaborative directory /home/collab with the following charactereistics:
- Group ownership of /home/collab is sales
- The directory should be readable, writeble and accesible to members of sales but no to any other user (we know that root has access to all the files and directories of the system)
- Files created in /home/collab automatically have group ownership set to the sales group
```
mkdir -p /home/collab
chown :sales /home/collab
chmod 2770 /home/collab
[root@servera home]# ls -lha
total 4.0K
drwxr-xr-x.  7 root          root            79 Aug 28 01:49 .
dr-xr-xr-x. 19 root          root           268 Aug 28 01:06 ..
drwxrws---.  2 root          sales           16 Aug 28 01:52 collab
```

Q6 Create a user dolly with uid 5120. The password for this user should be redhat
```
useradd -u 5120 dolly
echo "redhat" | passwd --stdin dolly
```


Q7 The user nancy must configure a cron job that run daily at 15:25 localtime and executes /usr/bin/echo hi
```
crontab -e -u nancy
25 15 * * * /usr/bin/echo hi
```

Q8 Find all the lines in the file /tmp/test.txt that contains the string start.
Put a copy of all these lines in the original order in the file /root/lines.text

```
cat /tmp/teste.txt | grep start > /root/lines.text
```

Q9. Create a tar archive named /root/data.tar.bz2 which contains the /usr/local contents. The tar must be comprressed usign bzip2. 
```
tar -cjvf /root/data.tar.bz2 /usr/local
```
or
```
tar -cvf /root/data.tar /usr/local
bunzip2 /root/data.tar
```

	
Q10 locate all the files o owned by a user smith and place a copy of them in the /root/found directory
```
mkdir -p /root/found
find / -user smith -exec cp -rf {} /root/found/ \;
```

Q11 Choose the recommend tuned profile for the system and set it the default
```
tuned-adm list
tuned-adm recommend 
tuned-adm profile virtual-guest 
tuned-adm active 
```

Q12 LOcate all uncommented lines in file /etc/sudoers and copy the lines in the same order in /root/list file
```
cat /etc/sudoers | grep -v ^# >> /root/list
```

Q13 A web server running on non-standard port 82 is having issue serving contents, debug and fix the issues as necessary so that:
1. The webserver on your system can serve all the existing HTML files from /var/www/html
2. The web server this content on port 82
3. This web server start automatically at system boot time

```
firewall-cmd --list-ports
firewall-cmd --add-service=http --permanent
firewall-cmd --add-ports=82/tcp --permanent
firewall-cmd --reload
```

```
semanage port -l | grep http
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
semanage port -a -t http_port_t -p tcp 82
```

```
cat /etc/httpd/conf/httpd.conf | grep ^Listen
Listen 80
vim /etc/httpd/conf/httpd.conf
Listen 82
```

```
systemctl restart httpd
ss -tlnp | grep 82
```

Q14 Copy the file /etc/fstab to /var/tmp/ direcotry
Configure the permission of /var/tmp/fstab so that:
The file /var/tmp/fstab is owned by root user
The file /var/tmp/fstab belongs to the group root
The file /var/tmp/fstab should not be executed by anyone 
the user harry is able to read and write by /var/tmp/fstab
the user natash can neitehr read nor write /var/tmp/fstab
all the other users (current and future) have have the ability to read /var/tmp/fstab
```
for user in natasha harry; do useradd $user; done
cp /etc/fstab /var/tmp
chown root:root /var/tmp/fstab
chmod 664 /var/tmp/fstab
setfacl -m u:harry:rw /var/tmp/fstab
setfacl -m u:natasha:0 /var/tmp/fstab
```

Q15 Enabled sudo access for members of group admin. Without entering their password
```
echo "%admin        ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers.d/admin
visudo -cf /etc/sudoers.d/admin
/etc/sudoers.d/admin: parsed OK
```

Q16 all newly craeted files for the user maxwell should have -rw-r-----
All newly created directorties for the above user should have drwxr-x---- as the default permissions
```
Diretorio = 777 - 750 = umask 027
Arquivos = 666 - 640 = umask 026

  echo "umask 026" >> /home/maxwell/.bashrc
  setfacl -d -m u::rwx /home/maxwell/
  setfacl -d -m g::rx /home/maxwell/
  setfacl -d -m o::0 /home/maxwell/
  
getfacl /home/maxwell/
# file: home/maxwell/
# owner: maxwell
# group: maxwell
user::rwx
group::---
other::---
default:user::rwx
default:group::r-x
default:other::---
```


Q17 find all the files under the /usr/share less than 5MB and save these files in the /root/myfiles

after xecuting the mysearch script file and listed files has to be copied under /root/myfiles

```
mkdir -p /root/myfiles

vim mysearch.sh
chmod +x mysearch.sh
./mysearch.sh

#!/bin/bash
clear
echo "----------------------------------------- LISTA FILE UNDER 5MB ---------------------------------"
find /usr/share -type f -size -5M
echo "-------------------------------------- COPY TO /root/myfiles -----------------------------------------"
find /usr/share -type f -size -5M -exec cp "{}" /root/myfiles/ \;
echo "----------------------------------------- DONE -----------------------------------------"
exit 0
```

18Q The container is named mycontainer
The Container uses the myimage image that was previously created in another item
The containers runs as a systemd service as the existing user mike only
The service is named container-mycontainer
The local directory /opt/files is attached to the container's /opt/incoming directory
the ocal directory /opt/processed is attached to thec onmtainers /opt/outgoing directory
The container automatically start after system reboot without any manual intervention

If the service is working properly any plain text file placed  in /opt/files will automatically be converted to a pdf file and placed into /opt/outgoing with th same file name.
```
chmod 777 /opt
su - mike
mkdir -p /opt/files && mkdir -p /opt/processed

touch /opt/file/oi{1..5} && touch /opt/processed/alo{1..5}

podman run -d --name mycontainer -v /opt/files:/opt/incoming:Z -v /opt/processed:/opt/outgoing:Z localhost/myimage

[mike@servera ~]$ podman exec mycontainer ls /opt/
incoming
outgoing

mkdir -p /home/mike/.config/systemd/user/
cd /home/mike/.config/systemd/user/
podman generate systemd --name mycontainer --files
sudo loginctl enable-linger mike
ssh mike@localhost
systemctl --user enable container-mycontainer.service
systemctl --user start container-mycontainer.service
```


Q19 Deny the natasha user to create cronjob in the system
```
echo "natasha" >> /etc/cron.deny
```

Q20 The password for all users in node2 should expire after 20 days
```
vim /etc/login.defs
PASS_MAX_DAYS 20
```

Q21 resize the LVM volume lv1 and its filesystem to 250mib, make sure that the filesystem contents remain intact

Note:
Partitions are seldome exactly the same size requested, so, a size within the range of 255MiB to 275 mIb is acceptable.

```
lvs
lvresize -L 250M /dev/wgroup/lv1
lvs
lv1    wgroup -wi-a----- 256.00m 
```

Q21 Find a .conf extention file name in /etc directory and copy them in /search file.
```
mkdir /search
find /etc -type f -name *.conf -exec cp {} /search \;
ls /search/ | head -n 10
000-shortnames.conf
001-rhel-shortnames.conf
002-rhel-shortnames-overrides.conf
00-base.conf
00-brotli.conf
00-dav.conf
00-keyboard.conf
00-lua.conf
00-mpm.conf
00-optional.conf
```

15. Create a script file with the name script.sh inside the /bin and locate all files from /usr/local directory which has more than 3k, and less than 5k and copy under /root/d1
```
#!/bin/bash

if [ -d /root/d1 ];
then
        echo "diretorio criado"
else
        mkdir -p /root/d1
        chmod 2770 /root/d1
        echo "Criando diretorio /root/d1"
fi

echo "============= Procurando arquivos acima de 3k, e menos de 5k =========="
find /usr/local -type f -size +3k -size -5k -exec cp {} /root/d1/ \;

exit 0
```

Q16 Recover Root Password, pasword should be Redhat@321
(sometimes it doesnt specify any password so that case you can set any password)
```
egrub
linux rescue
last line with begin linux
rd.break
ctrl+x
mount -o remount,rw /sysroot
chroot /sysroot
echo "redhat@321" | passwd --stdin root
touch /.autorelabel
exit
exit
```

Q17 Create a container image using the link:
XXXXXXXXXXXXXXXXXXX.com
Named it  "image32"
login to 'registry.lab.example.com' through 'admin' and 'Admin@123'
The container should be created with user "Sophia"

