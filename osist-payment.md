Berikut **summary + isi tabelnya** secara ringkas, biar kebayang struktur minimalnya.

---

## 1. Operasional & Kas (wajib)

### 1.1. `BillingType` – master jenis tagihan

Definisi jenis biaya, belum nempel ke siswa.

| Field                | Keterangan (singkat)          |
| -------------------- | ----------------------------- |
| id                   | ID unik jenis tagihan         |
| websiteId            | Sekolah/tenant                |
| code                 | Kode: `SPP_10`, `UP_2025`     |
| name                 | Nama: `SPP Kelas 10`          |
| category             | SPP / UANG_PANGKAL / KEGIATAN |
| baseAmount           | Nominal dasar rupiah          |
| periodType           | `ONE_TIME` / `RECURRING`      |
| active               | Aktif / tidak                 |
| createdAt, updatedAt | Timestamp                     |

---

### 1.2. `StudentBilling` – tagihan per siswa

Satu baris = satu tagihan milik siswa (per bulan/per event).

| Field                | Keterangan                 |
| -------------------- | -------------------------- |
| id                   | ID tagihan                 |
| websiteId            | Sekolah/tenant             |
| studentId            | Siswa pemilik tagihan      |
| billingTypeId?       | Referensi ke `BillingType` |
| description          | Contoh: `SPP Nov 2025`     |
| amount               | Total nominal tagihan      |
| outstanding          | Sisa belum dibayar         |
| status               | `OPEN` / `PAID` (MVP)      |
| periodKey?           | Contoh: `SPP-2025-11`      |
| dueDate?             | Jatuh tempo                |
| createdAt, updatedAt | Timestamp                  |

---

### 1.3. `Payment` – pemasukan (uang masuk)

Setiap baris = 1 transaksi uang masuk.

| Field                | Keterangan                   |
| -------------------- | ---------------------------- |
| id                   | ID pembayaran                |
| websiteId            | Sekolah/tenant               |
| studentId            | Siswa yang bayar             |
| studentBillingId     | Tagihan yang dibayar         |
| channel              | MANUAL / QRIS / VA_BCA / dll |
| amount               | Nominal dibayar              |
| paidAt               | Tanggal bayar                |
| createdAt, updatedAt | Timestamp                    |

> **Laporan pemasukan** diambil dari `SUM(Payment.amount)` per periode.

---

### 1.4. `ExpenseCategory` – jenis pengeluaran

Master kategori pengeluaran.

| Field     | Keterangan                         |
| --------- | ---------------------------------- |
| id        | ID kategori                        |
| websiteId | Sekolah/tenant                     |
| name      | Nama: Gaji Guru, Listrik, ATK, dll |
| createdAt | Timestamp                          |

---

### 1.5. `Expense` – pengeluaran (uang keluar)

Setiap baris = 1 transaksi uang keluar.

| Field       | Keterangan                       |
| ----------- | -------------------------------- |
| id          | ID pengeluaran                   |
| websiteId   | Sekolah/tenant                   |
| categoryId  | Referensi ke `ExpenseCategory`   |
| description | Contoh: `Bayar listrik November` |
| amount      | Nominal keluar                   |
| paidAt      | Tanggal pengeluaran              |
| method      | CASH / TRANSFER / dll            |
| createdAt   | Timestamp                        |

> **Laporan pengeluaran** dari `SUM(Expense.amount)` per periode.

---

## 2. Akuntansi (opsional tapi siap)

### 2.1. `ChartOfAccount` – daftar akun

Master akun-akun keuangan.

| Field                | Keterangan                                               |
| -------------------- | -------------------------------------------------------- |
| id                   | ID akun                                                  |
| websiteId            | Sekolah/tenant                                           |
| code                 | Kode akun: `1101`, `4101`, dll                           |
| name                 | Nama: Kas, Pendapatan SPP, Beban Listrik                 |
| type                 | `ASSET` / `LIABILITY` / `EQUITY` / `REVENUE` / `EXPENSE` |
| isCash               | True jika akun kas/bank                                  |
| active               | Aktif/tidak                                              |
| createdAt, updatedAt | Timestamp                                                |

---

### 2.2. `JournalEntry` – header jurnal per transaksi

Satu event keuangan (1 pembayaran / 1 pengeluaran).

| Field                | Keterangan                          |
| -------------------- | ----------------------------------- |
| id                   | ID jurnal                           |
| websiteId            | Sekolah/tenant                      |
| refType?             | `PAYMENT` / `EXPENSE` / dll         |
| refId?               | ID sumber (Payment.id / Expense.id) |
| description          | Deskripsi jurnal                    |
| entryDate            | Tanggal jurnal                      |
| createdAt, updatedAt | Timestamp                           |

---

### 2.3. `JournalEntryLine` – baris debit/kredit

Detail akun + nilai Dr/Kr.

| Field          | Keterangan                    |
| -------------- | ----------------------------- |
| id             | ID baris jurnal               |
| journalEntryId | Referensi ke `JournalEntry`   |
| accountId      | Referensi ke `ChartOfAccount` |
| debit          | Nilai debit (0 jika tidak)    |
| credit         | Nilai kredit (0 jika tidak)   |
| createdAt      | Timestamp                     |
