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

