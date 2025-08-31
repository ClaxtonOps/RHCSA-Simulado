Node1 Questions:
1. Configure a static IP with the following details:
	. IP: 192.168.1.20
	. Gateway: 192.168.1.2
	. DNS: 192.168.1.2
	. DomainName: machine1.example.com
	. Make sure networking starts automatically on boot
```
hostnamectl set-hostname machine1.example.com

nmcli con add con-name node1-ens18 ifname ens18 autoconnect yes ipv4.method manual type ethernet ip4 192.168.1.20 gw4 192.168.1.2 ipv4.dns 192.168.1.2 ipv4.dns-search machine1.example.com

nmcli con up node1-ens18
```

2. Configure the YUM repo using given links:
http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht

```
vim /etc/yum.repos.d/link.repo
[link-baseos]
name=BaseOS from URL
enabled=1
baseurl=http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht/BaseOS
gpgcheck=0

[link-appstream]
name=AppStream from URL
enabled=1
baseurl=http://content.example.com/rhel9.0/x86_64/rhcsa-practice/rht/AppStream
gpgcheck=0
```

```
dnf clean all
dnf repolist
```

3. Creata a directory /share/devteam
	. Give ownership to devuser:devteam
	.Set permission so that members of devteam can read, write, and execute inside /share/dev
	. use ACL to allow user john to have read-only access
```
mkdir -p /share/devteam
chown devuser:devteam /share/devteam
chmod 770 /share/devteam
setfacl -m u:john:r /share/devteam
```

4. Create a user called redhat
	. Set the Password to Redhat@123
		. Create a group called IT-Team and add Redhat to this group
```
useradd redhat
groupadd IT-Team
usermod -aG IT-Team redhat
echo "Redhat@123" | passwd --stdin redhat
```

5. Configure passwordless SSH login with node2 as the natasha user.
	. User ssh-keygen and ssh-copy-id
	. Teste connectivity to ensure it works

```
echo "192.168.1.60 node2" >> /etc/hosts
ssh-keygen
ssh-copy-id natasha@node2
ssh natasha@node2
```

6. A web server running on non-standard port 82 is having issues content. Debug and fix the issues.
```
cat /etc/httpd/conf/httpd.conf | egrep ^Listen
Listen 80 (change for 82)

semanage port -l | grep http_port_t
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000

semanage port -a -t http_port_t -p tcp 82

firewall-cmd --add-port=82/tcp --permanent
firewall-cmd --reload

ss -tlnp | grep 82
LISTEN 0      511                *:82              *:*    users:(("httpd",pid=36995,fd=4),("httpd",pid=36994,fd=4),("httpd",pid=36993,fd=4),("httpd",pid=36992,fd=4),("httpd",pid=36990,fd=4))

curl localhost:82
WELCOME HELLO WORLD
```

7. Create a logical volume lvdata of size 500m and volumegroup vgdata using /dev/sdb
	. Format it with xfs, mount permanently at /mnt/data

```
fdisk /dev/sdb
vgcreate vgdata /dev/sdb1
lvcreate -L 500M -n lvdata vgdata
mkfs.xfs /dev/vgdata/lvdata 
mkdir -p /mnt/data
mount /dev/vgdata/lvdata /mnt/data/
echo "/dev/vgdata/lvdata /mnt/data xfs defaults 0 0" >> /etc/fstab 
systemctl daemon-reload
```

8. Create a cronjob for user devuser that runs the command uptime every 5 minutes and saves output to /home/devuser/uptime.log
```
crontab -e -u devuser
*/5 * * * * uptime >> /home/devuser/uptime.log
```

9. Compress the directory /etc using gzip and save it as /root/etc_backup.tar.gz
```
tar -czvf /root/etc_backup.tar.gz /etc 
```

10. Create user "alex" and make alex user secondary group of group IT-Team
```
useradd alex
groupadd IT-Team
usermod -aG IT-Team alex
```

11. Create user
	John
	uid 2245
	Set the user password redhat@321
```
useradd -u 2245 john
echo "redhat@321" | passwd --stdin john
```

