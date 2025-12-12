# CIDR (Classless Inter-Domain Routing)
Perutean Antar Domain Tanpa Kelas (CIDR) adalah metode alokasi alamat IP yang meningkatkan efisiensi perutean data di internet.

***Note:*** Memetakan IP device agar mudah di kelola

## Pengenalan angka pada IP v4
IP v4 terdiri dari 4 angka (oktet):
```yml
A.B.C.D #4 Oktet
```

Contoh:
```yml
10.31.0.2
```

Setiap oktet:

- Nilai 0 – 255
- 1 oktet = 8 bit
  ```yml
  4 oktet × 8 bit = 32 bit
  ```


## Hubungan BIT → OKTET (ini penting)
| CIDR  | Network pakai | Host pakai | Jumlah IP | Pola alamat | Umum dipakai   |
| ----- | ------------- | ---------- | --------- | ----------- | -------------- | 
| `/8`  | 1 oktet       | 3 oktet    | 16 juta   | A.X.X.X     | Device tunggal | 
| `/16` | 2 oktet       | 2 oktet    | 65 ribu   | A.B.X.X     | Jaringan besar | 
| `/24` | 3 oktet       | 1 oktet    | 256       | A.B.C.X     | LAN / VPN      | 
| `/32` | 4 oktet       | 0          | 1         | A.B.C.D     | ISP / Cloud    | 

***NB:*** Kolom pola alamat adalah 1 subnet, `X` adalah `IP` yang berada di subnet tersebut

## Kenapa ada angka .0 dan .255?

Contoh dalam subnet `/24`:
| IP                | Fungsi                 |
| ----------------- | ---------------------- |
| `10.31.0.0`       | Nama wilayah (network) |
| `10.31.0.255`     | Broadcast              |
| `10.31.0.1 – 254` | Device                 |

## Aturan BESAR pembagian blok IP (global)
Secara resmi, alamat IPv4 dibagi jadi beberapa kategori fungsi.

| Kategori                    | **Blok IP Induk + CIDR yang Boleh** | Rentang Alamat                | Jumlah IP Total (Induk) | Jumlah Device (Usable) | Fungsi Utama                                  |
| --------------------------- | ----------------------------------- | ----------------------------- | ----------------------: | ---------------------: | --------------------------------------------- |
| **Private**                 | `10.0.0.0/(8–32)`                   | 10.0.0.0 – 10.255.255.255     |              16.777.216 |         **16.777.214** | Wilayah private sangat besar (bebas disubnet) |
| **Private**                 | `172.16.0.0/(12–32)`                | 172.16.0.0 – 172.31.255.255   |               1.048.576 |          **1.048.574** | Wilayah private ukuran sedang                 |
| **Private**                 | `192.168.0.0/(16–32)`               | 192.168.0.0 – 192.168.255.255 |                  65.536 |             **65.534** | LAN rumah / kantor                            |
| **Loopback**                | `127.0.0.0/(8–32)`                  | 127.0.0.0 – 127.255.255.255   |              16.777.216 |                  **0** | Diri sendiri (localhost)                      |
| **Link-Local**              | `169.254.0.0/(16–32)`               | 169.254.0.0 – 169.254.255.255 |                  65.536 |             **65.534** | Otomatis saat DHCP gagal                      |
| **Multicast**               | `224.0.0.0/(4–32)`                  | 224.0.0.0 – 239.255.255.255   |             268.435.456 |                  **0** | Pengiriman ke grup                            |
| **Reserved / Experimental** | `240.0.0.0/(4–32)`                  | 240.0.0.0 – 255.255.255.255   |             268.435.456 |                  **0** | Cadangan / eksperimen                         |
| **Public**                  | *(di luar blok khusus)*             | Contoh: 1.1.1.1, 8.8.8.8      |              Bervariasi |             Bervariasi | Internet global                               |

