# VERSI 1 — Ringkasan Produk + Fitur → Akses (dengan penjelasan)

## 🎯 Ringkasan Utama

* **Hierarki & peran:**
  **SA** (nasional), **AW** (provinsi, memanage provinsi & buat info tingkat kab/kota), **AC** (kota/kab, mengelola pelatihan kaderisasi untuk instruktur & peserta), **AAC** (kecamatan), **PT** (pelatih—berinduk ke AAC, bisa pegang ≥1 pelajaran), **PS** (peserta).
* **Prinsip akses:** “**fitur dulu → ceklis role**”. Data-scope mengikuti hierarki (NAS→PROV→KAB/KOTA→KEC→Kelas).
* **Registrasi:** **hanya PS** yang register via homepage (role lain dibuat oleh atasan).
* **Modul inti (MVP):** Auth, Profil, Dashboard, Master Data hirarki & Info Organisasi, Kalender, Approval PS, Pelatihan (Kategori, Persyaratan, Materi, Ujian), Absensi, ID Card, Sertifikat, RKTL.
* **Target pengalaman:** cepat dioperasikan petugas daerah, rapi terdokumentasi, jejak audit jelas, ekspor data mudah.

---

## 🧩 Fitur → Akses per Role + Penjelasan Fungsi

> Notasi: **\[x]** = punya akses, **\[ ]** = tidak.
> Setiap fitur disertai **fungsi/tujuan** agar tim paham konteks bisnisnya.

### 1) Autentikasi

**Fungsi:** Mengamankan akses; **register hanya PS** dari homepage; lupa/reset sandi.

* SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[x] · PS \[x]

### 2) Profil

**Fungsi:** Kelola biodata, foto, preferensi notifikasi/bahasa, ganti password.

* SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[x] · PS \[x]

### 3) Dashboard Kaderisasi (sesuai wilayah & cabang)

**Fungsi:** Ringkas KPI (peserta, kelas aktif, kelulusan, agenda) sesuai scope role.

* SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[x] · PS \[ ]

### 4) Dashboard Diklat (PS)

**Fungsi:** Progres pribadi: materi dibaca, tes lulus, cetak sertifikat (jika lulus).

* SA \[ ] · AW \[ ] · AC \[ ] · AAC \[ ] · PT \[ ] · PS \[x]

### 5) Manajemen Fitur (Menu Management)

**Fungsi:** SA mengatur fitur/menu yang tampil per role.

* SA \[x] · AW \[ ] · AC \[ ] · AAC \[ ] · PT \[ ] · PS \[ ]

### 6) Master Data

**Fungsi umum:** Mengelola struktur organisasi & aktor sesuai hierarki.

* **CRUD Admin Wilayah (AW)** — SA \[x] · lainnya \[ ]
* **CRUD Admin Cabang (AC)** — SA \[x] · AW \[x] · lainnya \[ ]
* **CRUD Admin Anak Cabang (AAC)** — SA \[x] · AW \[x] · AC \[x] · lainnya \[ ]
* **CRUD Instruktur (PT)** — SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PS \[ ]

  > Tujuan: AC/AAC menugaskan PT ke kelas/pelajaran lokal.
* **CRUD Peserta (PS)** — SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[ ]

  > Di luar register publik, data PS bisa ditambah/diedit oleh admin berjenjang.
* **Info Organisasi – Nasional** — SA \[x] · lainnya \[ ]
* **Info Organisasi – Provinsi** — SA \[x] · AW \[x] · lainnya \[ ]
* **Info Organisasi – Kecamatan** — SA \[x] · AW \[x] · AC \[x] · AAC \[x]

  > Tujuan: komunikasi resmi/edaran per level; ada versi/riwayat & publish.

### 7) Kalender

**Fungsi:** Menjadwalkan kegiatan/kelas; undang PT/PS; pengingat.

* **Read:** SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[x] · PS \[x]
* **Create/Update/Delete:** SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[ ] · PS \[ ]

### 8) Approval Peserta (dari register homepage)

**Fungsi:** Menyaring PS yang mendaftar; menempatkan ke kelas/gelombang.

* SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[ ] · PS \[ ]

### 9) ID Card

**Fungsi:** Identitas dengan **QR**; dipakai saat absensi/verifikasi.

* **Create:** PS \[x] · PT \[x]
* **Read/Download:** SA \[x] · AW \[x] · AC \[x] · AAC \[x]

### 10) Absensi

**Fungsi:** Catat kehadiran tiap sesi (QR/GPS/Kode/Manual) + rekap.

* **Input:** PS \[x] · PT \[x]
* **Read/Download (rekap):** SA \[x] · AW \[x] · AC \[x] · AAC \[x]

### 11) Sertifikat

**Fungsi:** Bukti kelulusan; PS unduh, admin/pt kelola.

* **Kelola/Akses admin:** SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[x]
* **Read/Download (pemilik):** PS \[x]

### 12) Kategori Pelatihan (Diklat)

**Fungsi:** Mengelompokkan program/kelas agar terstruktur.

* SA \[x] · AW \[x] · AC \[x] · AAC \[x] · PT \[ ] · PS \[ ]

### 13) Pelatihan

* **Persyaratan**
  **Fungsi:** Tetapkan & kumpulkan dokumen syarat dari PS.

  * **CRUD Master:** SA \[x] · AW \[x] · AC \[x] · AAC \[x]
  * **CRUD Detail (isi):** PS \[x]
* **Materi**
  **Fungsi:** Distribusi modul (file/link/video) & progres baca.

  * **Read:** SA/AW/AC/AAC/PT/PS \[x]
  * **CUD:** SA \[x] · AW \[x] · AC \[x] · AAC \[x]
* **Ujian (Pre/Post)**
  **Fungsi:** Evaluasi pembelajaran; skor & analitik.

  * **Master Soal – Read:** SA/AW/AC/AAC/PT/PS \[x]
  * **Master Soal – CUD:** SA \[x] · AW \[x] · AC \[x] · AAC \[x]
  * **Jawaban – Read:** SA/AW/AC/AAC/PT/PS \[x]
  * **Jawaban – CUD (isi):** PS \[x]
* **RKTL (Rencana Kerja Tindak Lanjut)**
  **Fungsi:** Rencana aksi pasca-diklat + alur approval.

  * **Read (termasuk status PS):** semua role \[x]
  * **CUD (isi):** PS \[x]
  * **Approval:** SA/AW/AC/AAC/PT \[x]

---

# VERSI 2 — Roadmap Teknis + Schedule 8 Minggu

## 🏗️ Arsitektur & Stack

* **Frontend/Backend:** Next.js (App Router) + TypeScript + Tailwind, Node v20
* **Database:** PostgreSQL + Prisma (migrasi & seeding)
* **Auth:** JWT (credentials), register **PS-only** (public), RBAC middleware
* **Storage:** S3 presigned URL (upload/download), validasi mime/size
* **Quality:** ESLint/Prettier/Vitest, OpenAPI/Swagger, audit log, observability

---

## 🗓️ Jadwal (Asia/Jakarta) — 8 Minggu

