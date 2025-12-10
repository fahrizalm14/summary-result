# Instalasi Docker Redis TLS VPS

## 1. Setup Web server Caddy untuk create `cert` TLS

- Install Caddy (aplikasi webserver seperti nginx)
  ```bash
  sudo apt update
  sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
  
  curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable.gpg
  
  curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | \
  sudo tee /etc/apt/sources.list.d/caddy-stable.list
  
  sudo apt update
  sudo apt install -y caddy
  ```

- Setup Caddfile
  Hal ini dilakukan gar certbot bisa genereate certifacte, soalnya caddy sudah otomatis SSL
  ```caddy
  sub.domain.my.id {
      root * /var/www/certbot
      file_server
  }
  ```
## 2. Generate Certificate
Agar redis bisa di akses melalui TLS

- Install certbot
  ```bash
  sudo apt update
  sudo apt install -y certbot
  ```
- Dapatkan Certifacte
  ```bash
  sudo certbot certonly --standalone -d sub.domain.my.id
  ```
- Cek certificate yang sudah di dapatkan
  ```bash
  /etc/letsencrypt/live/sub.domain.my.id/fullchain.pem
  /etc/letsencrypt/live/sub.domain.my.id/privkey.pem
  ```
## 3. Menyiapkan direktori dan file
- Buat direktori agar mudah dikelola
  
  Contoh skenario instance
  | Instance    | Domain                   | Port TLS |
  | ----------- | -------------------------| -------- |
  | redis-gatot | gatot-redis.domain.my.id | 6380     |
  | redis-lapu  | lapu-redis.domain.mmy.id | 6381     |
  | redis-joko  | joko-redis.domain.my.id  | 6382     |

  Contoh struktur folder multi-instnace
  ```yml
  /db
   ├─ redis-gatot/
   │   ├─ redis.conf   # <=== redis config yang dimount docker compose
   │   ├─ docker-compose.yml
   │   └─ (mount certs)
   │
   ├─ redis-lapu/
   │   ├─ redis.conf
   │   ├─ docker-compose.yml
   │   └─ (mount certs)
   │
   └─ redis-jaya/
       ├─ redis.conf
       ├─ docker-compose.yml
       └─ (mount certs)
  ```
- Konfigurasi  `redis-gatot/redis.conf`
  ```yml
  port 0 # Non TLS disabled
  tls-port 6380
  
  bind 0.0.0.0
  
  # Pakai path asli Let's Encrypt di dalam container,
  # karena nanti kita mount /etc/letsencrypt apa adanya
  tls-cert-file /etc/letsencrypt/live/sub.domain.my.id/fullchain.pem
  tls-key-file /etc/letsencrypt/live/sub.domain.my.id/privkey.pem
  tls-ca-cert-file /etc/letsencrypt/live/sub.domain.my.id/fullchain.pem
  
  requirepass passwordsayakuat
  
  protected-mode yes
  tls-auth-clients no
  ```
- Buat file docker-compose `redis-gatot/docker-compose.yml`
  ```yml
  services:
  lapu-redis:
    image: redis:7-alpine
    container_name: redis-lapu
    restart: unless-stopped
    command: ["redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf:ro
      - /etc/letsencrypt:/etc/letsencrypt:ro
    ports:
      - "6381:6380" # ==> 6381 port vps, 6380 port container sesuai redis.conf tls-port 6380
  ```

## Tes/cara koneksi redis
- Connection string
  ```env
  REDIS_URL=rediss://default:passwordkukuat@host:port/0
  ```
- Redis cli untuk tes koneksi
  ```bash
  sudo apt install redis-tools
  ```
  ```bash
  resdis-cli --version
  ```
  ```bash
  redis-cli --tls -h sub.domain.my.id -p 6381 -a 'passwordkukuat'
  ```
- Contoh app `main.js`
  ```js
  import Redis from "ioredis";
  import dotenv from "dotenv";
  
  dotenv.config();
  
  if (!process.env.REDIS_URL) {
    console.error("❌ REDIS_URL is not set in .env");
    process.exit(1);
  }
  
  const redis = new Redis(process.env.REDIS_URL, {
    tls: {
      rejectUnauthorized: false, 
      // Jika pakai Let's Encrypt, kamu bisa ganti:
      // rejectUnauthorized: true,
    },
  });
  
  async function test() {
    try {
      console.log("🚀 Connecting to Redis...");
      await redis.set("hello", "world");
      const value = await redis.get("hello");
      console.log("✔️ Redis Response:", value);
  
      redis.disconnect();
      console.log("🔌 Disconnected");
    } catch (err) {
      console.error("❌ Redis Error:", err);
    }
  }
  
  test();
  ```

## Troubleshooting
- Pastikan cloudflare Grey `DNS Only`
  ```java
  ☁️ Grey cloud (DNS Only)
  ```
- Firewall sudah buka port
  ```bash
  sudo ufw allow 6381/tcp
  ```
