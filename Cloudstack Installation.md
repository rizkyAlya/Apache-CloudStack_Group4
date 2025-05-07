# Setup Lingkungan Komputasi

## Spesifikasi Perangkat Keras

```
CPU : Intel Core i5 gen 8
RAM : 24 GB
Penyimpanan : 200GB
Network : Ethernet 10GB/s
Sistem Operasi : Ubuntu Server 24.04
```

## Alamat Jaringan

```
Alamat jaringan : 192.168.1.0/24
Alamat IP Host : 192.168.1.13/24
Gateway : 192.168.1.1
IP public : 139.192.5.123
```


## Konfigurasi Jaringan 

### Modifikasi Berkas Konfigurasi Jaringan di Direktori /netplan
Masuk sebagai root dan buka direktori konfigurasi jaringan dengan perintah berikut:

```
sudo -i 
cd /etc/netplan
nano ./*.yaml
```

Kemudian, modifikasi isi berkas konfigurasi menjadi seperti berikut:

```
# Ditulis oleh 'subiquity'
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0: # Adapter ethernet
      dhcp4: false
      dhcp6: false
      optional: true
  bridges:
    cloudbr0:
      addresses: [192.168.1.13/24]  # Alamat IP host
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

### Terapkan Konfigurasi Jaringan

```
netplan generate        # Menghasilkan berkas konfigurasi untuk renderer
netplan apply           # Menerapkan konfigurasi jaringan ke sistem
reboot                  # Memulai ulang sistem
```

> Catatan: Jika muncul error, pastikan bahwa menggunakan spasi, bukan tab saat memodifikasi berkas konfigurasi jaringan.
> Catatan: Untuk melihat apakah konfigurasi jaringan sudah aktif, dapat menggunakan perintah ifconfig dan memeriksa antarmuka br0. Alamat IP yang terlihat harus sama dengan yang dikonfigurasi sebelumnya.

### Uji jaringan

```
ip address        # Memeriksa alamat IP dan antarmuka yang tersedia
ping google.com   # Memastikan bahwa perangkat dapat terhubung ke internet
```

> Catatan: Jika tidak dapat ping ke google.com, maka dapat mencoba ping ke gateway dan 8.8.8.8
> Catatan: Tahapan ini membantu dalam proses identifikasi masalah koneksi antara komputer dan internet. Internet harus berjalan dengan lancar untuk mengunduh beberapa paket di tahapan selanjutnya.

### Login ke Sistem sebagai Pengguna Root

```
sudo -i
passwd root
```

> Catatan: Setelah menjalankan perintah di atas, sistem akan meminta untuk memasukkan password root baru.

### Aktifkan Login Root Melalui SSH

```
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
# Restart layanan SSH
service ssh restart
# atau
systemctl restart sshd.service
```


## Instalasi CloudStack (Controller dan Compute Node dalam Satu Host)

### Impor Kunci Repositori CloudStack

```
mkdir -p /etc/apt/keyrings 
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.20 / > /etc/apt/sources.list.d/cloudstack.list
```

> Penjelasan:
* Baris pertama digunakan untuk membuat direktori baru tempat menyimpan kunci publik CloudStack.
* `wget -O-` digunakan untuk mengunduh URL yang diberikan dan mengalirkan output ke perintah `gpg --dearmor`.
* Perintah `gpg --dearmor` digunakan untuk mengubah format ASCII ke format biner.
* Perintah `sudo tee` mengalihkan hasil dari `gpg --dearmor` ke berkas `/etc/apt/keyrings/cloudstack.gpg`.

### Install CloudStack dan MySQL Server

```
apt update -y
apt install cloudstack-management mysql-server
```

### Konfigurasi MySQL

#### Buka berkas konfigurasi MySQL

```
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

#### Tambahkan baris berikut di bahwa bagian [mysqld]

```
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

#### Restart layanan MySQL
```
systemctl restart mysql
```

### Deploy Database sebagai Root dan Buat Pengguna "cloud" dengan Kata Sandi "cloud

```
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:Pa$$w0rd -i 192.168.1.13
```

### Konfigurasi Penyimpanan Primer dan Sekunder

```
apt install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

> Catatan:
* Penyimpanan primer digunakan untuk menyimpan disk virtual machine (VM) dan merupakan penyimpanan utama.
* Penyimpanan sekunder digunakan untuk menyimpan template VM, ISO image, dan snapshot.

### Konfigurasi Server NFS
NFS digunakan sebagai shared storage untuk penyimpanan primer dan sekunder, supaya bisa diakses oleh banyak host secara bersamaan.

```
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```

>Penjelasan:
* Baris pertama: Mengubah baris `RPCMOUNTDOPTS="--manage-gids"` pada berkas `/etc/default/nfs-kernel-server` menjadi `RPCMOUNTDOPTS="-p 892 --manage-gids"` menggunakan sed.

* Baris kedua: Mengubah baris `STATDOPTS=` pada berkas `/etc/default/nfs-common` menjadi `STATDOPTS="--port 662 --outgoing-port 2020"`.

* Baris ketiga: Menambahkan baris `NEED_STATD=yes` ke akhir berkas `/etc/default/nfs-common`.

* Baris keempat: Mengubah baris `RPCRQUOTADOPTS=` pada berkas `/etc/default/quota` menjadi `RPCRQUOTADOPTS="-p 875"`.

* Baris kelima: Memulai ulang layanan NFS agar semua perubahan konfigurasi diterapkan

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
