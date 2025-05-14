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
Alamat jaringan : 192.168.X.X/24
Alamat IP Host : 192.168.X.X/24
Gateway : 192.168.X.X
IP public : 139.194.X.X
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
      addresses: [192.168.X.X/24]  # Alamat IP host
      routes:
        - to: default
          via: 192.168.X.X  # Gateway
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

> Catatan: 
* Jika muncul error, pastikan bahwa menggunakan spasi, bukan tab saat memodifikasi berkas konfigurasi jaringan.
* Untuk melihat apakah konfigurasi jaringan sudah aktif, dapat menggunakan perintah ifconfig dan memeriksa antarmuka br0. Alamat IP yang terlihat harus sama dengan yang dikonfigurasi sebelumnya.

### Uji jaringan

```
ip address        # Memeriksa alamat IP dan antarmuka yang tersedia
ping google.com   # Memastikan bahwa perangkat dapat terhubung ke internet
```

> Catatan:
* Jika tidak dapat ping ke google.com, maka dapat mencoba ping ke gateway dan 8.8.8.8
* Tahapan ini membantu dalam proses identifikasi masalah koneksi antara komputer dan internet. Internet harus berjalan dengan lancar untuk mengunduh beberapa paket di tahapan selanjutnya.

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

### Deploy Database sebagai Root dan Buat Pengguna

```
cloudstack-setup-databases <db_user>:<db_password>@localhost --deploy-as=<root_user>:<root_password>
```

### Konfigurasi Penyimpanan Primer dan Sekunder

```
apt install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

> Penjelasan:
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

> Penjelasan:
* Baris pertama: Mengubah baris `RPCMOUNTDOPTS="--manage-gids"` pada berkas `/etc/default/nfs-kernel-server` menjadi `RPCMOUNTDOPTS="-p 892 --manage-gids"` menggunakan sed.

* Baris kedua: Mengubah baris `STATDOPTS=` pada berkas `/etc/default/nfs-common` menjadi `STATDOPTS="--port 662 --outgoing-port 2020"`.

* Baris ketiga: Menambahkan baris `NEED_STATD=yes` ke akhir berkas `/etc/default/nfs-common`.

* Baris keempat: Mengubah baris `RPCRQUOTADOPTS=` pada berkas `/etc/default/quota` menjadi `RPCRQUOTADOPTS="-p 875"`.

* Baris kelima: Memulai ulang layanan NFS agar semua perubahan konfigurasi diterapkan

## Konfigurasi Host CloudStack dengan Hypervisor KVM

### Instalasi KVM dan Agen CloudStack

```
apt install qemu-kvm cloudstack-agent -y
```

### Konfigurasi Manajemen Virtualisasi KVM

#### Ubah beberapa baris konfigurasi

```
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

# Untuk Ubuntu 22.04 dan setelahnya, tambahkan baris berikut ke /etc/default/libvirtd
sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

> Penjelasan:
* Baris pertama: Mengaktifkan VNC supaya dapat diakses dari semua alamat IP.
* Baris kedua: Mengatur `libvirtd` supaya mendengarkan koneksi TCP dan menyimpan salinan cadangan dari konfigurasi sebelumnya.

#### Add some lines

```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

> Penjelasan: Untuk menonaktifkan TLS dan mdns, serta mengaktifkan akses TCP tanpa autentikasi melalui port 16509, supaya CloudStack dapat terhubung ke libvirt.

#### Restart libvirtd

```
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

> Penjelasan:
* Baris pertama: Digunakan untuk menonaktifkan lima socket default yang biasanya digunakan oleh daemon `libvirtd` untuk menerima koneksi lokal dan remote. Dengan menggunakan `mask`, maka libvirt hanya akan menerima koneksi dari konfigurasi yang ditentukan secara manual.
* Baris kedua: Memulai ulang layanan libvirtd supaya dapat membaca file konfigurasi yang telah dimodifikasi.

### Konfigurasi untuk Mendukung Docker dan Layanan Lain

