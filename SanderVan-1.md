After applying these tips, you’re ready to get started. Good luck!

1. Install a RHEL 9 virtual machine that meets the following requirements:
    
    - 2 GB of RAM
        
    - 20 GB of disk space using default partitioning
        
    - One additional 20-GB disk that does not have any partitions installed
        
    - Server with GUI installation pattern
        
2. Create user **student** with password **password**, and user **root** with password **password**.
    
3. Configure your system to automatically mount the ISO of the installation disk on the directory **/repo**. Configure your system to remove this loop-mounted ISO as the only repository that is used for installation. Do _not_ register your system with **subscription-manager**, and remove all references to external repositories that may already exist.
```
lsblk
sr0                 11:0    1 11.9G  0 rom 
mkdir -p /repo
mount /dev/cdrom /repo
rm -f /etc/yum.repos.d/*.rep
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

subscription-manager unregister 
subscription-manager clean

```

4. Reboot your server. Assume that you don’t know the root password, and use the appropriate mode to enter a root shell that doesn’t require a password. Set the root password to **mypassword**.
```
rd.break
CTRL+X
mount -o remount,rw /sysroot
chroot /sysroot
passwd
exit
exit
```


5. Set default values for new users. Make sure that any new user password has a length of at least six characters and must be used for at least three days before it can be reset.
```
vim /etc/login.defs
PASS_MAX_DAYS   3
PASS_MIN_LEN    6
```

6. Create users **linda** and **anna** and make them members of the group **sales** as a secondary group membership. Also, create users **serene** and **alex** and make them members of the group **account** as a secondary group.
```
useradd linda
useradd anna
groupadd sales
usermod -aG sales linda
usermod -aG sales anna
useradd alex
useradd serene
usermod -aG account serene
usermod -aG account alex
```

7. Configure an SSH server that meets the following requirements:

    - User root is allowed to connect through SSH.
```
    vim /etc/ssh/sshd_config
    PermitRootLogin yes
    systemctl restart sshd
```

		
-     - The server offers services on port 2022.
```
	 vim /etc/ssh/sshd_config
     Port 2022
     PermitRootLogin yes
     semanage port -a -t ssh_port_t -p tcp 2022
     firewall-cmd --add-port=2022/tcp --permanent
     firewall-cmd --reload
     systemctl restart sshd
```
8. Create shared group directories **/groups/sales** and **/groups/account**, and make sure these groups meet the following requirements:
```
mkdir -p /groups/sales && mkdir -p /groups/account
```

- Members of the group sales have full access to their directory.
```
chown :sales /groups/sales
chmod 2770 /groups/sales
```


-  Members of the group account have full access to their directory.
```
chown :account /groups/account
chmod 2770 /groups/account
```

- Users have permissions to delete only their own files, but Alex is the general manager, so user alex has access to delete all users’ files.
```
chmod +t /groups/account
chmod +t /groups/sales
chown alex:account /groups/account
chown alex:sales /groups/sales
```


9. Create a 4-GiB volume group, using a physical extent size of 2 MiB. In this volume group, create a 1-GiB logical volume with the name **myfiles**, format it with the Ext3 file system, and mount it persistently on /myfiles.
```
fdisk /dev/sdb (feito os procedimentos para ter 4GB + LVM)
lsblk
sdb1                  8:17   0    4G  0 part
vgcreate --physicalextentsize 2MiB vgestudo /dev/sdb1
lvcreate -L 1G -n myfiles vgestudo
mkfs.ext3 /dev/vgestudo/myfiles
mkdir -p /myfles
mount /dev/vgestudo/myfiles /myfiles
echo "/dev/vgestudo/myfiles /myfiles ext3 defaults 0 0" >> /etc/fstab
```


10. Create a group **sysadmins**. Make users linda and anna members of this group and ensure that all members of this group can run all administrative commands using **sudo**.
```
groupadd sysadmins
usermod -aG sysadmins linda
usermod -aG sysadmins anna
echo "%sysadmins ALL=(ALL) ALL" >> /etc/sudoers.d/sysadmins
```


11. Optimize your server with the appropriate profile that optimizes throughput.

12. Add a new disk to your virtual machine with a size of 10 GiB. On this disk, create a LVM logical volume with a size of 5 GiB, configure it as swap, and mount it persistently.
```
fdisk /dev/sdc
vgcreate vgswap /dev/sdc1
lvcreate -L 5G -n lvmswap vgswap
mkswap /dev/vgswap/lvmswap
swapon /dev/vgswap/lvmswap
echo  "/dev/vgswap/lvmswap none swap defaults 0 0" >> /etc/fstab
```

13. Create a directory **/users/** and in this directory create the directories **user1** through **user5** using one command.
```
mkdir -p /users/user{1..5}
```

14. Configure a web server to use the nondefault document root /webfiles. In this directory, create a file **index.html** that has the contents **hello world** and then test that it works.
```
dnf group install "Basic Web Server"
vim /etc/httpd/conf/httpd.conf
DocumentRoot "/webfiles"
<Directory "/webfiles">
mkdir -p /webfiles
echo "Hello world" >> /webfiles/index.html
semanage fcontext -a -t httpd_sys_content_t "/webfiles(/.*)?"
restorecon -Rv /webfiles
systemctl restart httpd
```

15. Configure your system to automatically start a mariadb container. This container should expose its services at port 3306 and use the directory /var/mariadb-container on the host for persistent storage of files it writes to the /var directory.
```
podman run -d \
  --name mariadb \
  -e MARIADB_ROOT_PASSWORD=password \
  -e MARIADB_USER=anna \
  -e MARIADB_PASSWORD=password \
  -e MARIADB_DATABASE=mydb \
  -p 3306:3306 \
  -v /var/mariadb-container:/var/lib/mysql \
  --restart always \
  mariadb:latest
```

16. Configure your system such that the container created in step 15 is automatically started as a Systemd user container.
```
# Como usuário comum (não root)
mkdir -p ~/.config/systemd/user
cd ~/.config/systemd/user

# Gerar arquivo unit do systemd para o contêiner existente
podman generate systemd --name mariadb --files

# Recarregar systemd user
systemctl --user daemon-reload

# Habilitar para iniciar automaticamente
systemctl --user enable --now container-mariadb.service

# Verificar status
systemctl --user status container-mariadb.service
```