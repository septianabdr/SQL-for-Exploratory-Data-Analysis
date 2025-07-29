# SQL for Exploratory Data Analysis
---
SQL for Data Exploratory: BlinkIt Sales Dataset
Repositori ini memuat eksporasi data Blinkit menngunakan software SSMS (SQL Server Management Studio)

Dataset [Blinkit grocery](https://www.kaggle.com/datasets/akxiit/blinkit-sales-dataset)

## Product and Brand Analysis
---
1. Category paling laris
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
**Output**
| Category              | Total Revenue | Total Items |
|-----------------------|---------------|--------------|
| Fruits & Vegetables   | 53,955,458    | 966          |
| Personal Care         | 35,018,297    | 887          |
| Grocery & Staples     | 33,853,141    | 895          |
| Cold Drinks & Juices  | 31,599,928    | 758          |
| Instant & Frozen Food | 29,209,697    | 742          |

2. Produk paling laris
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
**Output**
| Product Name    | Category        | Total Revenue | Total Item |
|----------------|------------------|----------------|-------------|
| Vitamins       | Pharmacy         | 26,082,201     | 380         |
| Pet Treats     | Pet Care         | 20,469,545     | 473         |
| Toilet Cleaner | Household Care   | 19,983,748     | 430         |
| Cough Syrup    | Pharmacy         | 17,338,101     | 373         |
| Dish Soap      | Household Care   | 15,969,544     | 397         |

3. Sentimen
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
**Output**
| Sentiment | Percentage (%) |
|-----------|----------------|
| Negative  | 21.56          |
| Neutral   | 27.96          |
| Positive  | 50.48          |

4. Barang yang rusak
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
**Output**
| Category                 | Total Damaged Stock  |
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

5. 10 customers
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
**Output**
| ID       | Name             | Email                              | Customer Code |
|----------|------------------|------------------------------------|----------------|
| 25128143 | Odika Kannan     | kondaadya@example.org              | 1053339        |
| 77869660 | Nidhi Sha        | ewali@example.org                  | 1011575        |
| 4597433  | Lipika Kumer     | faras46@example.org                | 955369         |
| 33331259 | Leena Deol       | nitesh33@example.com               | 940377         |
| 14472401 | Ayaan Rege       | dubeyekanta@example.com            | 927573         |
| 85561151 | Zinal Dhawan     | gulatiedhitha@example.org          | 926859         |
| 22800586 | Jacob Lad        | ramakrishnanchandresh@example.net  | 923724         |
| 34668988 | Wishi Varughese  | damyanti67@example.net             | 916073         |
| 13760839 | Anvi Savant      | yashvi72@example.org               | 911399         |
| 12832151 | Ekavir Bhalla    | ksehgal@example.net                | 884111         |

6. Product Feedback negative
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
**Output**
| product         | negative_feedback |
|------------------|----------|
| Pet Treats       | 44       |
| Baby Wipes       | 41       |
| Lotion           | 41       |
| Dish Soap        | 39       |
| Toilet Cleaner   | 38       |
**Output**
| product         | positive_feedback |
|------------------|----------|
| Pet Treats       | 121      |
| Toilet Cleaner   | 110      |
| Cough Syrup      | 108      |
| Vitamins         | 95       |
| Dish Soap        | 94       |

7. Brand 
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
**Output**
| Brand        | Revenue   |
|----------------|-----------|
| Karnik PLC     | 6,521,270 |
| Mandal-Kar     | 5,646,465 |
| Roy-Char       | 5,518,294 |
| Sundaram Inc   | 5,183,035 |
| Gole-Doshi     | 5,179,096 |

8. New Customer
```sql
SELECT 
    FORMAT(registration_date, 'yyyy-MM') AS month,
    COUNT(*) AS new_customers
FROM customers
WHERE customer_segment = 'New'
GROUP BY FORMAT(registration_date, 'yyyy-MM')
ORDER BY new_customers DESC
```
**Output**
| Month     | New Customers |
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
