---
title: Cloudstack HTTP

---



---

##  Cara Menyediakan HTTP/HTTPS Server Pakai DuckDNS + Port Forwarding (Tanpa Reverse Proxy/SSL Otomatis)




---

### 🧱 **Langkah 1: Setup DuckDNS**

1. Buka [https://www.duckdns.org](https://www.duckdns.org)
2. Login pakai GitHub / Google.
3. Tambahkan subdomain, contoh: `cc-kel4.duckdns.org`
4. Dapatkan **token DuckDNS** untuk update IP otomatis (opsional tapi disarankan)

---

### 🌐 **Langkah 2: Port Forwarding di Router**

Masuk ke halaman admin router kamu (biasanya `192.168.1.1` atau `192.168.100.1`) dan:

* Forward port 80 (HTTP) ke IP lokal server Apache.
  Contoh:

  ```
  External Port: 80
  Internal IP: 192.168.1.100
  Internal Port: 8080
  Protocol: TCP
  ```

---

### ⚙️ **Langkah 3: Jalankan Cloudstack Server**

```bash
sudo apt update
sudo apt install cloudstack-management
sudo systemctl start cloudstack-management
sudo systemctl enable cloudstack-management

```

Cek apakah halaman default Apache muncul dengan:

```
http://localhost:8080

```

---



### ✅ **Langkah 4: Akses Lewat Domain**

Buka:

```
http://cc-kel4.duckdns.org
```

Jika port forwarding benar dan Apache aktif, kamu akan melihat halaman Apache.

---

