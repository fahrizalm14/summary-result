Siap 💪 berikut **ringkasan lengkap percakapan kita dari awal sampai terakhir**, dalam urutan logis — biar kamu bisa pakai sebagai *dokumen pemahaman data & prediksi pasar dealer motor Honda (Kendal)*.

---

# 🧾 **Rangkuman Percakapan & Analisis Data Pasar Kendal**

## 🎯 **Tujuan Awal**

Kamu ingin tahu:

> “Bagaimana cara mengetahui total pasar motor Honda per kecamatan hari ini,
> kalau saya hanya punya data penjualan 3 bulan terakhir + data dealer saya sampai hari ini?”

---

## 📘 **Data yang Dipakai**

1. **Data Kota (Pasar total)** – dari file `sales.csv`

   * Tiap baris = 1 unit penjualan
   * Kolom utama: `date`, `subDistrict`, `cityCode`, `dealerCode`, dll
   * Periode: **Mei, Juni, Juli 2025**
   * Digunakan untuk **prediksi pasar Agustus 2025**.

2. **Data Dealer (dealerSales)**

   * Penjualan dealer kamu per kecamatan
   * Periode: **Agustus 1–14 (MTD)**
   * Digunakan untuk menghitung **pangsa pasar dealer per kecamatan.**

---

## ⚙️ **Langkah Analisis**

### 1️⃣ **Rekap Penjualan Pasar (Kota)**

* Hitung total penjualan per **kecamatan per bulan** (group by `subDistrict`, `month`).
* Contoh hasil (Kendal, cityCode=3324):

| Kecamatan | Mei | Jun | Jul |
| --------- | --- | --- | --- |
| Boja      | 173 | 131 | 193 |
| Kaliwungu | 169 | 153 | 148 |
| Patebon   | 118 | 112 | 157 |
| ...       | ... | ... | ... |

---

### 2️⃣ **Prediksi Pasar Bulan Agustus**

Gunakan **rata-rata tertimbang (weighted average)** agar bulan terakhir lebih berpengaruh:

[
Pred_{Agus} = 0.6×Juli + 0.3×Juni + 0.1×Mei
]

Contoh:

```
Boja → 0.6×193 + 0.3×131 + 0.1×173 = 172.4 unit (Agustus penuh)
```

Lalu karena baru pertengahan bulan (tgl 15):
[
Pred_{Agus_MTD} = Pred_{Agus} × 0.5
]
→ misal Boja ≈ 86 unit s/d 15 Agustus.

---

### 3️⃣ **Gabungkan dengan Data Dealer**

Data dealer kamu berisi penjualan per kecamatan s/d 14 Agustus.
Kita padankan `dealerSales` dengan `Pred_Aug_MTD`.

Lalu hitung pangsa pasar:
[
Share = \frac{DealerSales}{Pred_Aug_MTD}
]

Contoh Boja:

```
DealerSales = 6, Pred_Aug_MTD = 86
Share = 6 / 86 = 7.0 %
```

---

### 4️⃣ **Hasil Pangsa Pasar Kendal (MTD Agustus)**

| Kecamatan     | Pred Pasar (MTD) | Dealer MTD | Pangsa Dealer (%) |
| ------------- | ---------------- | ---------- | ----------------- |
| Boja          | 86               | 6          | 7.0               |
| Kaliwungu     | 76               | 7          | 9.2               |
| Patebon       | 70               | 9          | 12.9              |
| Kendal (Kota) | 68               | 0          | 0                 |
| Weleri        | 68               | 1          | 1.5               |
| ...           | ...              | ...        | ...               |
| Ngampel       | 36               | 9          | 25.0              |

---

### 5️⃣ **Total Kota Kendal**

* Total dealer: **≈102 unit**
* Total pasar prediksi (1–14 Agustus): **≈1,038 unit**
* Pangsa pasar dealer kota: **≈9.8 %**

---

## 📊 **Interpretasi Cepat**

| Kategori  | Range | Contoh Kecamatan                   | Catatan                                        |
| --------- | ----- | ---------------------------------- | ---------------------------------------------- |
| 🟩 Kuat   | ≥15%  | Ngampel, Ringinarum, Gemuh         | Dominasi tinggi, pertahankan supply            |
| 🟨 Sedang | 5–15% | Patebon, Pegandon, Boja, Brangsong | Potensi naik lewat promo & stok                |
| 🟥 Lemah  | <5%   | Weleri, Kendal Kota, Kangkung      | Pasar besar tapi share rendah → fokus ekspansi |

---

## 🧠 **Penjelasan Konsep Bobot (0.6, 0.3, 0.1)**

Kenapa bobotnya begitu?

* Karena **bulan terakhir paling relevan**,
* Data lama masih dipakai untuk stabilisasi.
* Total bobot = 1 (60% + 30% + 10%).

Cara menentukan:

* Bisa manual (rule of thumb),
* Bisa diuji (backtest MAPE),
* Bisa otomatis (exponential smoothing).

---

## 🔁 **Untuk Data yang Terus Bertambah**

Kalau database kamu bertambah setiap bulan:

### Opsi 1 — *Sliding Window*

* Gunakan hanya **3–6 bulan terakhir**.
* Data lama disimpan tapi tidak dihitung langsung.
* Cepat dan efisien untuk prediksi rutin.

### Opsi 2 — *Exponential Smoothing (α 0.5–0.7)*

* Bobot otomatis makin kecil untuk data lama.
* Tidak perlu pilih bulan manual.
* Cocok kalau data panjang dan update terus.

### Opsi 3 — *Hybrid Forecast*

[
Forecast_{Final} = 0.7×ShortTerm + 0.3×LongTerm
]

* Gabungkan tren pendek (3 bulan) dengan pola tahunan (12 bulan).

---

## 💼 **Kesimpulan Bisnis**

* Dealer kamu punya **share total 9.8% di Kendal**, dengan dominasi kuat di **Ngampel, Gemuh, Ringinarum**.
* **Target realistis bulan depan:** naik ke **12–15%**.
* Fokus peningkatan di **Weleri & Kendal Kota** (pasar besar, share rendah).
* Gunakan sistem *sliding 3 bulan* atau *exponential smoothing* agar prediksi otomatis saat data terus bertambah.
* Simpan seluruh histori agar bisa analisis tahunan & pola musiman.

---

## 🚀 **Next Step (yang bisa kamu lakukan)**

1. Buat tabel bulanan `MonthlySales` di database.
2. Tambahkan fungsi `forecastKecamatan()`:

   * ambil 3 bulan terakhir,
   * hitung prediksi bulan ini per kecamatan,
   * bandingkan dengan `dealerSales` aktual → dapat pangsa pasar otomatis.
3. Tambah fitur grafik / heatmap untuk visualisasi peta Kendal:

   * hijau = kuat, kuning = sedang, merah = lemah.
4. Recalibrate bobot tiap 6 bulan sekali berdasarkan error (MAPE).

---

Kalau kamu mau, saya bisa bantu buatkan **rancangan skrip `forecastKecamatan()`** (TypeScript/Node.js)
yang langsung ambil data dari tabel Prisma, update prediksi, dan simpan hasil share-nya ke database.

Apakah mau saya lanjut buatkan blueprint-nya (termasuk struktur tabel & fungsi)?
