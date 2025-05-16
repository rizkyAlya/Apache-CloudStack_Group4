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

(KVM, libvirt, NFS, AppArmor, iptables, dll)

## Arsitektur Sistem
(Diagram sederhana mengenai bagaimana setiap komponen terhubung)

## Referensi

