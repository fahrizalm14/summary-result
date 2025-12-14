# 🛡️ Generate Root CA Key dan Server Key
---

## Langkah (Copy–Paste Safe)

```bash
# masuk sebagai root
sudo -i

# buat folder PKI (jika belum ada)
mkdir -p /opt/pki/root-ca
chmod 700 /opt/pki/root-ca

# MASUK ke folder (WAJIB)
cd /opt/pki/root-ca

# pastikan lokasi benar
pwd
# HARUS: /opt/pki/root-ca

# generate Root CA private key
openssl genrsa -out ca.key 4096
chmod 600 ca.key
```

---

## Validasi

```bash
ls -l /opt/pki/root-ca/ca.key
```

**Output yang benar:**

```text
-rw------- 1 root root ... ca.key
```
**Kondisi Saat Ini (cek dulu)**

```text
/opt/pki/root-ca/
├── ca.key   ✅ (RAHASIA)
└── ca.crt   ❌ (BELUM ADA)
```

## Aturan Wajib (Keamanan)

* `ca.key` **HANYA** disimpan di `/opt/pki/root-ca`
* ❌ Jangan ke Docker
* ❌ Jangan ke repo Git
* ❌ Jangan ke server aplikasi
* Folder ini **khusus PKI**

---

# 🥇 Generate Certificate

**1️⃣ Generate Root CA Certificate (`ca.crt`)**

Masih di folder yang sama:

```bash
cd /opt/pki/root-ca
```

Jalankan:

```bash
openssl req -x509 -new \
  -key ca.key \
  -sha256 \
  -days 3650 \
  -out ca.crt \
  -subj "/C=ID/O=Internal-Infra/CN=Internal-Root-CA"
```

Validasi:

```bash
openssl x509 -in ca.crt -noout -subject -issuer -dates
```

**2️⃣ Generate SERVER CERT untuk PostgreSQL/Redis etc `INSTANCE`**

```bash
mkdir -p /opt/pki/franco-pg.rz-dbhost.my.id
cd franco-pg.rz-dbhost.my.id
```


**3️⃣ Generate private key server `INSTANCE`**

```bash
openssl genrsa -out server.key 4096
chmod 600 server.key
```

**4️⃣Buat config SAN**

Jalankan:
```bash
# HARUS: /opt/pki/franco-pg.rz-dbhost.my.id
cat > server.cnf <<EOF
[req]
prompt = no
distinguished_name = dn
req_extensions = ext

[dn]
CN = franco-pg.rz-dbhost.my.id

[ext]
subjectAltName = DNS:franco-pg.rz-dbhost.my.id
EOF

```

**5️⃣ Generate CSR (pakai SAN)**

```bash
openssl req -new \
  -key server.key \
  -out server.csr \
  -config server.cnf
```

**6️⃣ Sign CSR → server cert**

```bash
openssl x509 -req \
  -in server.csr \
  -CA /opt/pki/root-ca/ca.crt \
  -CAkey /opt/pki/root-ca/ca.key \
  -CAcreateserial \
  -out server.crt \
  -days 365 \
  -sha256 \
  -extfile server.cnf \
  -extensions ext
```

**7️⃣ Verifikasi (WAJIB)**

```bash
openssl x509 -in server.crt -noout -text | grep -A1 "Subject Alternative Name"
```

Output HARUS:
```bash
DNS:franco-pg.rz-dbhost.my.id
```

Kamu sekarang punya:
```bash
/opt/pki/franco-pg.rz-dbhost.my.id/
├── server.key   ❌ RAHASIA
├── server.csr
├── server.crt   ✅
└── server.cnf
```




