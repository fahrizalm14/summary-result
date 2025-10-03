Berikut ringkas (summary) langkah lengkap dari membuat Cloudflare API Token sampai Caddy berhasil melayani wildcard domain (`*.osist.my.id`) dan mem‑proxy ke Next.js di port 3001.

---

## 1. Persiapan DNS di Cloudflare
Pastikan di Cloudflare (menu DNS):
```
A  osist.my.id  -> IP_VPS   (proxied ON)
A  *            -> IP_VPS   (proxied ON)
```
Mode SSL/TLS di Cloudflare: Full (Strict) (setelah origin sudah punya sertifikat valid).  
(Jika awalnya belum ada cert, sementara boleh Full non‑Strict, jangan pernah pakai Flexible.)

---

## 2. Buat Cloudflare API Token
Cloudflare Dashboard → Profile (avatar kanan atas) → API Tokens → Create Token:
- Template: “Edit zone DNS”
- Permissions:
  - Zone → DNS → Edit
  - (Opsional) Zone → Zone → Read
- Zone Resources: Include → Specific Zone → pilih `osist.my.id`
- Create & salin token (simpan aman).  
Jangan gunakan Global API Key.

---

## 3. Install / Upgrade Go (Jika Belum ≥ 1.21)
Contoh (ganti versi terbaru bila beda):
```bash
sudo rm -rf /usr/local/go
VER=go1.23.2
wget https://go.dev/dl/${VER}.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf ${VER}.linux-amd64.tar.gz
echo 'export PATH=/usr/local/go/bin:$PATH' | sudo tee /etc/profile.d/go.sh
source /etc/profile.d/go.sh
go version   # pastikan go1.23.x
```
Tambahkan juga GOPATH bin (agar `xcaddy` dikenali):
```bash
echo 'export PATH=$HOME/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

## 4. Install xcaddy
```bash
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
which xcaddy
```

---

## 5. Build Caddy dengan Plugin Cloudflare DNS
(Misal ingin versi stable tertentu, contoh v2.10.2 — sesuaikan kalau ada versi baru)
```bash
mkdir -p ~/build-caddy && cd ~/build-caddy
xcaddy build v2.10.2 --with github.com/caddy-dns/cloudflare@latest
./caddy list-modules | grep cloudflare   # harus muncul tls.dns.cloudflare
```

---

## 6. Pasang Binary Caddy Kustom
Hentikan versi lama (apt/snap) jika ada:
```bash
sudo systemctl stop caddy 2>/dev/null || true
sudo snap remove caddy 2>/dev/null || true
```
Salin binary baru:
```bash
sudo cp ./caddy /usr/bin/caddy
sudo chmod +x /usr/bin/caddy
caddy version
```

---

## 7. Simpan Cloudflare API Token di File Environment
Buat file (ISI_TOKEN diganti token asli):
```bash
sudo mkdir -p /etc/caddy
sudo tee /etc/caddy/env >/dev/null <<'EOF'
CLOUDFLARE_API_TOKEN=ISI_TOKEN_CLOUDFLARE_ASLI
EOF
sudo chown caddy:caddy /etc/caddy/env
sudo chmod 640 /etc/caddy/env
```

---

## 8. Tambahkan EnvironmentFile ke Systemd Service
```bash
sudo mkdir -p /etc/systemd/system/caddy.service.d
sudo tee /etc/systemd/system/caddy.service.d/override.conf >/dev/null <<'EOF'
[Service]
EnvironmentFile=/etc/caddy/env
EOF
sudo systemctl daemon-reload
systemctl cat caddy | grep -A1 EnvironmentFile
```

---

## 9. Buat / Edit Caddyfile
Lokasi umum: `/etc/caddy/Caddyfile`
```caddy
{
    email youremail@domain.com
    # debug    # (opsional untuk log detail)
}

