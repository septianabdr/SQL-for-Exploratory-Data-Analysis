# SQL-for-Exploratory-Data-Analysis
SQL for Data Exploratory: BlinkIt Sales Dataset

Dataset [Blinkit grocery](https://www.kaggle.com/datasets/akxiit/blinkit-sales-dataset)

1. Category paling laris
```sql
WITH sales_category AS(
    SELECT TOP 5
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
SELECT * FROM sales_category
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