## Tabel Induk /8 → Subnet & Device (CIDR 8–32)
Induk itu pusat jaringan/server yang mengatur IP (main point)
|    CIDR | Level  | Contoh Bentuk IP | Jumlah Subnet dalam 1 Induk /8 | Device Usable per Subnet |
| ------: | ------ | ---------------- | -----------------------------: | -----------------------: |
|  **/8** | Induk  | `X.0.0.0/8`      |                              1 |           **16.777.214** |
|  **/9** | Subnet | `X.0.0.0/9`      |                              2 |                8.388.606 |
| **/10** | Subnet | `X.0.0.0/10`     |                              4 |                4.194.302 |
| **/11** | Subnet | `X.0.0.0/11`     |                              8 |                2.097.150 |
| **/12** | Subnet | `X.0.0.0/12`     |                             16 |                1.048.574 |
| **/13** | Subnet | `X.0.0.0/13`     |                             32 |                  524.286 |
| **/14** | Subnet | `X.0.0.0/14`     |                             64 |                  262.142 |
| **/15** | Subnet | `X.0.0.0/15`     |                            128 |                  131.070 |
| **/16** | Subnet | `X.0.0.0/16`     |                            256 |                   65.534 |
| **/17** | Subnet | `X.0.0.0/17`     |                            512 |                   32.766 |
| **/18** | Subnet | `X.0.0.0/18`     |                          1.024 |                   16.382 |
| **/19** | Subnet | `X.0.0.0/19`     |                          2.048 |                    8.190 |
| **/20** | Subnet | `X.0.0.0/20`     |                          4.096 |                    4.094 |
| **/21** | Subnet | `X.0.0.0/21`     |                          8.192 |                    2.046 |
| **/22** | Subnet | `X.0.0.0/22`     |                         16.384 |                    1.022 |
| **/23** | Subnet | `X.0.0.0/23`     |                         32.768 |                      510 |
| **/24** | Subnet | `X.0.0.0/24`     |                         65.536 |                      254 |
| **/25** | Subnet | `X.0.0.0/25`     |                        131.072 |                      126 |
| **/26** | Subnet | `X.0.0.0/26`     |                        262.144 |                       62 |
| **/27** | Subnet | `X.0.0.0/27`     |                        524.288 |                       30 |
| **/28** | Subnet | `X.0.0.0/28`     |                      1.048.576 |                       14 |
| **/29** | Subnet | `X.0.0.0/29`     |                      2.097.152 |                        6 |
| **/30** | Subnet | `X.0.0.0/30`     |                      4.194.304 |                        2 |
| **/31** | Subnet | `X.0.0.0/31`     |                      8.388.608 |                       0* |
| **/32** | Host   | `X.0.0.1/32`     |                     16.777.216 |                    **1** |

## Perspektif PUBLIK (Internet Global)

| Istilah    | Makna                            | Contoh                     | Siapa yang Menggunakan           |
| ---------- | -------------------------------- | -------------------------- | -------------------------------- |
| **Induk**  | Wilayah IP global besar (/8)     | `1.0.0.0/8`, `103.0.0.0/8` | **RIR (APNIC, ARIN), ISP besar** |
| **Subnet** | Pecahan induk untuk ISP / region | `103.41.0.0/16`            | **ISP / Telco**                  |
| **CIDR**   | Aturan pembagian IP publik       | `/8`, `/16`, `/24`         | **ISP / Network Architect**      |
| **Device** | Endpoint publik                  | Router ISP, server publik  | **Router, server, CDN**          |

## Perspektif SERVER / DEVOPS (Jaringan Virtual / Internal)

| Istilah    | Makna                          | Contoh                         | Siapa yang Menggunakan        |
| ---------- | ------------------------------ | ------------------------------ | ----------------------------- |
| **Induk**  | Network virtual buatan sendiri | `10.0.0.0/8`, `10.31.0.0/16`   | **DevOps / Sysadmin**         |
| **Subnet** | Segmentasi internal            | `10.31.1.0/24`, `10.31.2.0/24` | **DevOps / Sysadmin**         |
| **CIDR**   | Alat desain jaringan internal  | `/16`, `/24`, `/32`            | **DevOps / Network Engineer** |
| **Device** | Host internal                  | VM, container, VPN client      | **Server, agent, app**        |

## Perspektif jika bikin layanan VPN
| Level      | Istilah yang Lebih Akurat | Penjelasan                                                              |
| ---------- | ------------------------- | ----------------------------------------------------------------------- |
| **Induk**  | **Network / Pool VPN**    | Bukan server fisiknya, tapi **jaringan virtual** yang server VPN kelola |
| **Subnet** | **Customer / Tenant**     | Tiap customer dapat **wilayah IP sendiri**                              |
| **Device** | **Customer Device**       | Laptop, server, agent milik customer                                    |

### Untuk penyedia layanan VPN (satu produk VPN)
```yml
INDUK (VPN Network)
└── SUBNET (Customer)
    └── DEVICE (Customer Device)
```
```yml
Induk VPN        : 10.100.0.0/12
Customer A       : 10.100.1.0/24 # Customer hanya bisa generate di area ini  10.100.1.1 -  10.100.1.254 device
Customer B       : 10.100.2.0/24  # Customer hanya bisa generate di area ini  10.100.2.1 -  10.100.2.254 device
Device Customer  : 10.100.1.10/32 # IP realnya 10.100.1.10 => device yang customer punya

# BIAR LEBIH PAHAM
Induk VPN        : 10.100.0.0/12        # Pool IP VPN
Customer A       : 10.100.1.0/24        # Area milik Customer A
  - Device A1    : 10.100.1.10/32
  - Device A2    : 10.100.1.11/32

Customer B       : 10.100.2.0/24        # Area milik Customer B
  - Device B1    : 10.100.2.10/32
```

 
