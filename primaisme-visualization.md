Siap 🔥 — berikut **summary komprehensif** yang menjelaskan:

* konsep dan cara menghitung *sliding window forecasting*,
* bagaimana cara mengukur akurasi (MAPE),
* alasan kenapa metode ini relevan untuk data pasar dealer,
* contoh hasil nyata,
* dan kode TypeScript siap pakai.

---

# 🧾 **SLIDING WINDOW FORECASTING — RINGKASAN & IMPLEMENTASI**

## 🎯 **Tujuan**

Membuat prediksi **total pasar bulan berikutnya**
berdasarkan **3 bulan terakhir** data penjualan aktual.
Contoh kasus: memprediksi **pasar motor Honda per kecamatan** di Kendal.

---

## ⚙️ **1️⃣ Rumus dasar perhitungan**

Kita gunakan rata-rata tertimbang (*weighted moving average*):

[
\text{Forecast}*{t+1} = 0.6×Sales_t + 0.3×Sales*{t-1} + 0.1×Sales_{t-2}
]

Artinya:

* **Sales_t** → penjualan bulan terakhir (paling relevan, bobot 60%)
* **Sales_{t-1}** → bulan sebelumnya (bobot 30%)
* **Sales_{t-2}** → dua bulan sebelumnya (bobot 10%)
* total bobot = 1 (100%)

📌 Bobot ini bisa diubah sesuai karakter datamu, misalnya (0.5, 0.3, 0.2) jika pasar lebih stabil.

---

## 📈 **2️⃣ Kenapa 3 bulan terakhir relevan**

1. **Tren motor bersifat bulanan** → promo, cuaca, pameran, dan stok cepat berubah.
2. **Bulan terakhir** lebih mencerminkan kondisi aktual dibanding data lama.
3. **Data lama tetap penting** untuk stabilisasi — mencegah hasil “loncat-loncat”.
4. **Sliding window** menjaga sistem tetap adaptif tanpa kehilangan konteks.

   * Saat masuk bulan baru, window bergeser otomatis (misal dari Mei–Jul ke Jun–Agu).

---

## 📊 **3️⃣ Pengukuran akurasi — MAPE**

MAPE (Mean Absolute Percentage Error) mengukur seberapa jauh prediksi dari data aktual.

[
MAPE = \frac{1}{n} \sum_{i=1}^{n} \left|\frac{Actual_i - Forecast_i}{Actual_i}\right| \times 100%
]

Contoh:

| Bulan                | Aktual | Prediksi | Error      |
| -------------------- | ------ | -------- | ---------- |
| Juli                 | 120    | 130      | 8.3%       |
| Agustus              | 150    | 145      | 3.3%       |
| September            | 140    | 155      | 10.7%      |
| **Rata-rata (MAPE)** |        |          | **7.4%** ✅ |

🟢 Nilai MAPE bagus biasanya **<10%**,
🔵 di 10–20% masih bisa diterima untuk data pasar yang fluktuatif.

---

## 🧮 **4️⃣ Contoh perhitungan nyata**

Misal data 3 bulan terakhir kecamatan **Boja (cityCode 3324):**

| Bulan     | Total Sales |
| --------- | ----------- |
| Juli      | 193         |
| Agustus   | 131         |
| September | 173         |

Prediksi Oktober:
[
0.6×193 + 0.3×131 + 0.1×173 = 172.4
]
→ Jadi **prediksi pasar Oktober = 172 unit.**

---

## 🧠 **5️⃣ Logika otomatis: sliding window dinamis**

Sistem otomatis tahu bulan berjalan (misal Oktober 2025),
lalu ambil data **3 bulan sebelumnya** → Jul, Ags, Sep.

---

## 💻 **6️⃣ Kode TypeScript (Prisma + logika sliding window)**