| Minggu | Tanggal        | Fokus & DoD                                                                                                                                                                                                          |
| ------ | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1**  | 22–28 Sep 2025 | **Bootstrap & Pondasi**: Next.js+TS+Tailwind; Prisma+Postgres; layout dasar; pola response API; **DoD:** app jalan, DB terkoneksi, migrasi awal (`User`, `Role`, `Region`) & seed role.                              |
| **2**  | 29 Sep–5 Okt   | **Auth & RBAC**: login, register **PS-only**, forgot/reset; middleware RBAC; **Profil**. **DoD:** semua role bisa login; PS bisa register & reset; route terlindungi.                                                |
| **3**  | 6–12 Okt       | **Master Data & Info Org**: CRUD AW/AC/AAC/PT/PS; Info Organisasi NAS/PROV/KEC sesuai akses; scope filter. **DoD:** hierarki & scope berfungsi.                                                                      |
| **4**  | 13–19 Okt      | **Kalender & Approval PS**: Read all; CUD SA–AAC; event berulang; antrian approve/reject + catatan + audit. **DoD:** notifikasi dasar & audit trail.                                                                 |
| **5**  | 20–26 Okt      | **Kategori & Persyaratan**: CRUD kategori; persyaratan master (SA–AAC); **PS submit** syarat. **DoD:** validasi file/status.                                                                                         |
| **6**  | 27 Okt–2 Nov   | **Materi & S3**: CUD materi (SA–AAC); Read all; presigned S3 upload/download; progress baca. **DoD:** unggah/unduh stabil, authorized.                                                                               |
| **7**  | 3–9 Nov        | **Ujian & Absensi**: bank soal (CUD SA–AAC), attempt PS (timer/autosave), skor; Absensi input (PT/PS) + rekap (SA–AAC). **DoD:** nilai & rekap ekspor CSV/XLS.                                                       |
| **8**  | 10–16 Nov      | **ID Card, Sertifikat, RKTL & Dashboard**: generate ID (PS/PT) + download admin; sertifikat (PS download, admin/pt kelola); RKTL (PS CUD, approval SA–PT); widget dashboard. **DoD:** alur E2E, OpenAPI, smoke test. |

> Catatan: Minggu 6–8 bisa paralel (Content vs Evaluation vs Credentialing) bila tim >1 skuad.

---

## 🧱 Backlog Terstruktur (Checklist Eksekusi)

> Tag: **\[DB]** skema, **\[API]** endpoint/service, **\[APP]** UI/UX, **\[SEC]** keamanan, **\[QA]** pengujian

### A. Pondasi

* [ ] **\[APP]** Init Next.js+TS+Tailwind, ESLint/Prettier, layout & komponen dasar
* [ ] **\[DB]** Init Prisma, koneksi PostgreSQL, migrasi awal (`User`, `Role`, `Region`)
* [ ] **\[API]** Helper response & error handler seragam
* [ ] **\[SEC]** CORS, helmet basic, .env per environment
* [ ] **\[QA]** Vitest & workflow CI lint/test

### B. Auth & Profil

* [ ] **\[API]** Login/Refresh/Logout; **Register PS (public)**; Forgot/Reset
* [ ] **\[SEC]** Hash+salt, rate-limit, lockout; middleware RBAC (SA/AW/AC/AAC/PT/PS)
* [ ] **\[APP]** Form login/register/forgot/reset; halaman Profil (biodata/foto)
* [ ] **\[DB]** `PasswordResetToken`, `Profile`
* [ ] **\[QA]** Happy/negative path, expiry token

### C. Menu Management (SA)

* [ ] **\[DB]** `Feature`, `RoleFeature`
* [ ] **\[API]** CRUD fitur + assign/unassign ke role
* [ ] **\[APP]** Panel SA untuk toggle & urutan menu
* [ ] **\[SEC]** Guard server & client
* [ ] **\[QA]** Snapshot menu per role

### D. Master Data Hirarki

* [ ] **\[API]** CRUD **AW** (SA), **AC** (SA/AW), **AAC** (SA/AW/AC)
* [ ] **\[DB]** `Instructor`, `Participant` + relasi kelas/pelajaran
* [ ] **\[API]** CRUD **PT** (SA/AW/AC/AAC), **PS** (SA/AW/AC/AAC)
* [ ] **\[API]** Info Organisasi: **NAS** (SA), **PROV** (SA/AW), **KEC** (SA/AW/AC/AAC) + versi/publish
* [ ] **\[SEC]** Scope filter otomatis (NAS/PROV/KAB/Kec)
* [ ] **\[QA]** Akses lintas-scope ditolak

### E. Kalender

* [ ] **\[DB]** `CalendarEvent{title,start,end,repeat,scope,audience,reminder}`
* [ ] **\[API]** **Read** (semua), **CUD** (SA/AW/AC/AAC), undang PT/PS
* [ ] **\[APP]** View bulanan/agenda, form event (berulang, reminder)
* [ ] **\[QA]** Timezone, overlap, notifikasi

### F. Approval PS