```
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

> Penjelasan: Menonaktifkan pemrosesan ARP dan IP oleh `arptables` dan `iptables` untuk menghindari konflik dengan Docker dan sistem jaringan bridge lainnya.

### Menghasilkan Host UUID Unik

```
apt install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

> Penjelasan: UUID unik digunakan agar setiap host KVM dapat diindentifikasi secara berbeda oleh CloudStack. Untuk instalasi multi-host.

### Konfigurasi Firewall (iptables)

```
NETWORK=192.168.X.X/24
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT     # Portmap service (needed for NFS)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT     # Portmap service (needed for NFS)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT    # NFS server port
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT   # NFS mountd (dynamic port)
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT   # NFS nlockmgr
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT     # NFS rpc.mountd
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT     # NFS rquotad (quota)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT     # NFS statd
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8250 -j ACCEPT    # CloudStack Agent
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8080 -j ACCEPT    # CloudStack Management Server
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8443 -j ACCEPT    # CloudStack Management Server
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 9090 -j ACCEPT    # CloudStack Console Proxy (VM console)	
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 16514 -j ACCEPT   # libvirt (KVM communication)

apt install iptables-persistent
```
> Penjelasan: Firewall ini memastikan semua port penting untuk CloudStack, NFS, dan KVM dapat diakses dari jaringan lokal.
* `-A input`: Menambahkan aturan ke Input chain, yaitu jalur untuk paket data yang masuk ke sistem.
* `-s $NETWORK`: Menentukan sumber paket. Variabel `$NETWORK` berisi alamat jaringan.
* `-m state`: Menggunakan modul `state` untuk mencocokkan status koneksi. `NEW' berarti aturan hanya berlaku untuk koneksi yang baru dimulai.
* `-p udp/tcp --dport [PORT NUMBER]`: Menetapkan protokol (TCP dan UDP) serta nomor port tujuan yang akan diizinkan.
* `-j ACCEPT`: Menentukan tindakan terhadap paket yang cocok dengan aturan ini, yaitu mengizinkan paket untuk masuk. 

### Instalasi CloudStack Agent

```
systemctl unmask cloudstack-agent
apt update -y
apt install cloudstack-agent -y
```

> Penjelasan: Agen ini memungkinkan host KVM dikontrol oleh server manajemen CloudStack.

### Nonaktifkan AppArmor untuk Libvirt

```
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

> Penjelasan: Langkah ini menyelesaikan instalasi dan konfigurasi awal CloudStack serta memulai layanan manajemen.
* `ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/`
Untuk membuat symlink (tautan simbolik) ke direktori `disable/` agar AppArmor tidak lagi memuat profil `usr.sbin.libvirtd`. Merupakan cara standar untuk menonaktifkan profil tertentu tanpa menghapus file aslinya.
* `ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/`
Sama seperti perintah sebelumnya, namun yang ini untuk profil `virt-aa-helper`, yaitu helper bawaan libvirt untuk pengelolaan kebijakan keamanan VM.
* `apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd`
Untuk menghapus profil `usr.sbin.libvirtd` dari kernel secara langsung, sehingga perubahan dapat berlaku tanpa reboot.
* `apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper`
Untuk menghapus profil `virt-aa-helper` dari kernel agar tidak aktif saat layanan berjalan.

### Menjalankan CloudStack Management Server

```
cloudstack-setup-management
systemctl status cloudstack-management
```

> Penjelasan: Langkah ini menyelesaiakan instalasi dan konfigurasi awal CloudStack, serta memulai layanan manajemen.
* `cloudstack-setup-management`
Untuk menginisialisasi dan mengonfigurasi server manajemen Apache CloudStack.
* `systemctl status cloudstack-management`
Untuk menampilkan status terkini dari layanan CloudStack Management Server, dan juga menunjukkan log terbaru yang dapat membantu proses troubleshooting jika terjadi kegagalan.

### Akses Antarmuka Web

```
http://<IP_Address>:8080
```

> Penjelasan: Gunakan browser untuk membuka antarmuka web CloudStack dan lanjutkan ke konfigurasi melalui GUI.
