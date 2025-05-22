
# Tutorial Setting DuckDNS (Dynamic DNS)
DuckDNS merupakan layanan Dynamic DNS (DDNS) yang memungkinkan pengalihan domain ke alamat IP publik, baik statis maupun dinamis. Layanan ini bermanfaat untuk implementasi CloudStack di lingkungan jaringan yang tidak memiliki DNS publik sendiri.

## 1. Akses Situs DuckDNS
* Buka browser dan akses: https://www.duckdns.org
* Login menggunakan akun GitHub / Google / Reddit / Twitter.

## 2. Tambahkan Subdomain DuckDNS
* Pada kolom “sub domain”, masukkan nama yang diinginkan.
* Contoh: cc-kel4 → menjadi cc-kel4.duckdns.org
* Klik tombol `add domain`

## 3. Periksa dan Update IP Address
* Setelah domain berhasil ditambahkan, akan terlihat tampilan seperti berikut:
* Subdomain: cc-kel4.duckdns.org
* Current IP: 139.194.67.212 (otomatis diisi oleh DuckDNS)
* Update IP: Yaitu tombol untuk memperbarui IP ke alamat IP publik saat ini

<img src="https://hackmd.io/_uploads/BkNz1riZxx.jpg" alt="arsitektur" width="300"/>



