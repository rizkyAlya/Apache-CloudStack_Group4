## Computing Environment Setup

### Hardware Specification

```
CPU : Intel Core i5 gen 8
RAM : 24 GB
Storage : 200GB
Network : Ethernet 10GB/s
Operating System : Ubuntu Server 24.04
```

### Network Address

```
Network Address : 192.168.1.0/24
Host IP address : 192.168.1.13/24
Gateway : 192.168.1.1
Public IP : 139.192.5.123
```

## Configure Network 

### Edit the network configuration file under /netplan directory

```
sudo -i 
cd /etc/netplan
nano ./*.yaml
```

Edit the file

```
# This is the network config written by 'subiquity'
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0: #Your ethernet adapter
      dhcp4: false
      dhcp6: false
      optional: true
  bridges:
    cloudbr0:
      addresses: [192.168.1.13/24]  #Your host IP address
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
      interfaces: [enp1s0]
      dhcp4: false
      dhcp6: false
      parameters:
        stp: false
        forward-delay: 0
```

### Apply Network Configuration

```
netplan generate        #generate config file for the renderer
netplan apply           #applies network configuration to the system
reboot
```

> Note : You may encounter some error during this step, make sure you use "space" instead of "tab" when modifying network configuration file
> Note : To check whether your network configuration already applied or not, use ifconfig command and look for interface "br0", make sure the address is equal with the one you configured before



### Test the Network, make sure the configuration applied

```
ip address        #check the ip address and existing interface
ping google.com   #make sure you could connect to the internet
```

> Note : if you can't ping the google.com, try to ping your gateway and 8.8.8.8
> This step will tell you the connection problem between your computer and internet, make sure there's no problem with the internet as you will install some package from internet

### Login to the system as root user

```
sudo -i
passwd root
#change it to Pa$$w0rd
```

### Enable SSH root login

```
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
#restart ssh service
service ssh restart
#or
systemctl restart sshd.service
```

## Cloudstack Installation (Controller and Compute Node at the Same Host)


### Importing cloudstack repositories key

```
mkdir -p /etc/apt/keyrings 
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.20 / > /etc/apt/sources.list.d/cloudstack.list
```

* The first line is to create a directory to store cloudstack public key
* wget -O to download the given URL and redirect the output to 'gpg --dearmor' command
* 'gpg --dearmor' command will convert from ASCII armored to binary format
* 'sudo tee' command will redirect 'gpg --dearmor' command to file /etc/apt/keyrings/cloudstack.gpg

### Installing Cloudstack and Mysql Server

```
apt update -y
apt install cloudstack-management mysql-server
```

> Note : This process will take a long time

### Configure mysql

#### Open mysql config file

```
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

#### Copy these lines under [mysqld] section

```
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

#### Restart mysql service
```
systemctl restart mysql
```

### Deploy Database as Root and Then Create "cloud" User with Password "cloud" too

```
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:Pa$$w0rd -i 192.168.1.13
```

### Configure Primary and Secondary Storage

```
apt install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

### Configure NFS Server

```
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```
Explanation

* `sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server`
This command uses the sed (stream editor) command to search for the line RPCMOUNTDOPTS="--manage-gids" in the file /etc/default/nfs-kernel-server and replace it with RPCMOUNTDOPTS="-p 892 --manage-gids". The -i option tells sed to edit files in place (i.e., save the changes to the original file).

* `sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common`
This command uses sed to search for the line STATDOPTS= in the file /etc/default/nfs-common and replace it with STATDOPTS="--port 662 --outgoing-port 2020".

* `echo "NEED_STATD=yes" >> /etc/default/nfs-common`
This command appends the line NEED_STATD=yes to the end of the file /etc/default/nfs-common. The >> operator in bash is used to append output to a file.

* `sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota`
This command uses sed to search for the line RPCRQUOTADOPTS= in the file /etc/default/quota and replace it with RPCRQUOTADOPTS="-p 875".

* `service nfs-kernel-server restart`
This command will restart NFS Service

## Configure Cloudstack Host with KVM Hypervisor

### Install KVM and Cloudstack Agent

```
apt install qemu-kvm cloudstack-agent -y
```

### Configure KVM Virtualization Management

#### Change some lines

```
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

# On Ubuntu 22.04, add LIBVIRTD_ARGS="--listen" to /etc/default/libvirtd instead.

sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

Explanation

* `sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf`
This command uses the sed (stream editor) command to search for lines in the file /etc/libvirt/qemu.conf that start with #vnc_listen and replace them with vnc_listen = "0.0.0.0". The -i option tells sed to edit files in place (i.e., save the changes to the original file). The 0.0.0.0 address is a special IP address used in network programming to specify all IP addresses on the local machine.

