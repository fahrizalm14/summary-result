## Menyiapkan certificate yang sudah dibuat:

- Struktur Folder
  ```yml
    /home/rz-dbhost/db/postgres/franco-pg
    ├── docker-compose.yml
    ├── .env
    ├── data/
    └── certs/
        ├── server.crt
        ├── server.key
        └── ca.crt
  ```

- Salin sertifikat yang sudah dibuat ke folder docker compose
  
  ```bash
  rsync -avz \
    /opt/pki/franco-pg.rz-dbhost.my.id/server.crt \
    /opt/pki/franco-pg.rz-dbhost.my.id/server.key \
    /opt/pki/root-ca/ca.crt \
    /home/rz-dbhost/db/postgres/franco-pg/certs
  ```
- Permission (Wajib)
  Jalankan di HOST VPS (folder cert yang sama):
  ```bash
  sudo chown 70:70 ca.crt server.crt server.key
  ```
  
  Lalu set permission ulang (biar rapi & sesuai rule Postgres):
  
  ```bash
  sudo chmod 600 server.key
  sudo chmod 644 server.crt ca.crt
  ```
  
- File 
  ```yml
  
  # ENV FILE (.env)
  # =================================================================================================== #
  # PostgreSQL credentials
  POSTGRES_DB=
  POSTGRES_USER=
  POSTGRES_PASSWORD=
  
  # Network
  PG_PORT=5434
  WG_IP=10.10.0.4 # IP WG CLIENT

  # =================================================================================================== #
  
  # DOCKER COMPOSE  (docker-compose.yml)
  # =================================================================================================== #
    services:
    franco-pg:
      image: postgres:16-alpine
      container_name: franco-pg
      restart: unless-stopped
  
      env_file:
        - .env
  
      volumes:
        - ./data:/var/lib/postgresql/data
        - ./certs:/certs:ro
  
      command: >
        postgres
          -c ssl=on
          -c ssl_cert_file=/certs/server.crt
          -c ssl_key_file=/certs/server.key
          -c ssl_ca_file=/certs/ca.crt
          -c listen_addresses='*'
          -c port=${PG_PORT}
  
      # 🔒 BIND hanya ke IP wg0 (VPN)
      ports:
        - "${WG_IP}:${PG_PORT}:5432"
  
      networks:
        - pg_private
  
  networks:
    pg_private:
      driver: bridge
  # =================================================================================================== #
  ```