12. Locate all files owned by user tom and copy it under /root/tomfiles
```
mkdir -p /root/tomfiles
find / -type f -user tom -exec cp {} /root/tomfiles/ \;
```

13. locate all files and directories owned by user jerry and copy it under /root/jerryfiles
```
mkdir -p /root/jerryfiles
find / -user jerry -exec cp -rf {} /root/jerryfiles/ \;
```
	

14. Create directory /shared/projects on node1
	Set permissions so that all users can write but only file owners can delete their files
```
mkdir -p /shared/projects
chmod 777 /shared/projects
chmod +t /shared/projects
```

15. create a container image using this:
Named it "myimage"
Login to registry.redhat
The container should be craeted with user "Sophia"
FROM registry.access.redhat.com/ubi9/ubi:latest
RUN dnf install -y python3
RUN mkdir -p /opt/incoming /opt/outgoing
CMD [ "/bin/bash", "-c", "sleep infinity"]

```
useradd sophia
ssh sophia@localhost
podman login registry.lab.example
vim Containerfile
FROM registry.access.redhat.com/ubi9/ubi:latest
RUN dnf install -y python3
RUN mkdir -p /opt/incoming /opt/outgoing
CMD [ "/bin/bash", "-c", "sleep infinity"]

podman build -t myimage .
```

16. The container should be created with same regular user used in creating image
	1. The service must be enabled so you could start automatically after reboot
	2. The container name container24 should be created using previous image
	3. map the '/opt/files' to container '/opt/incoming'
	4. Map the /opt/processed to container /opt/outgoing
	5. Created systemd service as container-container24.service
	6. make service active after all server reboot
```
su -
chown -R sophia:sophia /opt/files
chown -R sophia:sophia /opt/processed
loginctl enable-linger sophia
ssh sophia@localhost
```

```
podman run -d --name container24 -v /opt/files:/opt/incoming:Z -v /opt/processed:/opt/processed:Z localhost/myimage

mkdir -p /home/sophia/.config/systemd/user
cd /home/sophia/.config/systemd/user
podman generate systemd container24 --files
mv container* container-container24.service
systemctl enable --user container-container24.service
systemctl start --user container-container24.service
systemctl daemon-reload --user
```

```
systemctl status --user container-container24.service 
● container-container24.service - Podman container-763c7e7c3aed6c8a95f6ebf198664a53fbd934739d28b68a50a5>
     Loaded: loaded (/home/sophia/.config/systemd/user/container-container24.service; enabled; preset: >
     Active: active (running) since Fri 2025-08-29 19:00:32 -03; 18min ago
       Docs: man:podman-generate-systemd(1)
   Main PID: 1344 (conmon)
      Tasks: 2 (limit: 10921)
     Memory: 72.3M
        CPU: 303ms
     CGroup: /user.slice/user-1002.slice/user@1002.service/app.slice/container-container24.service
             ├─1342 /usr/bin/pasta --config-net --dns-forward 169.254.1.1 -t none -u none -T none -U no>
             └─1344 /usr/bin/conmon --api-version 1 -c 763c7e7c3aed6c8a95f6ebf198664a53fbd934739d28b68a>

```

17. Create a  SWAP partition of 400MiB
- DO not modify current swap partition
- It should be persistent after reboot
```
fdisk /dev/sdc (g, n, enter, 500M, t, swap, w)
mkswap /dev/sdc1
swapon /dev/sdc1
echo "/dev/sdc1 none swap defaults 0 0" >> /etc/fstab
```

18. Create a logical volume called LV1 under the volume group called VG1 with 16M extend and logical extend should be 30 extend.
```
 fdisk /dev/sdc (g, n, enter, enter, t, swap, w)
 vgcreate --physicalextentsize 16M VG1 /dev/sdd1
 lvcreate -l 30 -n LV1 VG1
 mkfs.ext4 /dev/VG1/LV1 
 mkdir -p /database
 mount /dev/VG1/LV1 /database
 echo "/dev/VG1/LV1 /database ext4 defaults 0 0" >> /etc/fstab 
  systemctl daemon-reload 
```

19. Configure recommended tuned profile
```
tuned-adm list
tuned-adm recommend 
tuned-adm profile virtual-guest 
```