osist.my.id, *.osist.my.id {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:3001
}
```
Validasi:
```bash
caddy validate --config /etc/caddy/Caddyfile
```

---

## 10. Start / Restart Caddy
```bash
sudo systemctl restart caddy
sudo systemctl status caddy --no-pager
sudo journalctl -u caddy -n 50 --no-pager
```
Carilah log:
```
... obtained certificate for osist.my.id
... obtained certificate for *.osist.my.id
```

---

## 11. Verifikasi Sertifikat
```bash
echo | openssl s_client -servername osist.my.id -connect osist.my.id:443 2>/dev/null | openssl x509 -noout -subject -issuer
echo | openssl s_client -servername test123.osist.my.id -connect osist.my.id:443 2>/dev/null | openssl x509 -noout -dates
```
Subject / SAN harus mencantumkan `*.osist.my.id` dan `osist.my.id`.

---

## 12. Aplikasi Next.js
Pastikan aplikasi listen di `localhost:3001` (bukan 0.0.0.0 perlu? boleh, tapi localhost cukup). Contoh start:
```bash
PORT=3001 npm run start
```
Atau di file service PM2/systemd.

---

## 13. Cloudflare Mode Final
Set ke: Full (Strict).  
Hindari Flexible. Flexible akan menimbulkan error atau downgrade keamanan.

---

## 14. Troubleshooting Cepat

| Masalah | Penyebab | Solusi Singkat |
|---------|----------|----------------|
| API token '' appears invalid | Var env kosong | Pastikan /etc/caddy/env terisi & override systemd benar |
| Plugin cloudflare tidak muncul | Binary salah (pakai apt/snap) | Rebuild & ganti /usr/bin/caddy |
| Wildcard gagal: offered=[dns-01] only | Normal (wildcard butuh dns-01) | Pastikan plugin & token benar |
| Rate limit Let’s Encrypt | Terlalu sering restart gonta-ganti | Tunggu 1–10 menit, periksa log |
| Origin SSL error di Cloudflare Strict | Sertifikat belum ter-issue | Tunggu issuance selesai dulu |
| Subdomain acak 404 | Next.js belum handle host | Gunakan `req.headers.host` untuk multi-tenant routing |

---

## 15. Script Otomasi (Opsional Contoh)
```bash
#!/usr/bin/env bash
set -e
DOMAIN=osist.my.id
EMAIL=youremail@domain.com
CADDY_VER=v2.10.2
PLUGIN=github.com/caddy-dns/cloudflare
TOKEN="ISI_TOKEN_CLOUDFLARE_ASLI"

echo "[*] Build Caddy..."
mkdir -p ~/build-caddy && cd ~/build-caddy
xcaddy build ${CADDY_VER} --with ${PLUGIN}

echo "[*] Install Caddy..."
sudo cp ./caddy /usr/bin/caddy
sudo chmod +x /usr/bin/caddy

echo "[*] Write env token..."
sudo mkdir -p /etc/caddy
echo "CLOUDFLARE_API_TOKEN=${TOKEN}" | sudo tee /etc/caddy/env
sudo chown caddy:caddy /etc/caddy/env
sudo chmod 640 /etc/caddy/env

echo "[*] Systemd override..."
sudo mkdir -p /etc/systemd/system/caddy.service.d
cat <<EOF | sudo tee /etc/systemd/system/caddy.service.d/override.conf
[Service]
EnvironmentFile=/etc/caddy/env
EOF
sudo systemctl daemon-reload

echo "[*] Caddyfile..."
sudo tee /etc/caddy/Caddyfile >/dev/null <<EOF
{
    email ${EMAIL}
}
${DOMAIN}, *.${DOMAIN} {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:3001
}
EOF

echo "[*] Validate & Start..."
caddy validate --config /etc/caddy/Caddyfile
sudo systemctl restart caddy
sudo journalctl -u caddy -n 30 --no-pager
```
(Sunting TOKEN sebelum jalankan. Jangan simpan token di script produksi tanpa proteksi.)

---

## 16. Keamanan
- Jangan commit token ke repo.
- Jika token bocor → Revoke di Cloudflare → buat baru.
- Bisa batasi token hanya untuk 1 zone.

---

## 17. Checklist Akhir
- [ ] DNS A root & wildcard ke IP VPS
- [ ] Cloudflare API Token (Zone:DNS Edit)
- [ ] Go ≥ 1.21
- [ ] xcaddy ter-install
- [ ] Binary Caddy custom + plugin cloudflare
- [ ] /etc/caddy/env berisi token (owner caddy, mode 640)
- [ ] Systemd override memuat EnvironmentFile
- [ ] Caddyfile ada blok tls dns cloudflare
- [ ] Service jalan, log issuance sukses
- [ ] Cloudflare SSL Mode: Full (Strict)
- [ ] Akses https://subbaru.osist.my.id berhasil

---

Kalau mau saya buatkan versi “tanpa wildcard” atau menambahkan header khusus ke Next.js untuk identifikasi tenant, tinggal bilang. Ada yang ingin didalami lagi?