* `sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd`
This command uses sed to search for lines in the file /etc/default/libvirtd that start with LIBVIRTD_ARGS= and replace them with LIBVIRTD_ARGS="--listen". The -i.bak option tells sed to edit files in place and make a backup of the original file with the .bak extension.


#### Add some lines

```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```
Explanation

* `echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf` disable TLS listening on libvirt daemon

* `echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf` enable TCP listening on libvirt daemon

* `echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf` specify a port where daemon will listen for a TCP connection

* `echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf` disable multicast DNS, libvirtd service will not found through nDNS

* `echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf` disable authentication for TCP connection to libvirt daemon


#### Restart libvirtd

```
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

Command Explanation

* `systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket`
 This command uses systemctl, the system and service manager for Linux, to mask several socket units related to the libvirtd service. Masking a unit in systemd effectively disables it and makes it impossible to start it manually or allow other services to start it. In this case, the command is masking several sockets that libvirtd uses to communicate with other processes. This might be done to prevent libvirtd from accepting connections over these sockets.

* `systemctl restart libvirtd`
This command uses systemctl to restart the libvirtd service. This is often necessary after making changes to a service’s configuration or its related units (like sockets), to ensure the changes take effect.

libvirtd is a daemon that provides management of virtual machines (VMs), virtual networks, and storage for various virtualization technologies, such as KVM, QEMU, Xen, and others. It is part of the libvirt project, which offers a toolkit for managing virtualization platforms. The primary purpose of libvirtd is to provide a consistent and secure API for managing VMs and associated resources, regardless of the underlying virtualization technology.


### Configuration to Support Docker and Other Services

```
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

> ARP and IP packet will not processed by arptable and iptable


### Generate Unique Host ID

```
apt install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

### Configure Iptables Firewall and Make it persistent

```
NETWORK=192.168.1.0/24
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT     # Portmap service (needed for NFS)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT     # Portmap service (needed for NFS)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT    # NFS server port
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT   # NFS mountd (dynamic port)
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT   # NFS nlockmgr
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT     # NFS rpc.mountd
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT     # 	NFS rquotad (quota)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT     # 	NFS statd
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8250 -j ACCEPT    # CloudStack Agent
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8080 -j ACCEPT    # CloudStack Management Server
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8443 -j ACCEPT    # CloudStack Management Server
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 9090 -j ACCEPT    # 	CloudStack Console Proxy (VM console)	
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 16514 -j ACCEPT   # libvirt (KVM communication)

apt install iptables-persistent
#just answer yes yes
```

This step ensuring all service port used by cloudstack didn't blocked by firewall and accessible by network

Explanation

* '-A input' will append the rule to INPUT rule chain
* '-s $NETWORK' specifying the packet source, in this case the source is network 192.168.101.0/24
* '-m state' using state module to match packet state, '--state NEW' means rules applied only for packet that start new connection
* '-p udp/tcp --dport [PORT NUMBER]' apply rule for specific protocol and destination port number
* '-j ACCEPT' accepting packet matched with the rule  



### Install cloudstack agent

```
systemctl unmask cloudstack-agent
apt update -y
apt install cloudstack-agent -y
```

### Disable apparmour on libvirtd

```
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

Explanation


* `ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/`
This command creates a symbolic link (a type of file that points to another file or directory) from /etc/apparmor.d/usr.sbin.libvirtd to /etc/apparmor.d/disable/. This effectively disables the AppArmor profile for usr.sbin.libvirtd.

* `ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/`
This command creates a symbolic link from /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper to /etc/apparmor.d/disable/, disabling the AppArmor profile for usr.lib.libvirt.virt-aa-helper.

* `apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd`
This command uses the apparmor_parser utility to remove the AppArmor profile for usr.sbin.libvirtd from the kernel. The -R option tells apparmor_parser to remove a profile.

* `apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper`
This command removes the AppArmor profile for usr.lib.libvirt.virt-aa-helper from the kernel.



### Launch management Server

```
cloudstack-setup-management
systemctl status cloudstack-management
```

Explanation

* `cloudstack-setup-management`
This command is used to set up the management server for Apache CloudStack, an open-source cloud computing software for creating, managing, and deploying infrastructure cloud services. It configures the database connection, sets up the management server’s IP address, and starts the management server.

* `systemctl status cloudstack-management`
This command uses systemctl, the system and service manager for Linux, to display the status of the cloudstack-management service. It shows whether the service is running or not, and displays the most recent log entries. You can use this command to check if the CloudStack management server is running properly.



### Open web browser and type

```
http://<YOUR_IP_ADDRESS>:8080
```

Example:

```
http://139.192.5.123:8080
```
