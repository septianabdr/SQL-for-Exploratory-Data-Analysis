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
- Mengukur KPI penting
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
|-----------------------|---------------|--------------|
| Fruits & Vegetables   | 53,955,458    | 966          |
| Personal Care         | 35,018,297    | 887          |
| Grocery & Staples     | 33,853,141    | 895          |
| Cold Drinks & Juices  | 31,599,928    | 758          |
| Instant & Frozen Food | 29,209,697    | 742          |

TOP 5 kategori produk yang mendapatkan pendapatan tertinggi
| category               | total_revenue    | total_orders |
|------------------------|------------|--------|
| Instant & Frozen Food  | 29,209,697 | 742    |
| Cold Drinks & Juices   | 31,599,928 | 758    |
| Grocery & Staples      | 33,853,141 | 895    |
| Baby Care              | 34,822,718 | 655    |
| Personal Care          | 35,018,297 | 887    |

**📊 Insights:**
* 🔝 **Personal Care** mencetak pendapatan tertinggi (Rp 35 juta+) dan volume item tinggi (**887 produk**), menunjukkan konsistensi dalam kontribusi terhadap pendapatan.
* 🍼 **Baby Care** tidak termasuk dalam 5 besar dari sisi jumlah item, namun berhasil masuk top 5 dari sisi **pendapatan & jumlah pesanan**, menandakan **harga per produk relatif tinggi**.
* 🥬 **Fruits & Vegetables** memiliki **pendapatan tertinggi secara keseluruhan** dan juga jumlah item terbanyak (**966 item**), menunjukkan **keragaman produk tinggi**, meskipun harga per item bisa jadi lebih rendah.
* ❄️ **Instant & Frozen Food** meskipun jumlah item lebih sedikit, tetap memiliki performa tinggi dalam jumlah pesanan dan pendapatan, menunjukkan **permintaan stabil**.
* 🥤 **Cold Drinks & Juices** konsisten di kedua peringkat — memperkuat bahwa produk ini adalah kebutuhan rutin dengan **rotasi cepat**.

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

TOP 5 produk terlaris dalam pendapatan terendah
| product_name    | category        | total_revenue | total_item |
|------------------|------------------------|-----------|--------|
| Lemonade         | Cold Drinks & Juices   | 1,497,780 | 45     |
| Cereal           | Dairy & Breakfast      | 1,762,550 | 86     |
| Rice             | Grocery & Staples      | 2,013,980 | 109    |
| Spinach          | Fruits & Vegetables    | 2,523,360 | 40     |
| Instant Noodles  | Instant & Frozen Food  | 3,280,164 | 73     |

**📊 Insights:**
- 
- 


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

**📊 Insights:**
- 

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

**📊 Insight:**
- 

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

**📊 Insight:**
- 


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

**📊 Insight:**
- 

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

**📊 Insight:**
- 

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

**📊 Insight:**
-

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

**📊 Insight:**
-

## **✅ Rekomendasi**
---
1. Produk 
* 🎯 **Fokus promosi** dan bundling di kategori **Personal Care** dan **Cold Drinks & Juices** untuk mendorong repeat order.
* 💰 **Evaluasi margin** dan tingkatkan stok untuk kategori **Baby Care**, karena walau jumlah item lebih sedikit, revenue dan order tinggi.
* 📈 **Perluas varian dan supplier** untuk **Fruits & Vegetables**, karena volume tinggi menunjukkan potensi untuk diversifikasi produk lokal.
* 📦 Pertimbangkan **peningkatan logistik cold storage** untuk menunjang performa **Instant & Frozen Food**.
* 📊 Lakukan analisis lanjut pada kategori **Grocery & Staples** untuk mengidentifikasi produk dengan performa terbaik guna dioptimalkan dalam kampanye penjualan.



---

Feel free to fork, clone, or contribute! 🚀
