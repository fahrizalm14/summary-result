🔥 Siap — kita rangkum ulang dengan **versi lengkap final**:
seluruh alur dari **data mentah → prediksi pasar MTD → pangsa pasar dealer**,
beserta **cara hitung**, **MAPE (akurasi)**, dan **kode TypeScript Prisma** yang merefleksikan semua tahap itu.

---

# 🧾 **RINGKASAN FINAL — PREDIKSI PASAR MTD & PANGSA PASAR DEALER**

## 🎯 **Tujuan sistem**

Membangun modul analitik yang bisa:

1. Memperkirakan **total pasar (Market Forecast)** per kecamatan berdasarkan data 3 bulan terakhir,
2. Mengestimasi **pasar MTD (Month-to-Date)** bulan berjalan,
3. Menghitung **pangsa pasar dealer (Market Share)** berdasarkan data dealer aktual MTD.

---

## 🧩 **1️⃣ Rumus dasar prediksi pasar (forecast)**

Kita gunakan **Weighted Moving Average (Sliding Window 3 bulan)**:

[
\text{Forecast}*{bulan+1} = 0.6×Sales*{bulan-1} + 0.3×Sales_{bulan-2} + 0.1×Sales_{bulan-3}
]

Artinya:

* Bobot tinggi pada bulan terbaru (0.6)
* Dua bulan sebelumnya digunakan untuk menstabilkan tren (0.3 dan 0.1)
* Total bobot = 1

📌 Contoh:

| Bulan     | Penjualan |
| --------- | --------- |
| Juli      | 193       |
| Agustus   | 131       |
| September | 173       |

[
Prediksi\ Okt = 0.6×173 + 0.3×131 + 0.1×193 = 162.4 \text{ unit}
]

---

## 📅 **2️⃣ Konversi ke prediksi MTD (Month-to-Date)**

Kalau kamu ingin tahu pasar **sampai hari tertentu**, misalnya **14 Agustus**,
ambil proporsi hari berjalan terhadap total hari bulan ini.

[
Prediksi_{MTD} = Prediksi_{bulanan} × \frac{\text{hari berjalan}}{\text{jumlah hari bulan}}
]

Contoh:

* Prediksi pasar Agustus penuh: 160 unit
* Hari ini: 14 Agustus
* Bulan Agustus = 31 hari

[
Prediksi_{MTD} = 160 × \frac{14}{31} = 72.3
]

➡️ Artinya: sampai tanggal 14, pasar diperkirakan 72 unit di kecamatan tersebut.

---

## 🧮 **3️⃣ Hitung pangsa pasar dealer**

Setelah tahu pasar MTD, tinggal bandingkan dengan penjualan dealer aktual MTD:

[
Share_{dealer} = \frac{DealerSales_{MTD}}{PrediksiPasar_{MTD}} × 100%
]

Contoh:

| Kecamatan  | Dealer MTD | Prediksi Pasar MTD | Pangsa (%) |
| ---------- | ---------- | ------------------ | ---------- |
| Boja       | 6          | 86                 | 7.0        |
| Ngampel    | 9          | 36                 | 25.0       |
| Ringinarum | 7          | 39                 | 17.9       |

➡️ **Interpretasi:**

* Ngampel 25% → dealer dominan
* Boja 7% → cukup
* Weleri 1.5% → butuh promosi

---

## 📊 **4️⃣ Mengukur akurasi (MAPE)**

Untuk mengevaluasi prediksi bulan sebelumnya:

[
MAPE = \frac{1}{n} \sum_{i=1}^{n} \left| \frac{Actual_i - Forecast_i}{Actual_i} \right| × 100%
]

Contoh hasil:

| Bulan     | Aktual | Prediksi | Error      |
| --------- | ------ | -------- | ---------- |
| Juli      | 150    | 140      | 6.7%       |
| Agustus   | 180    | 190      | 5.5%       |
| September | 175    | 165      | 5.7%       |
| **MAPE**  |        |          | **5.9% ✅** |

🟢 MAPE <10% = akurasi bagus
🟡 10–20% = wajar (fluktuatif)
🔴 >25% = prediksi perlu diperbaiki

---

## 🧠 **5️⃣ Kenapa data 3 bulan relevan**

| Alasan                                            | Dampak                                               |
| ------------------------------------------------- | ---------------------------------------------------- |
| 🔹 Pasar motor cepat berubah (stok, promo, cuaca) | Data lama jadi kurang relevan                        |
| 🔹 3 bulan terakhir mencerminkan momentum aktual  | Responsif terhadap tren                              |
| 🔹 Data lama tetap disimpan                       | Berguna untuk pola musiman (Ramadhan, akhir tahun)   |
| 🔹 Bobot fleksibel                                | Bisa dikalibrasi ulang tiap 6 bulan berdasarkan MAPE |

---

## 💻 **6️⃣ Kode TypeScript (Prisma + logika lengkap)**

