# 🧠 SQL for Exploratory Data Analysis
---
📊 **Exploratory Data Analysis (EDA)** menggunakan **SQL** pada dataset penjualan dari Blinkit sebuah platform e-commerce kebutuhan sehari-hari.

Repositori ini berisi kumpulan query SQL untuk menganalisis berbagai aspek bisnis seperti:
- 📦 Kinerja produk dan kategori
- 👥 Segmentasi pelanggan
- 📈 Tren penjualan bulanan
- 💬 Umpan balik pelanggan

### 🛠 Tools
- **SQL Server Management Studio (SSMS)**
- Dataset: [Blinkit Sales Dataset – Kaggle](https://www.kaggle.com/datasets/akxiit/blinkit-sales-dataset)

### 📌 Tujuan
Mendemonstrasikan bagaimana SQL digunakan untuk eksplorasi data dalam konteks bisnis nyata:
- Mencari insight untuk pengambilan keputusan
- Menjawab pertanyaan-pertanyaan bisnis menggunakan data

## 📦 Kinerja produk dan kategori
---
1. Menampilkan kategori produk yang menghasilkan pendapatan tertinggi dan terendah
```sql
WITH sales_category AS(
    SELECT
	category,
	SUM(quantity * price) AS total_revenue,
	SUM(quantity) AS total_item
    FROM orders AS o
    INNER JOIN order_items AS oi
    ON o.order_id = oi.order_id
    INNER JOIN products AS p
    ON oi.product_id = p.product_id
    GROUP BY category
)
SELECT TOP 5 * FROM sales_category
ORDER BY 2 DESC
```
### **Output:**
TOP 5 kategori produk yang mendapatkan pendapatan tertinggi
| category              | total_revenue | total_item |
|------------------------|---------------|-------------|
| Dairy & Breakfast      | 58,707,763    | 1,114       |
| Pharmacy               | 55,974,393    | 973         |
| Fruits & Vegetables    | 53,955,458    | 966         |
| Household Care         | 41,949,848    | 1,078       |
| Pet Care               | 40,990,733    | 1,003       |

TOP 5 kategori produk yang mendapatkan pendapatan terendah
| category               | total_revenue    | total_orders |
|------------------------|------------|--------|
| Instant & Frozen Food  | 29,209,697 | 742    |
| Cold Drinks & Juices   | 31,599,928 | 758    |
| Grocery & Staples      | 33,853,141 | 895    |
| Baby Care              | 34,822,718 | 655    |
| Personal Care          | 35,018,297 | 887    |

🔍 **Insights:**
- 🥞 **Dairy & Breakfast** mencetak pendapatan tertinggi (Rp 58 juta+) dan volume item tertinggi (**1,114 produk**), menunjukkan konsistensi dalam kontribusi terhadap pendapatan.
- ❄️ **Instant & Frozen Food** dan **Cold Drinks & Juices** mendapatkan pendapatan terendah (Rp 29 juta+) dari 11 kategori.

2. Produk terlaris berdasarkan kuantitas penjualan
```sql
WITH sales_product AS(
    SELECT
	p.product_name,
        p.category AS product_category,
	SUM(quantity * price) AS total_revenue,
	SUM(quantity) AS total_item
    FROM orders AS o
    INNER JOIN order_items AS oi
    ON o.order_id = oi.order_id
    INNER JOIN products AS p
    ON oi.product_id = p.product_id
    GROUP BY p.product_name, p.category
)
SELECT TOP 5 * FROM sales_product
ORDER BY total_revenue DESC
```
### **Output**
TOP 5 produk terlaris dalam pendapatan tertinggi
| product_name    | category        | total_revenue | total_item |
|----------------|------------------|----------------|-------------|
| Vitamins       | Pharmacy         | 26,082,201     | 380         |
| Pet Treats     | Pet Care         | 20,469,545     | 473         |
| Toilet Cleaner | Household Care   | 19,983,748     | 430         |
| Cough Syrup    | Pharmacy         | 17,338,101     | 373         |
| Dish Soap      | Household Care   | 15,969,544     | 397         |

TOP 5 produk dalam pendapatan terendah
| product_name    | category        | total_revenue | total_item |
|------------------|------------------------|-----------|--------|
| Lemonade         | Cold Drinks & Juices   | 1,497,780 | 45     |
| Cereal           | Dairy & Breakfast      | 1,762,550 | 86     |
| Rice             | Grocery & Staples      | 2,013,980 | 109    |
| Spinach          | Fruits & Vegetables    | 2,523,360 | 40     |
| Instant Noodles  | Instant & Frozen Food  | 3,280,164 | 73     |

🔍 **Insights:**
- 💊 **Produk kategori Pharmacy (Vitamins & Cough Syrup)** mendominasi pendapatan tertinggi meskipun jumlah item tidak paling banyak. Hal ini menunjukkan bahwa **harga per item tinggi dan permintaan stabil**.
- 🐶 **Pet Treats** menjadi produk dengan item terbanyak di antara top 5, menunjukkan permintaan kuat dari pemilik hewan peliharaan.
- 🥬 Produk seperti **Spinach dan Lemonade** meskipun relevan dalam kebutuhan harian, memiliki **pendapatan rendah**, kemungkinan karena **harga per item rendah**.

3. Brand terlaris dalam pendapatan
```sql
SELECT TOP 5
    p.brand,
    SUM(oi.unit_price * oi.quantity) AS total_revenue
FROM products AS p
JOIN order_items AS oi
ON p.product_id = oi.product_id
GROUP BY p.brand
ORDER BY total_revenue DESC
```
### **Output**
| brand        | total_revenue   |
|----------------|-----------|
| Karnik PLC     | 6,521,270 |
| Mandal-Kar     | 5,646,465 |
| Roy-Char       | 5,518,294 |
| Sundaram Inc   | 5,183,035 |
| Gole-Doshi     | 5,179,096 |

🔍 **Insights:**
- 🏆 **Karnik PLC** menjadi brand dengan pendapatan tertinggi (6,5 juta+), unggul lebih dari **900 ribu** dibanding posisi kedua. Hal ini menunjukkan **dominasi pasar** atau keberhasilan strategi distribusi/produk.
- 📈 Brand lainnya seperti **Mandal-Kar**, **Roy-Char**, dan **Sundaram Inc** memiliki pendapatan yang relatif berdekatan, menandakan persaingan yang ketat di bawah pemimpin pasar.
- ⚖️ **Selisih antara brand ke-4 dan ke-5 sangat kecil** (hanya sekitar 4 ribu), menunjukkan performa serupa dan **potensi saling salip** di masa depan.


4. Stok barang yang mengalami kerusakan di gudang
```sql
WITH stock_product AS(
SELECT 
    p.product_name,
    p.category,
    p.shelf_life_days,
    p.min_stock_level,
    p.max_stock_level,
    SUM(i.stock_received) AS total_stock_received,
    SUM(i.damaged_stock) AS total_damaged_stock
FROM inventory AS i
JOIN products AS p
ON i.product_id = p.product_id
GROUP BY p.product_name, p.category, p.shelf_life_days, p.min_stock_level, p.max_stock_level
)
SELECT category, SUM(total_damaged_stock) AS damaged_stock FROM stock_product
GROUP BY category
ORDER BY damaged_stock DESC
```
### **Output**
| category                 | total_damaged_stock  |
|--------------------------|-------------|
| Dairy & Breakfast        | 8,890       |
| Household Care           | 8,210       |
| Snacks & Munchies        | 8,144       |
| Fruits & Vegetables      | 8,120       |
| Pet Care                 | 7,528       |
| Personal Care            | 7,524       |
| Pharmacy                 | 7,366       |
| Grocery & Staples        | 7,144       |
| Cold Drinks & Juices     | 6,604       |
| Instant & Frozen Food    | 5,910       |
| Baby Care                | 4,828       |

🔍 **Insight:**
- 🚨 **Dairy & Breakfast** memiliki jumlah *damaged stock* tertinggi (**8.890 unit**), kemungkinan besar karena **produk mudah rusak dan umur simpan pendek**.
- 🧴 Kategori seperti **Household Care**, **Personal Care**, dan **Pharmacy** — yang umumnya tidak mudah rusak, memiliki jumlah kerusakan cukup tinggi, menandakan potensi adanya **masalah dalam penanganan/logistik**.

## 👥 Segmentasi pelanggan
---
5. Jumlah customer berdasarkan segmentasi pelanggan
```sql
SELECT 
    customer_segment,
    COUNT(*) AS total_customer
FROM customers
GROUP BY customer_segment
```
### **Output**
| customer_segment | toal_customer |
|------------------|-------|
| New              | 628   |
| Premium          | 633   |
| Inactive         | 600   |
| Regular          | 639   |

🔍 **Insight:**
- ⚖️ Distribusi pelanggan cukup **merata di semua segmen**, dengan selisih kecil antara yang tertinggi (Regular: 639) dan terendah (Inactive: 600).
- 🆕 Segmen **New** cukup besar (**628 pelanggan**), menunjukkan potensi pertumbuhan namun keberlanjutannya tergantung pada retensi.
- 💤 **600 pelanggan Inactive** menunjukkan adanya kehilangan engagement, yang bisa berdampak pada pendapatan jangka panjang.
- 💎 **Premium** memiliki jumlah hampir setara dengan Regular, menandakan **peluang upsell dan loyalitas** sudah mulai terbentuk.

6. Customer yang sering berbelanja
```sql
SELECT TOP 10
    c.customer_id,
    c.customer_name,
    c.email,
    SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM customers AS c
JOIN orders AS o
ON c.customer_id = o.customer_id
JOIN order_items AS oi
ON o.order_id= oi.order_id
GROUP BY c.customer_name, c.customer_id, c.email
ORDER BY total_revenue DESC
```
### **Output**
| customer_id | customer_name | email                              | total_revenue  |
|----------|------------------|------------------------------------|----------------|
| 25128143 | Odika Kannan     | kondaadya@example.org              | 1,053,339      |
| 77869660 | Nidhi Sha        | ewali@example.org                  | 1,011,575      |
| 4597433  | Lipika Kumer     | faras46@example.org                | 955,369        |
| 33331259 | Leena Deol       | nitesh33@example.com               | 940,377        |
| 14472401 | Ayaan Rege       | dubeyekanta@example.com            | 927,573        |
| 85561151 | Zinal Dhawan     | gulatiedhitha@example.org          | 926,859        |
| 22800586 | Jacob Lad        | ramakrishnanchandresh@example.net  | 923,724        |
| 34668988 | Wishi Varughese  | damyanti67@example.net             | 916,073        |
| 13760839 | Anvi Savant      | yashvi72@example.org               | 911,399        |
| 12832151 | Ekavir Bhalla    | ksehgal@example.net                | 884,111        |

🔍 **Insight:**
- 💰 Semua pelanggan dalam daftar menghasilkan pendapatan di atas **Rp 880.000**, dengan **top spender** adalah **Odika Kannan** (Rp 1.053.339).
- 🧍‍♀️ Terdapat **10 pelanggan loyal** yang secara kolektif memberikan kontribusi besar terhadap pendapatan — ini adalah segmen **High Value Customers (HVC)**.

## 📈 Tren penjualan bulanan
---
7. Customer baru dalam setiap bulan
```sql
SELECT 
    FORMAT(registration_date, 'yyyy-MM') AS month,
    COUNT(*) AS new_customers
FROM customers
WHERE customer_segment = 'New'
GROUP BY FORMAT(registration_date, 'yyyy-MM')
ORDER BY new_customers DESC
```
### **Output**
| month     | new_customers |
|-----------|----------------|
| 2024-03   | 47             |
| 2023-12   | 40             |
| 2024-05   | 38             |
| 2023-11   | 38             |
| 2023-08   | 37             |
| 2023-10   | 37             |
| 2024-09   | 37             |
| 2024-10   | 31             |
| 2023-04   | 30             |
| 2024-08   | 30             |
| 2024-01   | 30             |
| 2024-04   | 29             |
| 2023-09   | 29             |
| 2024-02   | 28             |
| 2024-06   | 27             |
| 2024-07   | 27             |
| 2023-05   | 25             |
| 2023-06   | 25             |
| 2023-03   | 21             |
| 2023-07   | 19             |
| 2024-11   | 3              |

🔍 **Insight:**
- 📈 **Maret 2024 (2024-03)** mencatat jumlah pelanggan baru tertinggi (47 pelanggan) → potensi ada kampanye/aktivitas promosi yang berhasil saat itu.
- 📉 **Bulan November 2024 (2024-11)** memiliki angka terendah (hanya 3 pelanggan baru) → kemungkinan disebabkan oleh **kurangnya aktivitas promosi atau faktor eksternal (musiman, persaingan, dll)**.

## 💬 Umpan balik pelanggan
---
8. Analisis berdasarkan sentimen pelanggan
```sql
WITH rating_new AS(
    SELECT 
        rating,
        CASE 
            WHEN rating BETWEEN 1 AND 2 THEN 'Negative'
            WHEN rating BETWEEN 4 AND 5 THEN 'Positive'
            WHEN rating = 3 THEN 'Neutral'
        END sentiment
    FROM customer_feedback
)
SELECT 
    sentiment,
    COUNT(*) * 100.0 / (SELECT COUNT(*) FROM customer_feedback) AS percentage
FROM rating_new
GROUP BY sentiment
```
### **Output**
| sentiment | percentage |
|-----------|----------------|
| Negative  | 21.56          |
| Neutral   | 27.96          |
| Positive  | 50.48          |

🔍 **Insight:**
- ✅ **Mayoritas ulasan pelanggan bersentimen positif (50.48%)**, menandakan bahwa lebih dari separuh pelanggan puas terhadap layanan/produk.
- ⚖️ **Sentimen netral cukup signifikan (27.96%)**, menunjukkan adanya kelompok pelanggan yang merasa pengalaman mereka biasa saja tidak terlalu buruk atau bagus.
- ❗ **Sentimen negatif mencapai 21.56%**, cukup besar dan patut diperhatikan karena berpotensi memengaruhi reputasi dan loyalitas pelanggan.

9. Product yang mendapatkan umpan balik positif dan umpan balik negatif
```sql
SELECT TOP 5
    p.product_name,
    COUNT(*) AS negative_feedback
FROM customer_feedback AS cf
JOIN order_items AS oi
ON cf.order_id = oi.order_id
JOIN products AS p
ON oi.product_id = p.product_id
WHERE cf.sentiment = 'Negative'
GROUP BY p.product_name
ORDER BY negative_feedback DESC
```
### **Output**
TOP 5 produk yang menerima umpan balik positif
| product         | positive_feedback |
|------------------|----------|
| Pet Treats       | 121      |
| Toilet Cleaner   | 110      |
| Cough Syrup      | 108      |
| Vitamins         | 95       |
| Dish Soap        | 94       |

TOP 5 produk yang menerima umpan balik negatif
| product         | negative_feedback |
|------------------|----------|
| Pet Treats       | 44       |
| Baby Wipes       | 41       |
| Lotion           | 41       |
| Dish Soap        | 39       |
| Toilet Cleaner   | 38       |

🔍 **Insight:**
* 🥇 **Pet Treats** mendominasi baik dalam **feedback positif (121)** maupun **negatif (44)**. Produk ini sangat populer, tapi juga punya tantangan kualitas/pengalaman terhadap pelanggan.
* 🧼 **Toilet Cleaner** dan **Dish Soap** masuk dalam daftar **feedback positif dan negatif**, menandakan **inkonsistensi pengalaman pelanggan**.
* 🤒 **Cough Syrup** dan **Vitamins** mendapatkan banyak feedback positif, menandakan kepercayaan terhadap produk kesehatan cukup tinggi.

## **✅ Rekomendasi**
---
1. Kategori 
* 🎯 **Fokus promosi** dan bundling di kategori **Personal Care** dan **Cold Drinks & Juices** untuk mendorong repeat order.
* 💰 **Evaluasi margin** dan tingkatkan stok untuk kategori **Baby Care**, karena walau jumlah item lebih sedikit, revenue dan order tinggi.
* 📈 **Perluas varian dan supplier** untuk **Fruits & Vegetables**, karena volume tinggi menunjukkan potensi untuk diversifikasi produk lokal.
* 📦 Pertimbangkan **peningkatan logistik cold storage** untuk menunjang performa **Instant & Frozen Food**.
* 📊 Lakukan analisis lanjut pada kategori **Grocery & Staples** untuk mengidentifikasi produk dengan performa terbaik guna dioptimalkan dalam kampanye penjualan.

2. Produk
* 💰 **Fokuskan promosi premium** untuk kategori **Pharmacy** karena kontribusi per produk tinggi; pertimbangkan juga bundling paket kesehatan.
* 🧽 **Pertahankan stok dan iklankan produk kebutuhan rumah tangga** (Household Care), karena performanya baik dari segi volume dan nilai penjualan.
* 🐾 Untuk **Pet Treats**, bisa diperluas ke produk pelengkap lain (mainan, perawatan) karena basis pelanggan sudah kuat.
* 📉 **Evaluasi strategi penetapan harga atau bundling** untuk produk dengan pendapatan rendah (Lemonade, Cereal, Rice), agar nilai transaksi per pembelian meningkat.
* 📦 **Spinach dan produk segar lainnya** dapat ditingkatkan melalui penawaran langganan atau diskon pembelian rutin untuk meningkatkan volume penjualan.

3. Brand
* 🧠 **Pertahankan keunggulan Karnik PLC** dengan memperkuat kampanye loyalitas dan promosi produk unggulan agar tetap jadi top performer.
* 🚀 Untuk brand pesaing (Mandal-Kar, Roy-Char, Sundaram Inc), lakukan **differensiasi produk atau layanan tambahan** agar bisa menyalip brand dominan.
* 📊 Lakukan **analisis mendalam per kategori produk per brand** untuk mengidentifikasi lini produk paling menguntungkan dari masing-masing brand.
* 🔍 **Monitor brand Gole-Doshi dan Sundaram Inc** secara berkala karena posisinya sangat dekat — strategi kecil bisa berdampak besar pada peringkat mereka.

4. Stok barang rusak
* 🧊 **Perkuat rantai pendingin (cold chain)** dan SOP penyimpanan untuk **Dairy, Fruits & Vegetables** untuk mengurangi kerusakan akibat suhu dan kelembapan.
* 📦 Audit dan pelatihan ulang staf logistik khususnya untuk **Household Care dan Personal Care**, karena produk non-perishable pun mengalami kerusakan tinggi.
* 🔍 Lakukan **analisis kerusakan berdasarkan gudang atau jalur distribusi** untuk mengidentifikasi titik rawan.
* 🚚 Evaluasi kembali metode pengemasan dan transportasi terutama untuk **produk sensitif**, agar dapat **mengurangi kerugian dan meningkatkan efisiensi operasional**.
* ✅ Terapkan **best practice dari Baby Care** ke kategori lain karena terbukti lebih aman dari kerusakan.

5. Pelanggan
* 🔁 **Fokuskan upaya retensi** pada segmen **New**, misalnya lewat welcome offers, edukasi produk, atau email onboarding untuk mendorong mereka menjadi pelanggan aktif.
* 🚨 Lakukan **aktivasi ulang pelanggan Inactive** dengan kampanye khusus seperti diskon comeback, rekomendasi produk personal, atau reminder keranjang belanja.
* 🌟 Manfaatkan basis pelanggan **Premium** untuk program referral, testimoni, dan program loyalitas — karena mereka berpotensi menjadi brand ambassador.
* 📈 Analisis lebih lanjut perilaku pelanggan **Regular** untuk mendorong transisi ke Premium dengan penawaran eksklusif atau langganan berbayar.

6. TOP customer
* 🎁 Buat **program loyalitas khusus** (reward tier atau diskon eksklusif) untuk pelanggan dengan total belanja tinggi guna menjaga retensi.
* ✉️ Kirim **email marketing personalisasi** (produk rekomendasi, promo ulang tahun, notifikasi stok ulang) untuk memperkuat engagement.
* 🛍️ Tawarkan **early access** ke produk baru atau diskon flash untuk pelanggan top spender sebagai bentuk apresiasi.
* 📊 Lakukan **analisis lebih lanjut** pada perilaku belanja pelanggan ini (frekuensi, kategori favorit) untuk mendesain **kampanye upsell/cross-sell** yang efektif.

7. New customer
* 🗓️ **Analisis strategi pemasaran di Maret dan Desember** untuk mengidentifikasi apa yang berhasil dan bisa direplikasi di bulan lain.
* 📢 Tingkatkan promosi menjelang bulan-bulan yang historisnya **rendah (seperti Juli dan November)** agar pertumbuhan lebih merata.
* 🧪 Lakukan **A/B testing** pada saluran akuisisi pelanggan (iklan, referral, diskon onboarding) untuk menemukan metode paling efektif.
* 🎯 Bangun **retensi funnel** dari pelanggan baru menjadi pelanggan aktif/loyal, terutama di bulan dengan pertumbuhan tinggi.
* 📅 Jadwalkan kampanye besar di kuartal awal dan akhir tahun untuk **mengoptimalkan momentum tren pertumbuhan** yang sudah terbukti.

8. Umpan balik
* 📌 **Analisis lebih lanjut isi ulasan negatif** untuk mengidentifikasi masalah utama (misalnya: pengiriman, kualitas produk, layanan).
* ✨ Tingkatkan kualitas di area yang sering disebutkan dalam ulasan netral agar bisa diubah menjadi ulasan positif.
* 💬 **Respon aktif dan cepat terhadap ulasan negatif** di berbagai platform agar pelanggan merasa didengar dan dihargai.
* 🔍 Lakukan segmentasi sentimen berdasarkan kategori produk atau wilayah untuk strategi perbaikan yang lebih terarah.
* 🎉 Promosikan ulasan positif sebagai **bukti sosial (social proof)** di media sosial dan kampanye pemasaran.

9. Barang
* 🔍 **Audit kualitas untuk produk dengan feedback tinggi (positif & negatif)** seperti Pet Treats, Dish Soap, dan Toilet Cleaner guna memastikan konsistensi.
* 📣 Promosikan produk dengan dominasi **feedback positif** (Cough Syrup, Vitamins) sebagai produk unggulan atau best-rated.
* 🚨 Lakukan evaluasi menyeluruh pada produk dengan **feedback negatif tinggi** (Baby Wipes, Lotion), termasuk pengemasan, bahan, dan instruksi penggunaan.
* 🗣️ Kumpulkan dan analisis **feedback lebih dalam dari pelanggan** (review teks) untuk mencari akar penyebab ketidakpuasan.
* 📦 Gunakan hasil analisis ini untuk perbaikan produk, pelatihan staf CS, atau edukasi pelanggan tentang cara penggunaan produk tertentu.

---
Feel free to fork, clone, or contribute! 🚀
