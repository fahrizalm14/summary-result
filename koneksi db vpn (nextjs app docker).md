Siap 👍
Berikut **summary + command praktis (copy–paste ready)**.

---

## 🔐 1. Instal WireGuard (Host VPS)

```bash
sudo apt update
sudo apt install -y wireguard resolvconf
```

Cek:

```bash
wg --version
```

---

## ⚙️ 2. Konfigurasi WireGuard

### Simpan config

```bash
sudo nano /etc/wireguard/wg0.conf
sudo chmod 600 /etc/wireguard/wg0.conf
```

### Jalankan WG

```bash
sudo wg-quick up wg0
```

### Auto-start saat boot

```bash
sudo systemctl enable wg-quick@wg0
```

### Verifikasi

```bash
ip a show wg0
wg
ping 10.10.0.4   # IP WG VPS tujuan
```

---

## 🐳 3. Network Bridge Docker → WireGuard (KALAU ERROR BARU DIJALANKAN)

### Aktifkan IP forwarding

```bash
sysctl net.ipv4.ip_forward
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-ipforward.conf
```

---

### NAT subnet Docker → wg0

```bash
sudo iptables -t nat -A POSTROUTING -s 172.17.0.0/16 -o wg0 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 172.19.0.0/16 -o wg0 -j MASQUERADE
```

---

### Allow forward Docker ↔ WireGuard

```bash
sudo iptables -I DOCKER-USER -i docker0 -o wg0 -j ACCEPT
sudo iptables -I DOCKER-USER -i br-81a48a2c1d38 -o wg0 -j ACCEPT

sudo iptables -I DOCKER-USER -i wg0 -o docker0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -I DOCKER-USER -i wg0 -o br-81a48a2c1d38 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

---

## 🔎 4. Verifikasi dari Container (Debug)

```bash
docker run --rm -it alpine sh
apk add iputils postgresql-client
ping 10.10.0.4
psql -h 10.10.0.4 -U franco_user -d osist_db
```

---

## 🧾 5. App Container

### `.env.production`

```env
DATABASE_URL=postgresql://franco_user:password@10.10.0.4:5432/osist_db
```

### Restart app

```bash
docker compose down
docker compose up -d
docker logs -f osist-app
```

---

## Docker compose next app
```yml
services:
  app:
    image: emhafahrizal/osist:latest
    container_name: osist-app
    env_file: .env.production
    ports:
      - "3001:3001"
    restart: unless-stopped
    network_mode: bridge # agar bisa koneksi vpn
```

## ✅ Hasil Akhir

* WireGuard aktif & persistent
* Host & container bisa akses DB via VPN
* Docker bridge aman (tanpa `network_mode: host`)
* Database private, production-ready

