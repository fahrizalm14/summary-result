# Diagram Arsitektur VPN + DNS + Firewall + Docker

```diagram
┌──────────────────────────────────────────────────────────┐
│                     PUBLIC INTERNET                      │
└───────────────▲───────────────────────────────▲──────────┘
                │                               │
        Public DNS (A Record)            VPN Client
  lapu-redis.rz-dbhost.my.id             (Laptop / Admin)
  suyou-pg.rz-dbhost.my.id                      │
                │                               │  WireGuard
                │                               │
                ▼                               ▼
┌──────────────────────────────────────────────────────────┐
│                         VPS HOST                         │
│                    (Ubuntu / Kernel)                     │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │               WireGuard Client (wg0)               │  │
│  │               IP: 10.10.0.4/24                     │  │
│  └──────────────▲──────-───────────────────▲───────-──┘  │
│                 │                          │             │
│       Split DNS │                          │ VPN Access  │
│   (/etc/hosts)  │                          │             │
│                 │                          │             │
│  lapu-redis.* ──┘                          │             │
│  suyou-pg.*   ─────────────────────────────┘             │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │                  FIREWALL (UFW)                    │  │
│  │────────────────────────────────────────────────────│  │
│  │ ALLOW 51820/udp        (WireGuard)                 │  │
│  │ ALLOW 22/tcp via wg0   (SSH VPN-only)              │  │
│  │ ALLOW 80,443           (Public Web)                │  │
│  │ ALLOW 6381             (Redis TLS – sementara)     │  │
│  │ DENY all other inbound                             │  │
│  └───────────────▲────────────────────────────────────┘  │
│                  │                                       │
│             docker0 / bridge                             │
│                  │                                       │
│  ┌────────────────────────────────────────────────────┐  │
│  │                      DOCKER                        │  │
│  │────────────────────────────────────────────────────│  │
│  │                                                    │  │
│  │  wg-easy (VPN Server)                              │  │
│  │   - wg0: 10.10.0.1                                 │  │
│  │                                                    │  │
│  │  redis-lapu (TLS)                                  │  │
│  │   - internal : 6380                                │  │
│  │   - exposed  : 6381                                │  │
│  │                                                    │  │
│  │  postgres (TLS)                                    │  │
│  │   - internal : 5432                                │  │
│  │   - exposed  : 5433                                │  │
│  │                                                    │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```
---

# SOP — Setup WireGuard Client, Split DNS, TLS & Firewall di VPS

## Tujuan
Menyediakan standar operasional untuk:
- Menghubungkan **VPS host** ke **WireGuard VPN (wg-easy)**
- Menghindari error **TLS Redis & PostgreSQL**
- Menyiapkan **firewall UFW yang aman, minimal, dan zero-trust**

---

## 1. Instal & Aktifkan WireGuard Client di VPS (Auto-Run)

### Tujuan
VPS host ikut join VPN (bukan hanya container wg-easy).

### Langkah
```bash
sudo apt update
sudo apt install wireguard wireguard-tools -y
````

Pastikan config client tersedia:

```bash
/etc/wireguard/wg0.conf
```

Aktifkan WireGuard:

```bash
sudo wg-quick up wg0
```

Verifikasi:

```bash
ip a show wg0
```

Aktifkan auto-run saat boot:

```bash
sudo systemctl enable wg-quick@wg0
```

### Hasil

* Interface `wg0` aktif
* VPS mendapat IP VPN (contoh: `10.10.0.4`)
* VPS dapat diakses via VPN

---

## 2. Split DNS untuk HOST VPS (Best Practice)

### Masalah

Setelah WireGuard aktif:

* Redis & PostgreSQL TLS gagal saat diakses via **domain publik**
* Akses via **IP VPN berhasil**

### Penyebab

* Domain resolve ke **IP public VPS**
* VPS mengakses **dirinya sendiri lewat public IP**
* Terjadi **hairpin NAT / asymmetric routing**
* TLS handshake gagal

### Solusi (Split DNS di HOST)

Edit file hosts:

```bash
sudo nano /etc/hosts
```

Tambahkan:

```text
10.10.0.4   lapu-redis.rz-dbhost.my.id
10.10.0.4   suyou-pg.rz-dbhost.my.id
```

Verifikasi:

```bash
getent hosts lapu-redis.rz-dbhost.my.id
getent hosts suyou-pg.rz-dbhost.my.id
```

### Catatan

* Berlaku **hanya di HOST VPS**
* Tidak memengaruhi akses publik
* Tidak perlu restart VPS
* Restart aplikasi jika DNS di-cache

---

## 3. Root Cause TLS Redis & PostgreSQL

### Gejala

* TLS gagal jika akses via domain
* TLS berhasil jika akses via IP VPN

### Root Cause

Redis & PostgreSQL TLS gagal karena:

* Koneksi **ke diri sendiri lewat PUBLIC DOMAIN**
* Routing berubah setelah WireGuard aktif
* Jalur masuk & keluar tidak simetris

### Kesimpulan

* ❌ Bukan bug Redis
* ❌ Bukan bug PostgreSQL
* ❌ Bukan masalah sertifikat
* ✅ Murni masalah routing & DNS

---

## 4. Reset & Setup Firewall UFW (Minimal & Aman)

### Tujuan

* Menghapus rule dobel / acak
* Menyiapkan firewall zero-trust

### Reset Total

```bash
sudo ufw disable
sudo ufw reset
```

### Default Policy

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow Port Wajib

```bash
# WireGuard
sudo ufw allow 51820/udp

# SSH hanya via VPN
sudo ufw allow in on wg0 to any port 22 proto tcp

# Web
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Redis (sementara public)
sudo ufw allow 6381/tcp
```

Aktifkan firewall:

```bash
sudo ufw enable
```

Verifikasi:

```bash
sudo ufw status
```

### Prinsip Firewall

* ❌ Tidak ada SSH public
* ❌ Tidak ada `ALLOW Anywhere on wg0`
* ✅ Allow hanya port yang dibutuhkan
* ✅ VPN sebagai boundary keamanan

---

## 5. Best Practice Tambahan

### WireGuard

* Semua **server-to-server peer**:

  ```ini
  PersistentKeepalive = 25
  ```

### Akses Internal

* Gunakan **IP VPN** untuk koneksi internal
* Gunakan **domain publik** hanya dari luar

### Keamanan

* Jangan hairpin ke domain sendiri dari host
* Audit port berkala:

  ```bash
  ss -lntup
  ```

---

## 6. Final State (Checklist)

* [x] VPS host join VPN
* [x] WireGuard auto-run saat reboot
* [x] SSH hanya via VPN
* [x] Redis & PostgreSQL TLS normal
* [x] Firewall rapi & minimal
* [x] Tidak ada asymmetric routing
* [x] Siap produksi (zero-trust)