```ts
import { prisma } from "@/utils/db/prisma";

interface ForecastResult {
  subDistrict: string;
  forecastNextMonth: number;
  monthsUsed: string[];
}

/**
 * Prediksi bulan berikutnya berdasarkan 3 bulan terakhir
 * Sliding window otomatis geser sesuai bulan berjalan
 */
export async function forecastKecamatan(cityCode: string): Promise<ForecastResult[]> {
  const now = new Date();

  // Awal bulan ini & awal 3 bulan sebelumnya
  const startOfThisMonth = new Date(now.getFullYear(), now.getMonth(), 1);
  const startOf3MonthsAgo = new Date(now.getFullYear(), now.getMonth() - 3, 1);

  // Ambil hanya data dari 3 bulan terakhir
  const data = await prisma.monthlySales.findMany({
    where: {
      cityCode,
      month: {
        gte: startOf3MonthsAgo, // >= awal bulan ke-3 sebelumnya
        lt: startOfThisMonth,   // < awal bulan ini
      },
    },
    orderBy: { month: "asc" },
  });

  // Kelompokkan per kecamatan
  const grouped = data.reduce<Record<string, { month: Date; total: number }[]>>((acc, d) => {
    if (!acc[d.subDistrict]) acc[d.subDistrict] = [];
    acc[d.subDistrict].push({ month: d.month, total: d.totalSales });
    return acc;
  }, {});

  const weights = [0.1, 0.3, 0.6];
  const results: ForecastResult[] = [];

  // Hitung forecast
  for (const [subDistrict, list] of Object.entries(grouped)) {
    if (list.length < 3) continue; // butuh minimal 3 bulan

    const last3 = list.sort((a, b) => a.month.getTime() - b.month.getTime()).slice(-3);
    const forecast = last3.reduce((sum, d, i) => sum + d.total * weights[i], 0);

    results.push({
      subDistrict,
      forecastNextMonth: Math.round(forecast),
      monthsUsed: last3.map((d) => d.month.toISOString().slice(0, 7)),
    });
  }

  return results;
}
```

---

## 📊 **7️⃣ Contoh hasil output**

Misal sistem dijalankan 8 Oktober 2025:

| subDistrict | Forecast Oktober | Data Bulan Dipakai |
| ----------- | ---------------- | ------------------ |
| Boja        | 172              | Jul, Ags, Sep      |
| Kaliwungu   | 151              | Jul, Ags, Sep      |
| Patebon     | 139              | Jul, Ags, Sep      |

Output (JavaScript):

```ts
[
  { subDistrict: "Boja", forecastNextMonth: 172, monthsUsed: ["2025-07","2025-08","2025-09"] },
  { subDistrict: "Kaliwungu", forecastNextMonth: 151, monthsUsed: ["2025-07","2025-08","2025-09"] },
]
```

---

## 💬 **8️⃣ Kesimpulan**

| Aspek             | Nilai                                                             |
| ----------------- | ----------------------------------------------------------------- |
| 🔹 Model          | Sliding window (3 bulan terakhir)                                 |
| 🔹 Rumus          | 0.6×Last + 0.3×Prev + 0.1×Prev2                                   |
| 🔹 Akurasi (MAPE) | ±7–15% tergantung kestabilan pasar                                |
| 🔹 Kelebihan      | Adaptif, ringan, cocok untuk sistem real-time                     |
| 🔹 Kekurangan     | Tidak menangkap pola musiman tahunan (perlu model tambahan nanti) |

---

## 🚀 **Next step (opsional untuk sistemmu)**

1. Simpan hasil prediksi ke tabel `ForecastResult`.
2. Jalankan `forecastKecamatan()` tiap awal bulan via cron job.
3. Bandingkan hasil dengan aktual tiap akhir bulan → hitung MAPE otomatis.
4. Tambahkan *long-term model* (12 bulan) untuk memperbaiki akurasi musiman.

---

Apakah kamu mau saya bantu tambahkan tahap **penyimpanan hasil forecast ke DB + auto-MAPE checker** biar nanti sistemmu bisa menilai seberapa akurat prediksinya setiap bulan?