* [ ] **\[DB]** `Enrollment{participantId,classId,status:'PENDING|APPROVED|REJECTED',note}`
* [ ] **\[API]** Queue pending; Approve/Reject + catatan (SA/AW/AC/AAC)
* [ ] **\[APP]** Tabel approval + filter wilayah/kelas + bulk action
* [ ] **\[QA]** Audit trail; cegah double-approve

### G. Pelatihan

* **Kategori (SA/AW/AC/AAC)**

  * [ ] **\[DB]** `TrainingCategory` · **\[API]** CRUD + urutan
* **Persyaratan**

  * **Master (SA/AW/AC/AAC):** \[ ] CRUD `RequirementMaster`
  * **Detail (PS):** \[ ] `RequirementSubmission{status,note,files}` + submit/revisi
* **Materi**

  * **CUD (SA/AW/AC/AAC):** \[ ] `Material{type,visibility,order}`
  * **Read (semua):** \[ ] Viewer + **`MaterialProgress`**
* **Ujian (Pre/Post)**

  * **Master Soal:** **Read (semua)**, **CUD (SA–AAC)** → `[Exam]`, `[ExamQuestion]`
  * **Jawaban:** **Read (semua)**, **CUD (PS)** → `[ExamAttempt]`, `[ExamAnswer]`, `[ExamResult]`
  * [ ] **\[APP]** UI ujian (timer, autosave, resume), skor & review
  * [ ] **\[SEC]** Randomisasi soal/opsi, rate-limit submit

### H. Absensi

* [ ] **\[DB]** `Attendance{sessionId,userId,method:'QR|GPS|CODE|MANUAL',time,latlon}`
* [ ] **\[API]** **Input** (PT/PS), **Rekap/Download** (SA–AAC)
* [ ] **\[APP]** Scan QR / geofence; rekap CSV/XLS
* [ ] **\[SEC]** Anti-spoof (radius/time window), anti-duplikasi
* [ ] **\[QA]** Edge offline/terlambat

### I. ID Card

* [ ] **\[DB]** `IdCard{userId,number,qr,template,issuedAt}`
* [ ] **\[API]** **Create** (PS/PT), **Download** (SA/AW/AC/AAC)
* [ ] **\[APP]** Pratinjau + export PDF
* [ ] **\[QA]** Scan QR → status valid

### J. Sertifikat

* [ ] **\[DB]** `Certificate{userId,programId,number,criteria,qr,status}`
* [ ] **\[API]** Generate/revoke (SA–AAC/PT); **Download (PS)**
* [ ] **\[APP]** Unduh PDF; verifikasi QR
* [ ] **\[QA]** Nomor unik; rule kelulusan konsisten

### K. RKTL

* [ ] **\[DB]** `RKTL{plan,proof,status}`, `RKTLApproval`, `RKTLAudit`
* [ ] **\[API]** **CUD (PS)**, **Read (semua)**, **Approval (SA–PT)**
* [ ] **\[APP]** Form + timeline approval
* [ ] **\[QA]** SLA notifikasi & histori keputusan

### L. S3 Object Storage

* [ ] **\[SEC]** IAM policy minimal; bucket private/public sesuai berkas
* [ ] **\[API]** Presigned URL upload/download + validasi mime/size
* [ ] **\[APP]** Uploader (progress, retry, drag\&drop)
* [ ] **\[QA]** File besar/putus-sambung (resumable opsional)

### M. Observability & Docs

* [ ] **\[OPS]** Audit log (login, CRUD, approval)
* [ ] **\[OPS]** Error tracking & request logging
* [ ] **\[QA]** Unit + integration + e2e (flow register→approval→materi→ujian→sertifikat)
* [ ] **\[DX]** OpenAPI/Swagger + README (alur, RBAC, skema)
* [ ] **\[SEED]** Role, wilayah & kategori dasar

---

## ✅ DoD Umum (Definition of Done)

* [ ] RBAC & scope **match** tabel fitur→akses
* [ ] Audit trail aktif pada aksi penting
* [ ] Presigned S3 aman & tervalidasi
* [ ] Ekspor CSV/XLS tersedia (Absensi, rekap nilai)
* [ ] OpenAPI up-to-date, smoke test lulus untuk alur E2E
