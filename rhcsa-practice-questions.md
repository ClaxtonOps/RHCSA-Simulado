### Question:
You do not know the root password but You have physical access to the machine. Create new root password and log into the system.
```
rd.break
mount -o remount,rw /sysroot
chroot /sysroot
passwd
touch /.autorelabel
```

### Question:
Please create new network connection using provided values: IP: some.ip.to.be.used/mask Gateway: some.gateway.to.be.used DNS server: some.dns.to.be.used Interface: eth0
```
nmcli connection add con-name rhcsa ifname eth0 autoconnect yes ipv4.method manual type ethernet ip4 192.168.1.77/24 gw4 192.168.1.2 ipv4.dns 192.168.1.2

nmcli connection up rhcsa
```

### Question:
Change hostname of the system
```
hostnamectl set-hostname node1.example.lab
```

### Question:
Enable SELinux in enforcing mode
```
setenforce 1
getenforce
Enforcing
```

