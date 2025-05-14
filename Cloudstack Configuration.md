# Manajemen Infrastruktur Cloud melalui Dashboard CloudStack

## Login dan Overview GUI
Halaman pertama yang ditampilkan setelah mengakses antarmuka web CloudStack adalah halaman login. Untuk login pertama kali, gunakan akun default admin yang dibuat secara otomatis saat setup database. 
```
username: admin
password: password
```
> Catatan: Field `Domain` dapat dikosongkan

Berikut adalah tampilan awal dari web CloudStack:
![{ABB1069A-3E75-458E-BC1D-AB773FF76C26}](https://github.com/user-attachments/assets/37893420-a7cb-459b-a97d-b5b5a3be72d7)

> Catatan: Setelah login, password dapat langsung diubah

## Konfigurasi Zone
Zone merupakan unit geografis yang mencakup kumpulan server, storage, dan jaringan yang bekerja sama untuk menyediakan layanan cloud. Konfigurasi ini bertujuan untuk mendefinisikan arsitektur dan ketersediaan sumber daya cloud. 

Konfigurasi zone dilakukan di awal, setelah memilih opsi `Continue with installation`

### 1. Zone type
Terdapat dua jenis zone yang menentukan arsitektur jaringan dan cara distribusi layanan di zone tersebut, yaitu:
* Core Zone: Digunakan di data center pusat. Lebih fleksibel dan redundan.
* Edge Zone: Digunakan di remote/branch site. Lebih ringan dan minim konfigurasi.

Pilih Core Zone karena sedang membangun infrastruktur utama.
![{A3D29032-A19F-4595-AE9E-959E3D9292B4}](https://github.com/user-attachments/assets/a0850f4d-561f-4e0d-a99e-871123f1e20b)

### 2. Core zone type
Terdapat dua jenis core zone yang dapat dipilih, yaitu:
* Advanced: VM dapat berada di jaringan virtual terpisah dan menyediakan layanan jaringan yang canggih. Digunakan saat ingin deploy banyak user atau menjalankan skenario enterprise.
* Basic: VM berada dalam satu jaringan. Digunakan saat ingin setup yang sederhana dan cepat, serta sesuai untuk topologi kecil.

Pilih Basic karena tidak membutuhkan layanan jaringan yang canggih dan ingin deploy VM yang cepat untuk testing.
![{57C9A91D-702D-4ACF-B282-48B142D31A3D}](https://github.com/user-attachments/assets/a7fa98e9-eee8-4b25-9393-b39e340403a2)

### 3. Zone details
Isi field yang wajib dengan ketentuan sebagai berikut:
* Name: Yaitu nama untuk zone yang sedang dibuat. Bebas namun deskriptif.
* IPv4 DNS1: Yaitu alamat IP DNS publik utama untuk VM yang akan dibuat di zone ini. Biasanya diisi dengan IP DNS yang valid seperti Google DNS (8.8.8.8).
* Internal DNS 1: Yaitu DNS internal yang digunakan oleh CloudStack untuk manajemen (bukan untuk VM).
* Hypervisor: Yaitu jenis hypervisor yang digunakan di zone ini. Menggunakan KVM.

```
Name: NEW-ZONE
IPv4 DNS1: 8.8.8.8
Internal DNS 1: [ALAMAT_GATEWAY] 
Hypervisor: KVM
```

> Catatan: Field IPv4 DNS2, Internal DNS 2, Network offering, dan Default network domain for Isolated networks dapat dikosongkan.

### 4. Network
Physical Network
![{FDD4BA80-7966-4800-8EBB-B0DC540167D9}](https://github.com/user-attachments/assets/d6e2d382-4c3d-482c-80c1-056d779eac60)
Pod
![{E5A6A50D-7319-4624-A2C6-8459901951A9}](https://github.com/user-attachments/assets/ffb4cabb-a734-4758-b602-9057ecf888f9)
Guest traffic
![{CEC51EA5-E4F9-43A1-8262-DFD1FCFDAD06}](https://github.com/user-attachments/assets/24340c04-cf3a-48c8-8002-902c12f7d4c9)

Add resources
Cluster
IP address
Primary storage
Secondary storage

