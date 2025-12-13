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
   
Siap. Ini **SUMMARY FINAL + CONTOH KONFIGURASI**, lengkap **server & client**, dan **DITAMBAHKAN poin “subnet di sisi server”**.
Ini versi **pegangan teknis** (bisa kamu simpan).

---

# NOTE WIREGUARD CONFIGURATION

## 🔹 1. **AllowedIPs**

### 1️⃣ AllowedIPs di **CLIENT**

**Fungsi:** routing lokal di device (split-tunnel)

```ini
# CLIENT
[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = vpn.nexwage.id:51820
AllowedIPs = 10.200.0.0/16
```

**Makna:**

* Traffic ke `10.200.x.x` → lewat VPN
* Traffic lain → lewat network biasa
* ❌ BUKAN subnet customer
* ❌ BUKAN izin

---

### 2️⃣ AllowedIPs di **SERVER**

**Fungsi:** auth + kepemilikan IP (ACL keras)

```bash
# SERVER (runtime)
wg set wg0 peer PUBKEY_DEVICE_1 allowed-ips 10.100.10.2/32
```

**Makna:**

* PublicKey ini **hanya sah** sebagai `10.100.10.2`
* IP lain → DROP
* ✅ Source of truth keamanan

---

## 🔹 2. **Peer di Config SERVER**

* 1 peer = 1 device
* Identitas = **PublicKey client**
* Server **tidak tahu** subnet / customer / plan

```ini
# SERVER (konseptual)
Peer: PUBKEY_DEVICE_1
AllowedIPs = 10.100.10.2/32

Peer: PUBKEY_DEVICE_2
AllowedIPs = 10.100.10.3/32
```

---

## 🔹 3. **Peer di Config CLIENT**

* Peer = **server WireGuard**
* Client **tidak punya info** peer client lain

```ini
# CLIENT
[Interface]
PrivateKey = PRIVATEKEY_DEVICE_1
Address = 10.100.10.2/32

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = vpn.nexwage.id:51820
AllowedIPs = 10.200.0.0/16
```

---

## 🔹 4. **Konfigurasi WireGuard TIDAK ADA SUBNET**

🔥 **PENTING**

* Tidak ada baris config WireGuard yang menyatakan:

  * “ini subnet customer”
  * “ini tenant”
* WireGuard **hanya kenal IP per device (/32)**

---

## 🔹 5. **Subnet DI SETTING DI SISI SERVER (DI LUAR WIREGUARD)**

### 1️⃣ Subnet didefinisikan di **BACKEND (logic)**

Contoh:

```text
Customer A → 10.100.10.0/24
Customer B → 10.100.20.0/24
```

Backend memastikan:

* IP device A **selalu** di `10.100.10.0/24`
* IP device B **selalu** di `10.100.20.0/24`

---

### 2️⃣ Subnet ditegakkan di **FIREWALL SERVER**

```bash
# DROP default
iptables -P FORWARD DROP

# Allow WireGuard
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -A FORWARD -o wg0 -j ACCEPT

# Subnet customer A
iptables -A FORWARD -s 10.100.10.0/24 -j ACCEPT

# Block antar customer
iptables -A FORWARD -s 10.100.10.0/24 -d 10.100.20.0/24 -j DROP
iptables -A FORWARD -s 10.100.20.0/24 -d 10.100.10.0/24 -j DROP
```

➡️ **DI SINILAH subnet “hidup” secara nyata**

---

## 🧠 CHEAT SHEET (1 LAYAR)

| Item                | Di mana            | Fungsi           |
| ------------------- | ------------------ | ---------------- |
| AllowedIPs (client) | Client             | Routing lokal    |
| AllowedIPs (server) | Server             | Auth + ACL       |
| Peer server         | Server             | 1 device = 1 key |
| Peer client         | Client             | Server tujuan    |
| Subnet customer     | Backend + Firewall | Group & isolasi  |

---

## 🧠 1 KALIMAT FINAL (INGAT INI)

> **WireGuard hanya mengamankan koneksi per device; subnet dan isolasi customer sepenuhnya ditentukan oleh backend dan firewall di sisi server.**

---

Kalau mau, aku bisa lanjutkan dengan:

* **diagram ASCII final**
* **checklist implementasi NexWage VPN**
* **agent-only zero-trust flow**

Tinggal bilang mau lanjut yang mana.

