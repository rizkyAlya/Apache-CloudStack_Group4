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

## Definisi
Apache CloudStack merupakan sebuah platform *open-source* yang digunakan untuk membangun layanan IaaS (*Infrastructure as a Service*). Apache CloudStack memungkinkan pembuatan dan pengelolaan lingkungan komputasi awan, seperti layanan *public cloud* dan *private cloud* untuk perusahaan.

## Tahapan Pengerjaan
1. Instalasi CloudStack pada server mencakup konfigurasi jaringan, instalasi CloudStack Management Server dan KVM Hypervisor dalam satu host. (link : https://github.com/rizkyAlya/Apache-CloudStack_Group4/blob/main/Cloudstack%20Installation.md)
2. Setup Basic Zone, pengaturan jaringan dan resource (cluster, host, storage), lalu dilanjutkan dengan registrasi ISO, pembuatan compute offering, dan peluncuran VM (instance) pada UI Cloudstack. (link : https://github.com/rizkyAlya/Apache-CloudStack_Group4/blob/main/Cloudstack%20Configuration.md)
3. Membuat ddns pada server sehingga server bisa diakses menggunakan domain name. (link : https://github.com/rizkyAlya/Apache-CloudStack_Group4/blob/main/Cloudstack%20DNS.md)
4. Setelah domain DuckDNS siap, lakukan port forwarding di router agar permintaan ke port 80 diteruskan ke server lokal (Apache port 8080). (link : https://github.com/rizkyAlya/Apache-CloudStack_Group4/blob/main/Cloudstack%20HTTP.md)

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

## Referensi
* "What is Apache CloudStack?" Apache Cloudstack. [Online]. Available: https://docs.cloudstack.apache.org/en/4.11.1.0/conceptsandterminology/concepts.html. [Accessed: May 14, 2025].
* "Apa Itu KVM (Mesin Virtual Berbasis Kernel)?" AWS. [Online]. Available: https://aws.amazon.com/id/what-is/kvm/. [Accessed: May 17, 2025].
* P. Loshin, "Network File System (NFS)" TechTarget. [Online]. Available: https://www.techtarget.com/searchenterprisedesktop/definition/Network-File-System. [Accessed: May 17, 2025].
* J. Ellingwood, "How the Iptables Firewall Works," Dec. 2022. [Online]. Available: https://www.digitalocean.com/community/tutorials/how-the-iptables-firewall-works. [Accessed: May 19, 2025].