```ts
import { prisma } from "@/utils/db/prisma";

/**
 * Prediksi pasar & hitung pangsa dealer berdasarkan 3 bulan terakhir
 */
export async function forecastAndMarketShare(cityCode: string, dealerSalesMap: Record<string, number>, dayOfMonth = 14) {
  const now = new Date();
  const startOfThisMonth = new Date(now.getFullYear(), now.getMonth(), 1);
  const startOf3MonthsAgo = new Date(now.getFullYear(), now.getMonth() - 3, 1);

  // Ambil data 3 bulan terakhir
  const data = await prisma.monthlySales.findMany({
    where: {
      cityCode,
      month: {
        gte: startOf3MonthsAgo,
        lt: startOfThisMonth,
      },
    },
    orderBy: { month: "asc" },
  });

  // Group by kecamatan
  const grouped = data.reduce<Record<string, { month: Date; total: number }[]>>((acc, d) => {
    if (!acc[d.subDistrict]) acc[d.subDistrict] = [];
    acc[d.subDistrict].push({ month: d.month, total: d.totalSales });
    return acc;
  }, {});

  const weights = [0.1, 0.3, 0.6];
  const results: any[] = [];
  const daysInMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
  const progress = dayOfMonth / daysInMonth;

  // Hitung prediksi per kecamatan
  for (const [subDistrict, list] of Object.entries(grouped)) {
    if (list.length < 3) continue;
    const last3 = list.sort((a, b) => a.month.getTime() - b.month.getTime()).slice(-3);
    const predMonth = last3.reduce((sum, d, i) => sum + d.total * weights[i], 0);
    const predMTD = predMonth * progress;

    const dealerSales = dealerSalesMap[subDistrict.toUpperCase()] ?? 0;
    const share = dealerSales > 0 ? (dealerSales / predMTD) * 100 : 0;

    results.push({
      subDistrict,
      forecastMonth: Math.round(predMonth),
      forecastMTD: Math.round(predMTD),
      dealerSales,
      sharePercent: Math.round(share * 10) / 10,
      monthsUsed: last3.map((d) => d.month.toISOString().slice(0, 7)),
    });
  }

  return results.sort((a, b) => b.forecastMonth - a.forecastMonth);
}
```

### 🔹 **Contoh cara pakai**

```ts
const dealerSalesMap = {
  BOJA: 6,
  KALIWUNGU: 7,
  PATEBON: 9,
  NGAMPEL: 9,
  RINGINARUM: 7,
  GEMUH: 12,
};

const result = await forecastAndMarketShare("3324", dealerSalesMap, 14);
console.table(result);
```

---

## 📊 **7️⃣ Contoh hasil output**

| Kecamatan  | Pred Bulan | Pred MTD (s/d 14) | Dealer MTD | Pangsa (%) |
| ---------- | ---------- | ----------------- | ---------- | ---------- |
| Boja       | 172        | 86                | 6          | 7.0        |
| Patebon    | 140        | 70                | 9          | 12.9       |
| Ngampel    | 72         | 36                | 9          | 25.0       |
| Ringinarum | 78         | 39                | 7          | 17.9       |
| Gemuh      | 122        | 61                | 12         | 19.6       |
| Weleri     | 135        | 67                | 1          | 1.5        |

➡️ Total kota Kendal:

* Prediksi pasar MTD ≈ **1,038 unit**
* Dealer total MTD ≈ **102 unit**
* Pangsa pasar total ≈ **9.8%**

---

## 📈 **8️⃣ Cara mengevaluasi akurasi otomatis (MAPE)**

Jika kamu sudah punya data aktual bulan Agustus,
kamu bisa tambahkan logika evaluasi sederhana:

```ts
function calculateMAPE(actual: number[], forecast: number[]) {
  const errors = actual.map((a, i) => Math.abs((a - forecast[i]) / a));
  const mape = (errors.reduce((s, e) => s + e, 0) / errors.length) * 100;
  return Math.round(mape * 10) / 10;
}
```

---

## ✅ **Kesimpulan Akhir**

| Aspek                 | Hasil                                             |
| --------------------- | ------------------------------------------------- |
| **Metode**            | Weighted Sliding Window (3 bulan)                 |
| **Rumus**             | 0.6×Terbaru + 0.3×Sebelum + 0.1×2 bulan lalu      |
| **Output**            | Prediksi pasar bulan penuh + MTD                  |
| **Dealer Share**      | DealerSales / PrediksiPasarMTD × 100              |
| **Akurasi (MAPE)**    | ±7–15% tergantung kestabilan                      |
| **Kelebihan**         | Adaptif, cepat, cocok untuk data bulanan dealer   |
| **Kapan recalibrate** | Tiap 6 bulan (cek error MAPE dan sesuaikan bobot) |

---

Kalau kamu mau, saya bisa bantu **lanjutkan modul `autoForecastJob()`**
yang menjalankan fungsi ini tiap awal bulan, menyimpan hasil ke DB (`ForecastResult`),
dan menghitung MAPE bulan lalu secara otomatis — biar sistem kamu belajar terus.
Apakah mau saya buatkan lanjutan blueprint-nya?
