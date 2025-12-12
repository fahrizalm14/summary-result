## 🔐 Apa itu VPN?

**VPN (Virtual Private Network)** adalah teknologi yang membuat **jalur koneksi aman (terenkripsi)** antara **device kamu** dan **server VPN** 🌐.
Hasilnya, device kamu **seolah-olah berada di jaringan privat yang sama dengan server**, walaupun sebenarnya lewat internet publik.

---

## ⚙️ Cara Kerja VPN (Sederhana & Teknis)

1️⃣ Device membuat **network virtual**
2️⃣ Device membangun **tunnel terenkripsi** ke VPN server 🔑
3️⃣ Semua traffic diarahkan lewat tunnel tersebut
4️⃣ VPN server:

* membuka enkripsi
* meneruskan data ke jaringan tujuan

Alur singkatnya:

```
Device 📱💻
   ↓
Tunnel VPN (Encrypted 🔐)
   ↓
VPN Server 🖥️
   ↓
Internet / Network Tujuan 🌍
```

---

## 🎯 Fungsi Nyata VPN (Dipakai untuk Apa?)

VPN dipakai untuk hal-hal berikut:

🔹 **Keamanan koneksi**
Melindungi data saat pakai WiFi publik (kafe, bandara, hotel)

🔹 **Akses jaringan internal**
Masuk ke server, database, atau sistem kantor dari luar jaringan

🔹 **Kontrol akses**
Hanya device tertentu yang boleh masuk ke jaringan privat

🔹 **Manajemen server**
Admin mengelola server tanpa membuka port sensitif ke publik

🔹 **Menyatukan banyak lokasi**
Kantor pusat, cabang, dan device remote berada di **satu jaringan privat**

## 💻 Setup Server

- 🏠 Setup DNS

  | Type | Name | Value  | Proxy           |
  | ---- | ---- | ------ | --------------- |
  | A    | vpn  | IP_VPS | ☁️ Proxied (ON) |

- 🔒 Ufw PORT

  ```bash
  sudo ufw allow ssh
  sudo ufw allow 51820/udp
  sudo ufw allow 80
  sudo ufw allow 443
  sudo ufw enable
  ```
- 📂 Direktori
  ```yml
  ~/wireguard/
  ├── docker-compose.yml
  ├── .env
  └── wg-data/
  ```

- 📋 Environment `.env`
  
  ```yml
  # === SERVER ===
  WG_HOST=vpn1.rz-dbhost.my.id
  
  # === UI AUTH ===
  WG_ADMIN_PASSWORD=$2a$12$/8dLH1qR6C/mfp4JKJODFuDgevX9nWODTuhLBeijFCm1xpSjX1o6i
  
  # === OPTIONAL (BIASANYA TIDAK PERLU DIUBAH UNTUK POIN 1) ===
  WG_PORT=51820
  WG_UI_PORT=51821
  WG_DEFAULT_ADDRESS=10.10.0.x
  WG_DEFAULT_DNS=1.1.1.1
  ```
- 🐳 Docker compose `wg-easy`
  ```yml
    services:
    wg-easy:
      image: ghcr.io/wg-easy/wg-easy
      container_name: wg-easy
      env_file:
        - .env
      environment:
        WG_HOST: ${WG_HOST}
        PASSWORD_HASH: ${WG_ADMIN_PASSWORD}
        WG_PORT: ${WG_PORT}
        WG_DEFAULT_ADDRESS: ${WG_DEFAULT_ADDRESS}
        WG_DEFAULT_DNS: ${WG_DEFAULT_DNS}
      volumes:
        - ./wg-data:/etc/wireguard
      ports:
        - "${WG_PORT}:51820/udp"
        - "127.0.0.1:51821:51821/tcp"
      cap_add:
        - NET_ADMIN
        - SYS_MODULE
      sysctls:
        - net.ipv4.conf.all.src_valid_mark=1
      restart: unless-stopped
  ```
  
  * **wg-easy sudah mencakup WireGuard server**, bukan hanya UI.
  * wg-easy adalah **paket lengkap dalam satu container**, berisi:
  
    * WireGuard server
    * Web UI
    * Tools manajemen
  * Fitur yang sudah ada di dalam wg-easy:
  
    * Generator key (public/private key)
    * Manajemen IP client
    * Manajemen konfigurasi
    * Layanan VPN **UDP 51820**
    * Web UI **TCP 51821**
  * Dengan wg-easy:
  
    * ❌ Tidak perlu install WireGuard native di host
    * ❌ Tidak perlu membuat `wg0.conf` manual
    * ✅ Semua dikonfigurasi otomatis
  * Walaupun berjalan di Docker:
  
    * WireGuard tetap **memakai kernel host**, bukan kernel container
    * Karena itu membutuhkan capability:
  
      * `NET_ADMIN`
      * `SYS_MODULE`
  * WireGuard native di host **hanya diperlukan jika**:
  
    * Setup WireGuard manual tanpa Docker
    * Membutuhkan routing lanjutan di luar wg-easy
  * **Kesimpulan**:
  
    * wg-easy = **WireGuard VPN server lengkap + UI**
    * Setup sudah siap
    * Tinggal **connect client** 🚀
