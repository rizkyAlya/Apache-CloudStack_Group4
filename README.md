[![Logo](https://github.com/user-attachments/assets/30735ef8-5268-4c95-b172-f1b826ecc745)](https://ee.ui.ac.id/)

# Kelompok 4:
* Adam Bintang Arafah Poernomo (2206029273)
* Aliyah Rizky Al-Afifah Polanda (2206024682)
* Naufal Rusyda Santosa (2206813353)
* Raditya Akhila Ganapati (2206026151)
* Reiki Putra Darmawan (2206062882)

# Proyek Komputasi Awan: Implementasi Apache CloudStack
![CloudStack Logo](https://upload.wikimedia.org/wikipedia/commons/7/70/Apache_CloudStack_Logo.svg)

## Tujuan Proyek
1. Membangun infrastruktur komputasi awan yang mudah dikelola menggunakan CloudStack.
2. Mengimplementasikan layanan *virtual machine* (VM) yang dapat diakses secara fleksibel, termasuk akses internet dan SSH dari luar.
3. Mengatur konfigurasi jaringan termasuk Dynamic DNS (DDNS), agar dapat diakses dengan nama domain dinamis dari internet.
4. Menyediakan layanan web server berbasis WordPress di lingkungan VM sebagai contoh aplikasi nyata.

## Link YouTube
[Video Tutorial Instalasi dan Konfigurasi CloudStack](https://youtu.be/s4qJlfPEgig?si=a-OsVXyBi5i9Fssl)

## Definisi
Apache CloudStack merupakan sebuah platform *open-source* yang digunakan untuk membangun layanan IaaS (*Infrastructure as a Service*). Apache CloudStack memungkinkan pembuatan dan pengelolaan lingkungan komputasi awan, seperti layanan *public cloud* dan *private cloud* untuk perusahaan.

## Fitur dan Kemampuan Utama
#### 1. Penyediaan mandiri
Pengguna dapat membuat, mengelola, dan menghentikan VM (*Virtual Machine*) sendiri melalui antarmuka web.
#### 2. Virtualisasi terintegrasi
Mendukung berbagai hypervisor industri standar seperti:
* KVM (*Kernel-based Virtual Machine*)
* XenServer
* VMware vSphere
#### 3. Manajemen terpusat
Semua sumber daya seperti host, penyimpanan, dan jaringan dikendalikan oleh Management Server.
#### 4. Skalabilitas dan elastisitas
CloudStack dapat dikembangkan untuk skala kecil (lab/testing) hingga skala besar (*data center production*).

## Teknologi yang Digunakan
### KVM (*Kernel-based Virtual Machine*)
Merupakan fitur perangkat lunak pada Linux yang memungkinkan mesin fisik untuk menjalankan banyak VM secara efisien. KVM memungkinkan setiap VM berfungsi sebagai komputer independen yang saling berbagi sumber daya dengan mesin fisik, yaitu dengan mengubah sistem Linux menjadi hypervisor bare-metal. KVM mempermudah tugas administrator server dengan menghilangkan kebutuhan untuk mengatur infrastruktur virtualisasi secara manual, sekaligus memungkinkan pengelolaan banyak VM secara efisien dalam lingkungan cloud.

### NFS (*Network File System*)
Merupakan protokol jaringan yang digunakan untuk berbagi file secara terdistribusi, memungkinkan sistem penyimpanan seperti hard disk atau SSD diakses melalui jaringan. Dengan NFS, administrator sistem dapat membagikan seluruh atau sebagian *file system* ke server jaringan, sehingga dapat diakses oleh pengguna komputer jarak jauh yang memiliki otorisasi. Protokol NFS menggunakan *Remote Procedure Calls* (RPC) untuk mengatur komunikasi dan permintaan antara klien dan server, sehingga proses transfer data berjalan efisien.

### iptables
Merupakan _firewall_ standar yang disertakan di sebagian besar distribusi Linux dan berfungsi sebagai antarmuka baris perintah untuk mengelola aturan _firewall_ di tingkat kernel melalui netfilter. Dalam konteks CloudStack, iptables berperan dalam mengamankan lalu lintas jaringan dengan memfilter paket data berdasarkan seperangkat aturan yang telah ditentukan. Aturan-aturan ini menentukan karakteristik paket yang harus sesuai, seperti jenis protokol, alamat sumber atau tujuan, _port_, serta antarmuka jaringan yang digunakan. Dengan ini, iptables memungkinkan administrator CloudStack untuk mengontrol lalu lintas jaringan masuk dan keluar, memastikan bahwa hanya koneksi yang sah yang diizinkan, sehingga memperkuat keamanan infrastruktur _cloud_.

## Arsitektur Sistem
<img src="https://github.com/user-attachments/assets/62652671-4837-4fec-a694-190a61a7f24a" alt="arsitektur" width="400"/>

CloudStack membagi infrastruktur cloud ke dalam beberapa lapisan: `Zone → Pod → Cluster → Host & Primary Storage`
* `Zone`: Mewakili satu lokasi fisik pusat data.
* `Pod`: Grup server dalam satu subnet yang terhubung ke switch layer-2.
* `Cluster`: Sekelompok host dan penyimpanan primer yang memiliki jenis hypervisor yang sama.
* `Host`: Mesin fisik tempat VM berjalan.
* `Primary Storage`: Digunakan untuk menyimpan disk image VM (_root_ dan _data disk_).

Terpisah dari struktur Pod-Cluster, terdapat `Secondary Storage`, yang digunakan untuk menyimpan template VM, ISO, dan snapshot. Dapat digunakan bersama oleh semua pod dan zone, dengan konfigurasi menggunakan protokol seperti NFS. 

# Instalasi CloudStack
## Setup Lingkungan Komputasi
### Spesifikasi Perangkat Keras

```
CPU : Intel Core i5 gen 8
RAM : 24 GB
Penyimpanan : 200GB
Network : Ethernet 10GB/s
Sistem Operasi : Ubuntu Server 24.04
```

### Alamat Jaringan

```
Alamat jaringan : 192.168.1.0/24
Alamat IP Host : 192.168.1.X/24
Gateway : 192.168.1.1
IP public : 139.194.X.X
```


### Konfigurasi Jaringan 

#### Modifikasi Berkas Konfigurasi Jaringan di Direktori /netplan
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
      addresses: [192.168.1.X/24]  # Alamat IP host
      routes:
        - to: default
          via: 192.168.1.1  # Gateway
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
      interfaces: [enp1s0]
      dhcp4: false
      dhcp6: false
      parameters:
        stp: false
        forward-delay: 0
```

#### Terapkan Konfigurasi Jaringan

```
netplan generate        # Menghasilkan berkas konfigurasi untuk renderer
netplan apply           # Menerapkan konfigurasi jaringan ke sistem
reboot                  # Memulai ulang sistem
```

> Catatan: 
* Jika muncul error, pastikan bahwa menggunakan spasi, bukan tab saat memodifikasi berkas konfigurasi jaringan.
* Untuk melihat apakah konfigurasi jaringan sudah aktif, dapat menggunakan perintah ifconfig dan memeriksa antarmuka br0. Alamat IP yang terlihat harus sama dengan yang dikonfigurasi sebelumnya.

#### Uji jaringan

```
ip address        # Memeriksa alamat IP dan antarmuka yang tersedia
ping google.com   # Memastikan bahwa perangkat dapat terhubung ke internet
```

> Catatan:
* Jika tidak dapat ping ke google.com, maka dapat mencoba ping ke gateway dan 8.8.8.8
* Tahapan ini membantu dalam proses identifikasi masalah koneksi antara komputer dan internet. Internet harus berjalan dengan lancar untuk mengunduh beberapa paket di tahapan selanjutnya.

#### Login ke Sistem sebagai Pengguna Root

```
sudo -i
passwd root
```

> Catatan: Setelah menjalankan perintah di atas, sistem akan meminta untuk memasukkan password root baru.

#### Aktifkan Login Root Melalui SSH

```
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
# Restart layanan SSH
service ssh restart
# atau
systemctl restart sshd.service
```


### Instalasi CloudStack (Controller dan Compute Node dalam Satu Host)

#### Impor Kunci Repositori CloudStack

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

#### Install CloudStack dan MySQL Server

```
apt update -y
apt install cloudstack-management mysql-server
```

#### Konfigurasi MySQL

##### Buka berkas konfigurasi MySQL

```
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

##### Tambahkan baris berikut di bahwa bagian [mysqld]

```
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

##### Restart layanan MySQL
```
systemctl restart mysql
```

#### Deploy Database sebagai Root dan Buat Pengguna

```
cloudstack-setup-databases <db_user>:<db_password>@localhost --deploy-as=<root_user>:<root_password>
```

#### Konfigurasi Penyimpanan Primer dan Sekunder

```
apt install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

> Penjelasan:
* Penyimpanan primer digunakan untuk menyimpan disk virtual machine (VM) dan merupakan penyimpanan utama.
* Penyimpanan sekunder digunakan untuk menyimpan template VM, ISO image, dan snapshot.

#### Konfigurasi Server NFS
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

### Konfigurasi Host CloudStack dengan Hypervisor KVM

#### Instalasi KVM dan Agen CloudStack

```
apt install qemu-kvm cloudstack-agent -y
```

#### Konfigurasi Manajemen Virtualisasi KVM

##### Ubah beberapa baris konfigurasi

```
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

# Untuk Ubuntu 22.04 dan setelahnya, tambahkan baris berikut ke /etc/default/libvirtd
sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

> Penjelasan:
* Baris pertama: Mengaktifkan VNC supaya dapat diakses dari semua alamat IP.
* Baris kedua: Mengatur `libvirtd` supaya mendengarkan koneksi TCP dan menyimpan salinan cadangan dari konfigurasi sebelumnya.

##### Add some lines

```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

> Penjelasan: Untuk menonaktifkan TLS dan mdns, serta mengaktifkan akses TCP tanpa autentikasi melalui port 16509, supaya CloudStack dapat terhubung ke libvirt.

##### Restart libvirtd

```
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

> Penjelasan:
* Baris pertama: Digunakan untuk menonaktifkan lima socket default yang biasanya digunakan oleh daemon `libvirtd` untuk menerima koneksi lokal dan remote. Dengan menggunakan `mask`, maka libvirt hanya akan menerima koneksi dari konfigurasi yang ditentukan secara manual.
* Baris kedua: Memulai ulang layanan libvirtd supaya dapat membaca file konfigurasi yang telah dimodifikasi.

#### Konfigurasi untuk Mendukung Docker dan Layanan Lain

```
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

> Penjelasan: Menonaktifkan pemrosesan ARP dan IP oleh `arptables` dan `iptables` untuk menghindari konflik dengan Docker dan sistem jaringan bridge lainnya.

#### Menghasilkan Host UUID Unik

```
apt install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

> Penjelasan: UUID unik digunakan agar setiap host KVM dapat diindentifikasi secara berbeda oleh CloudStack. Untuk instalasi multi-host.

#### Konfigurasi Firewall (iptables)

```
NETWORK=192.168.1.0/24
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

#### Instalasi CloudStack Agent

```
systemctl unmask cloudstack-agent
apt update -y
apt install cloudstack-agent -y
```

> Penjelasan: Agen ini memungkinkan host KVM dikontrol oleh server manajemen CloudStack.

#### Nonaktifkan AppArmor untuk Libvirt

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

#### Menjalankan CloudStack Management Server

```
cloudstack-setup-management
systemctl status cloudstack-management
```

> Penjelasan: Langkah ini menyelesaiakan instalasi dan konfigurasi awal CloudStack, serta memulai layanan manajemen.
* `cloudstack-setup-management`
Untuk menginisialisasi dan mengonfigurasi server manajemen Apache CloudStack.
* `systemctl status cloudstack-management`
Untuk menampilkan status terkini dari layanan CloudStack Management Server, dan juga menunjukkan log terbaru yang dapat membantu proses troubleshooting jika terjadi kegagalan.

#### Akses Antarmuka Web

```
http://<IP_Address>:8080
```

> Penjelasan: Gunakan browser untuk membuka antarmuka web CloudStack dan lanjutkan ke konfigurasi melalui GUI.

# Manajemen Infrastruktur Cloud melalui Dashboard CloudStack

> Catatan Penting  
Pada konfigurasi ini, host KVM dan server NFS berada pada perangkat yang sama. Artinya IP address untuk Host, Primary Storage, dan Secondary Storage adalah sama. 

## Login dan Overview GUI
Halaman pertama yang ditampilkan setelah mengakses antarmuka web CloudStack adalah halaman login. Untuk login pertama kali, gunakan akun default admin yang dibuat secara otomatis saat setup database. 
```
username: admin
password: password
```
> Catatan: Field `Domain` dapat dikosongkan

Berikut adalah tampilan awal dari web CloudStack:  
<img src="https://github.com/user-attachments/assets/37893420-a7cb-459b-a97d-b5b5a3be72d7" alt="dashboard" width="650" height="400"/>

> Catatan: Setelah login, password dapat langsung diubah

## Konfigurasi Zone
Zone merupakan unit geografis yang mencakup kumpulan server, storage, dan jaringan yang bekerja sama untuk menyediakan layanan cloud. Konfigurasi ini bertujuan untuk mendefinisikan arsitektur dan ketersediaan sumber daya cloud. 

Konfigurasi zone dilakukan di awal, setelah memilih opsi **Continue with installation**.

### 1. Zone type
Terdapat dua jenis zone yang menentukan arsitektur jaringan dan cara distribusi layanan di zone tersebut, yaitu:
* Core Zone: Digunakan di data center pusat. Lebih fleksibel dan redundan.
* Edge Zone: Digunakan di remote/branch site. Lebih ringan dan minim konfigurasi.

Pilih Core Zone karena sedang membangun infrastruktur utama.  
<img src="https://github.com/user-attachments/assets/a0850f4d-561f-4e0d-a99e-871123f1e20b" alt="core_zone" width="600" height="400"/>

### 2. Core zone type
Terdapat dua jenis core zone yang dapat dipilih, yaitu:
* Advanced: VM dapat berada di jaringan virtual terpisah dan menyediakan layanan jaringan yang canggih. Digunakan saat ingin deploy banyak user atau menjalankan skenario enterprise.
* Basic: VM berada dalam satu jaringan. Digunakan saat ingin setup yang sederhana dan cepat, serta sesuai untuk topologi kecil.

Pilih Basic karena tidak membutuhkan layanan jaringan yang canggih dan ingin deploy VM yang cepat untuk testing.  
<img src="https://github.com/user-attachments/assets/a7fa98e9-eee8-4b25-9393-b39e340403a2" alt="basic_core_zone" width="600" height="400"/>

### 3. Zone details
Isi field yang wajib dengan ketentuan sebagai berikut:
* `Name`: Yaitu nama untuk zone yang sedang dibuat. Bebas namun deskriptif.
* `IPv4 DNS1`: Yaitu alamat IP DNS publik utama untuk VM yang akan dibuat di zone ini. Biasanya diisi dengan IP DNS yang valid seperti Google DNS (8.8.8.8).
* `Internal DNS 1`: Yaitu DNS internal yang digunakan oleh CloudStack untuk manajemen (bukan untuk VM).
* `Hypervisor`: Yaitu jenis hypervisor yang digunakan di zone ini. Menggunakan KVM.

```
Name: NEW-ZONE
IPv4 DNS1: 8.8.8.8
Internal DNS 1: 192.168.1.1
Hypervisor: KVM
```

> Catatan: Field IPv4 DNS2, Internal DNS 2, Network offering, dan Default network domain for Isolated networks dapat dikosongkan.

### 4. Network
**Physical Network**  
Pada jenis Basic Zone, hanya terdapat satu physical network yang digunakan dan seluruh VM akan berbagi jaringan yang sama (shared network). Network ini terkait langsung dengan NIC (interface jaringan) pada hypervisor host, dan membawa berbagai jenis trafik, seperti:
* Management Traffic
* Guest Traffic
* Storage Traffic

<img src="https://github.com/user-attachments/assets/d6e2d382-4c3d-482c-80c1-056d779eac60" alt="physical_network" width="600" height="400"/>

> Catatan: Pada proses setup Basic Zone, tidak ada konfigurasi khusus yang perlu diubah di bagian ini. Cukup klik **Next** untuk melanjutkan ke konfigurasi Pod.

**Pod**  
Pod merepresentasikan sekumpulan host dalam satu subnet Layer 2.  
Isi field yang wajib dengan ketentuan sebagai berikut:
* `Pod Name`: Nama untuk pod yang sedang dibuat. Bebas, namun sebaiknya bersifat deskriptif agar mudah dikenali.
* `Reserved System Gateway`: Alamat IP gateway dari subnet yang digunakan oleh pod ini. IP ini harus sesuai dengan jaringan fisik tempat host berada.
* `Reserved System Netmask`: Netmask dari subnet yang digunakan untuk sistem internal CloudStack. 
* `Start Reserved System IP`: Alamat IP pertama dalam rentang IP yang dicadangkan untuk sistem internal CloudStack.
* `End Reserved System IP`: Alamat IP terakhir dalam rentang IP yang dicadangkan untuk sistem internal.

```
Pod Name: NEW-POD
Reserved System Gateway: 192.168.1.1
Reserved System Netmask: 255.255.255.0
Start Reserved System IP: 192.168.1.Y
End Reserved System IP: 192.168.1.Z
```

**Guest Network**  
Pada Basic Zone, semua guest VM menggunakan shared network dan mendapatkan IP dari subnet yang sama dengan physical network.

```
Guest Gateway: 192.168.1.1
Guest Netmask: 255.255.255.0
Start IP: 192.168.1.A
End IP: 192.168.1.B
```

> Catatan: Rentang IP ini akan digunakan oleh CloudStack untuk provisioning IP ke VM (guest). Pastikan rentang ini tidak tumpang tindih dengan IP sistem internal yang digunakan oleh Pod atau perangkat lain.


### 5. Add resources
**Cluster**  
Cluster merupakan sekumpulan host yang berbagi storage dan menjalankan hypervisor yang sama. Field yang harus diisi pada bagian ini hanya nama cluster.

```
Cluster Name: NEW-CLUSTER
```

**IP address**  
Bagian ini mendefinisikan host, yaitu server fisik yang akan menjalankan VM di dalam cluster.  
Isi field yang wajib dengan ketentuan sebagai berikut:
* `Hostname`: IP address dari host (server) KVM.
* `Username`: Username login ke host, biasanya `root.
* `Authentication Method`: Pilih antara **password** atau **system SSH key**.
* `Password`: Jika memilih password, maka isikan password untuk login ke host.

```
Hostname: 192.168.1.X
Username: root
Authentication Method: Password
Password: ********
```

**Primary storage**  
Primary storage digunakan untuk menyimpan disk VM, template lokal, dan snapshot. Storage ini harus bisa diakses oleh semua host dalam cluster.
Isi field yang wajib dengan ketentuan sebagai berikut:
* `Name`: Nama untuk primary storage. Bebas, namun sebaiknya bersifat deskriptif agar mudah dikenali.
* `Scope`: Lingkup penggunaan dari primary storage (cluster/zone).
* `Protocol`: Protokol manajemen storage yang digunakan (biasanya NFS).
* `Server`: IP dari storage server.
* `Path`:  Path direktori NFS yang diekspor.

```
Name: NEW-PRISTORE
Scope: Zone
Protocol: NFS
Server: 192.168.1.X 
Path: /export/primary
Provider: DefaultPrimary
```

> Catatan: Field Provider tidak perlu diubah, karena terisi secara otomatis sesuai protokol yang digunakan.

**Secondary storage**  
Secondary storage digunakan untuk menyimpan ISO, template VM, dan snapshot backup. 

```
Provider: NFS
Name: NEW-SECSTORE
Server: 192.168.1.X 
Path: /export/secondary
```

### 6. Launch zone
Pada tahap akhir ini, semua komponen yang telah dikonfigurasi akan dikonfirmasi dan diaktifkan agar zone dapat digunakan untuk menjalankan VM. Jika semua konfigurasi valid, maka zone sudah aktif dan dapat digunakan.  
Tampilan jika konfigurasi berhasil:  
<img src="https://github.com/user-attachments/assets/a1d081a6-f8be-483f-b973-f9b7ed9ad0a7" alt="dashboard" width="800" height="350"/>


## Pembuatan Instance (VM)
### 1. Register ISO
Pada bagian ini akan dilakukan pendaftaran file ISO yang digunakan sebagai sistem operasi VM.

**Akses menu:**
`Images` -> `ISO` -> `Register ISO`

**Field yang wajib diisi:**
* `URL`: Berisi link ke file .iso yang akan digunakan.
* `Name`: Nama untuk ISO. Bebas, namun sebaiknya bersifat deskriptif agar mudah dikenali.
* `Zone`: Zone tempat ISO ini tersedia.
* `OS Type`: Sistem operasi yang sesuai dengan ISO yang digunakan.

```
URL: https://saimei.ftp.acc.umu.se/debian-cd/current/amd64/iso-cd/debian-12.10.0-amd64-netinst.iso
Name: Debian
Zone: NEW-ZONE
OS Type: Debian GNU/Linux 12 (64-bit)
```

> Catatan: Field lain dapat dikosongkan atau mengikuti pilihan default.

<img src="https://github.com/user-attachments/assets/22fc6a3d-04bd-425a-94b0-c26bcb0ae88f" alt="ISO" width="600" height="200"/>

### 2. Compute offering
Bagian ini menentukan alokasi CPU dan memori untuk VM yang akan dibuat.

**Akses menu:**
`Service Offerings` -> `Compute Offerings` -> `Add Compute Offering`

**Field yang wajib diisi:**
* `Name`: Nama untuk offering. Bebas, namun sebaiknya bersifat deskriptif agar mudah dikenali.
* `Compute Offering Type`:  Yaitu jenis penawaran sumber daya, ada 3 opsi yang dapat dipilih:  
  - Fixed Offering –> berupa nilai tetap  
  - Custom Constrained –> dapat diatur oleh user tapi pengaturannya dibatasi  
  - Custom Unconstrained –> dapat diatur bebas oleh user 
* `CPU Cores`: Jumlah core CPU yang akan dialokasikan
* `CPU (in MHz)`: Kecepatan tiap core CPU.
* `Memory (in MB)`: Jumlah memori RAM dalam MB.

```
Name: Big Instance
Compute Offering Type: Fixed offering
CPU Cores: 2
CPU (in MHz): 1500
Memory (in MB): 4096
```

> Catatan: Field lain dapat dikosongkan atau mengikuti pilihan default.

<img src="https://github.com/user-attachments/assets/32b08348-c6ba-48a8-80ce-9978f0250012" alt="Offering" width="600" height="200"/>

### 3. Instance
Langkah terakhir adalah membuat dan menjalankan VM menggunakan ISO dan compute offering yang telah didaftarkan.

**Akses menu:**
`Compute` → `Instances` → `Add Instace`

**Field yang wajib diisi:**
* `Template/ISO`: Gunakan ISO yang telah didaftarkan sebelumnya.
* `Compute Offering`: Gunakan offering yang telah dibuat.
* `Disk Size`: Memilih ukuran hard disk untuk VM.

```
Template/ISO: Debian
Compute Offering: Big Instance
Disk Size: Medium
```

**Tahapan akhir:**
* Klik `Launch Instance`
* VM akan berada di status `Starting`, dan saat status berubah menjadi `Running`, klik tombol `View Console` untuk melihat tampilan layar VM.

<img src="https://github.com/user-attachments/assets/d7f29153-4477-47df-b701-542583da9950" alt="VM" width="600" height="200"/>

# Konfigurasi Tambahan
## Tutorial Setting DuckDNS (Dynamic DNS)
DuckDNS merupakan layanan Dynamic DNS (DDNS) yang memungkinkan pengalihan domain ke alamat IP publik, baik statis maupun dinamis. Layanan ini bermanfaat untuk implementasi CloudStack di lingkungan jaringan yang tidak memiliki DNS publik sendiri.

### 1. Akses Situs DuckDNS
* Buka browser dan akses: https://www.duckdns.org
* Login menggunakan akun GitHub / Google / Reddit / Twitter.

### 2. Tambahkan Subdomain DuckDNS
* Pada kolom “sub domain”, masukkan nama yang diinginkan.
* Contoh: cc-kel4 → menjadi cc-kel4.duckdns.org
* Klik tombol `add domain`

### 3. Periksa dan Update IP Address
* Setelah domain berhasil ditambahkan, akan terlihat tampilan seperti berikut:
* Subdomain: cc-kel4.duckdns.org
* Current IP: 139.194.67.212 (otomatis diisi oleh DuckDNS)
* Update IP: Yaitu tombol untuk memperbarui IP ke alamat IP publik saat ini

<img src="https://hackmd.io/_uploads/BkNz1riZxx.jpg" alt="arsitektur" width="300"/>


## Tutorial Menyediakan HTTP/HTTPS Server Menggunakan DuckDNS + Port Forwarding (Tanpa Reverse Proxy/SSL Otomatis)
Langkah di bawah ini dilakukan setelah mengatur DuckDNS.

### 1. Port Forwarding di Router
* Masuk ke halaman admin router (biasanya `192.168.1.1`).
* Forward port 80 (HTTP) ke IP lokal server Apache.
  Contoh:
  ```
  External Port: 80
  Internal IP: 192.168.1.13
  Internal Port: 8080
  Protocol: TCP
  ```
    <img src="https://i.imgur.com/ifm2S7a.png" alt="Konfigurasi Router" width="300"/>

### 2. Jalankan Cloudstack Server

```bash
sudo apt update
sudo apt install cloudstack-management
sudo systemctl start cloudstack-management
sudo systemctl enable cloudstack-management
```

Periksa apakah halaman default Apache muncul dengan:
```
http://localhost:8080
```

### 3. Akses Melalui Domain

Buka:
```
http://cc-kel4.duckdns.org
```

Jika port forwarding benar dan Apache aktif, halaman Apache akan muncul.


## Referensi
* "What is Apache CloudStack?" Apache Cloudstack. [Online]. Available: https://docs.cloudstack.apache.org/en/4.11.1.0/conceptsandterminology/concepts.html. [Accessed: May 14, 2025].
* "Apa Itu KVM (Mesin Virtual Berbasis Kernel)?" AWS. [Online]. Available: https://aws.amazon.com/id/what-is/kvm/. [Accessed: May 17, 2025].
* P. Loshin, "Network File System (NFS)" TechTarget. [Online]. Available: https://www.techtarget.com/searchenterprisedesktop/definition/Network-File-System. [Accessed: May 17, 2025].
* J. Ellingwood, "How the Iptables Firewall Works," Dec. 2022. [Online]. Available: https://www.digitalocean.com/community/tutorials/how-the-iptables-firewall-works. [Accessed: May 19, 2025].
* AhmadRifqi86, "cloudstack-install-and-configure," GitHub. [Online]. Available: https://github.com/AhmadRifqi86/cloudstack-install-and-configure/tree/main/cloudstack-install. [Accessed: May 7, 2025].

