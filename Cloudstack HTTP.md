
# Tutorial Menyediakan HTTP/HTTPS Server Menggunakan DuckDNS + Port Forwarding (Tanpa Reverse Proxy/SSL Otomatis)
Langkah di bawah ini dilakukan setelah mengatur DuckDNS.

## 1. Port Forwarding di Router
* Masuk ke halaman admin router (biasanya `192.168.1.1`).
* Forward port 80 (HTTP) ke IP lokal server Apache.
  Contoh:
  ```
  External Port: 80
  Internal IP: 192.168.1.13
  Internal Port: 8080
  Protocol: TCP
  ```
![](https://i.imgur.com/ifm2S7a.png)

## 2. Jalankan Cloudstack Server

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

## 3. Akses Melalui Domain

Buka:
```
http://cc-kel4.duckdns.org
```

Jika port forwarding benar dan Apache aktif, halaman Apache akan muncul.
