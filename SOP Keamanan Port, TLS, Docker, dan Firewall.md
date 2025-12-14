# 🛡️ SOP Keamanan Port, TLS, Docker, dan Firewall

## Tujuan

Menjamin bahwa:

* Data **terenkripsi** (TLS)
* Service **tidak terbuka ke publik tanpa sengaja**
* Docker **tidak membypass firewall**
* Database & internal service **hanya bisa diakses secara private**

---

## 1️⃣ Prinsip Dasar (WAJIB PAHAM)

### 1. TLS hanya mengamankan **DATA**

TLS (SSL):

* ✅ Mengenkripsi data
* ✅ Memvalidasi identitas server

TLS **TIDAK**:

* ❌ Menutup port
* ❌ Mengatur siapa yang boleh konek
* ❌ Menggantikan firewall

> Jika port terbuka, TLS **tidak mencegah akses** ke port tersebut.

---

### 2. Port Binding Menentukan Keamanan

| Binding          | Makna               | Aman |
| ---------------- | ------------------- | ---- |
| `0.0.0.0:PORT`   | Terbuka ke internet | ❌    |
| `[::]:PORT`      | Terbuka IPv6        | ❌    |
| `127.0.0.1:PORT` | Lokal saja          | ✅    |
| `10.x.x.x:PORT`  | VPN / private       | ✅    |
| `wg0:PORT`       | WireGuard           | ✅    |

---

## 2️⃣ Fakta Penting tentang Docker & UFW

### Masalah Umum

Walaupun **UFW tidak membuka port**, service tetap bisa diakses.

### Penyebab

Docker dengan `ports:`:

```yaml
ports:
  - "5433:5432"
```

Docker akan:

* Membuka port di host
* Listen di `0.0.0.0`
* **Membypass UFW**

> Ini perilaku **normal Docker**, bukan bug.

---

## 3️⃣ Aturan Wajib (DO & DON’T)

### ❌ JANGAN

* Publish port DB ke `0.0.0.0`
* Mengandalkan TLS untuk keamanan akses
* Menganggap UFW melindungi Docker
* Menjalankan service backend listen public

---

### ✅ WAJIB

* Bind service internal ke `127.0.0.1` atau `wg0`
* Gunakan reverse proxy (Caddy) untuk service publik
* Audit port dengan `ss -lntp`
* Audit Docker dengan `docker ps`

---

## 4️⃣ SOP Audit Port (WAJIB RUTIN)

### 4.1 Cek Port Aktif

```bash
ss -lntp
```

### 4.2 Filter Port Public

```bash
ss -lntp | grep -E '0.0.0.0|\[::\]'
```

### 4.3 Cek Docker Port

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

---

## 5️⃣ SOP WireGuard (`wg0`)

Rule seperti:

```text
Anywhere on wg0 ALLOW Anywhere
```

Artinya:

* Semua **client VPN**
* BUKAN internet
* Aman secara default

> `wg0` adalah **network privat**, bukan public interface.

---

# 🐳 CONTOH SETUP DOCKER POSTGRESQL YANG AMAN

## 🎯 Target

* PostgreSQL **tidak public**
* Akses hanya dari:

  * Docker internal network **atau**
  * WireGuard **atau**
  * SSH tunnel
* TLS tetap aktif (opsional, defense-in-depth)

---

## Contoh 1️⃣ – PALING AMAN (REKOMENDASI)

### docker-compose.yml

```yaml
version: "3.9"

services:
  postgres:
    image: postgres:16-alpine
    container_name: pg-secure
    restart: unless-stopped

    environment:
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: strongpassword
      POSTGRES_DB: app_db

    volumes:
      - ./data:/var/lib/postgresql/data

    networks:
      - internal

networks:
  internal:
    driver: bridge
```

### Hasil

* ❌ Tidak ada `ports:`
* ❌ Tidak bisa diakses dari internet
* ✅ Hanya container satu network yang bisa akses

---

## Contoh 2️⃣ – AMAN via Localhost

Jika butuh akses dari host (Prisma / CLI):

```yaml
ports:
  - "127.0.0.1:5433:5432"
```

### Hasil

* ❌ Tidak bisa diakses publik
* ✅ Bisa via `localhost:5433`
* Cocok untuk SSH tunnel / VPN

---

## Contoh 3️⃣ – AMAN via WireGuard (`wg0`)

```yaml
ports:
  - "10.8.0.1:5433:5432"
```

### Hasil

* ❌ Tidak bisa diakses internet
* ✅ Hanya client VPN WireGuard

---

## ❌ Contoh SALAH (JANGAN DITIRU)

```yaml
ports:
  - "5433:5432"
```

❌ Database PUBLIC
❌ Bypass UFW
❌ Rentan serangan

---

## 6️⃣ SOP Reverse Proxy (Node / API)

### Pola yang BENAR

```
INTERNET → CADDY → 127.0.0.1 → APP
```

### Node.js

```js
app.listen(3000, '127.0.0.1')
```

### Caddyfile

```caddyfile
api.example.com {
    reverse_proxy 127.0.0.1:3000
}
```

---

## 7️⃣ Checklist Sebelum Production 🚨

* [ ] Tidak ada DB listen `0.0.0.0`
* [ ] Tidak ada Redis public
* [ ] Backend listen `127.0.0.1`
* [ ] Caddy satu-satunya pintu publik
* [ ] Audit `ss -lntp`
* [ ] Audit `docker ps`
* [ ] TLS aktif (data protection)

---

## 🔚 Kesimpulan Akhir

> **TLS = keamanan DATA**
> **Firewall + Binding Port = keamanan AKSES**
> **Docker = harus diaudit ekstra**
