# <p align="center" style="margin-top: 0px;"> Fresh Segments 🍊
## <p align="center"> Customer Segment Analysis

## Question 1
*Which product has the highest price? Only return a single row*
```sql
 -- =============================================================================
-- Question 1 : Which product has the highest price? Only return a single row.
-- =============================================================================

SELECT TOP 1 * FROM products
ORDER BY price DESC; 
  
```
| product_id | product_name | price |
|------------|--------------|-------|
|     13     |  Product M   |   70  |
  
  
  
## Question 2
*Which customer has made the most orders?*
```sql
 -- =============================================================================
-- Question 2 : Which customer has made the most orders?
-- =============================================================================

WITH order_count AS
(SELECT 
	A.customer_id, 
	COUNT(A.order_id) AS total_order,
	CONCAT(B.first_name, ' ' ,B.last_name) AS customer_fullname,
	MAX(COUNT(A.order_id)) OVER () AS max_total_order
FROM orders A
LEFT JOIN customers B
ON A.customer_id = B.customer_id
GROUP BY A.customer_id, CONCAT(B.first_name, ' ' ,B.last_name)
) 
-- =============================================================
-- Another method is to use subquery in where to select record 
-- WHERE total order is max, but this is more efficient.
-- =============================================================
SELECT 
	customer_id,
	total_order, 
	customer_fullname
FROM order_count
WHERE total_order = max_total_order; 
```
| customer_id | total_order | customer_fullname |
|-------------|-------------|------------------|
|      3      |      2      |    Bob Johnson    |
|      2      |      2      |    Jane Smith     |
|      1      |      2      |     John Doe      |
  
  
  
## Question 3
*What’s the total revenue per product?*   
```sql
 SELECT 
	A.product_id, 
	B.product_name,
	SUM(A.quantity) AS total_quantity_sold, 
	SUM(A.quantity * B.price) AS total_revenue
FROM order_items A
LEFT JOIN products B
ON A.product_id = B.product_id
GROUP BY A.product_id,B.product_name; 
  
```
| product_id | product_name | total_quantity_sold | total_revenue |
|------------|--------------|---------------------|---------------|
|     1      |  Product A   |         5           |      50       |
|     2      |  Product B   |         9           |     135       |
|     3      |  Product C   |         8           |     160       |
|     4      |  Product D   |         3           |      75       |
|     5      |  Product E   |         3           |      90       |
|     6      |  Product F   |         6           |     210       |
|     7      |  Product G   |         3           |     120       |
|     8      |  Product H   |         3           |     135       |
|     9      |  Product I   |         3           |     150       |
|    10      |  Product J   |         6           |     330       |
|    11      |  Product K   |         3           |     180       |
|    12      |  Product L   |         3           |     195       |
|    13      |  Product M   |         6           |     420       |
  
  
  
## Question 4
*Find the day with the highest revenue.*   
```sql
 WITH order_rev AS 
(SELECT 
	C.order_date,
	A.order_id, 
	SUM(A.quantity * B.price) AS total_revenue
FROM order_items A
LEFT JOIN products B
ON A.product_id = B.product_id
LEFT JOIN orders C
ON A.order_id = C.order_id
GROUP BY C.order_date,A.order_id)

-- Select the date record for highest revenue
SELECT TOP 1 * FROM order_rev
ORDER BY total_revenue DESC;
  
```  
 | order_date  | order_id | total_revenue |
|-------------|----------|---------------|
| 2023-05-16  |    16    |     340       |
 
  
  
## Question 5  
*Find the first order (by date) for each customer.*
```sql
  SELECT 
	DISTINCT(A.customer_id),
	CONCAT(B.first_name, ' ' ,B.last_name) AS customer_fullname,
	MIN(A.order_date) OVER(PARTITION BY A.customer_id) AS Earliest_date
FROM orders A
LEFT JOIN customers B
ON A.customer_id = B.customer_id;
  
```
| customer_id | customer_fullname | Earliest_date |
|-------------|------------------|---------------|
|      1      |     John Doe      |  2023-05-01   |
|      2      |    Jane Smith     |  2023-05-02   |
|      3      |    Bob Johnson    |  2023-05-03   |
|      4      |    Alice Brown    |  2023-05-07   |
|      5      |  Charlie Davis    |  2023-05-08   |
|      6      |    Eva Fisher     |  2023-05-09   |
|      7      |  George Harris    |  2023-05-10   |
|      8      |     Ivy Jones     |  2023-05-11   |
|      9      |   Kevin Miller    |  2023-05-12   |
|     10      |   Lily Nelson     |  2023-05-13   |
|     11      | Oliver Patterson  |  2023-05-14   |
|     12      |  Quinn Roberts    |  2023-05-15   |
|     13      |  Sophia Thomas    |  2023-05-16   |
 
  
## Question 6  
*Find the top 3 customers who have ordered the most distinct products*
```sql
WITH customer_order AS
(SELECT
	A.customer_id,
	CONCAT(B.first_name, ' ' ,B.last_name) AS customer_fullname,
	COUNT(A.order_id) AS order_count
FROM orders A
LEFT JOIN customers B
ON A.customer_id = B.customer_id
GROUP BY A.customer_id, CONCAT(B.first_name, ' ', B.last_name))

SELECT * FROM customer_order
WHERE order_count = (SELECT MAX(order_count)
					 FROM customer_order);  
  
```
| customer_id | customer_fullname | order_count |
|-------------|------------------|-------------|
|     3       |   Bob Johnson    |      2      |
|     2       |   Jane Smith     |      2      |
|     1       |    John Doe      |      2      |
  
  
## Question 7    
*Which product has been bought the least in terms of quantity?*
```sql
WITH product_quantity AS
(SELECT
	A.product_id,
	B.product_name,
    SUM(A.quantity) AS total_quantity,
    DENSE_RANK() OVER (ORDER BY SUM(A.quantity) DESC) AS purchase_rate
    FROM order_items A
	LEFT JOIN products B
	ON A.product_id = B.product_id
    GROUP BY A.product_id,B.product_name
)
SELECT 
    product_name,
    total_quantity
FROM product_quantity
WHERE purchase_rate = 5;  
  
```
| product_name | total_quantity |
|--------------|----------------|
|  Product K   |       3        |
|  Product L   |       3        |
|  Product G   |       3        |
|  Product H   |       3        |
|  Product I   |       3        |
|  Product D   |       3        |
|  Product E   |       3        |

  
  
## Question 8
*What is the median order total?*
```sql
WITH order_total AS
(SELECT 
	A.order_id, 
	SUM(A.quantity * B.price) AS total_revenue
FROM order_items A
LEFT JOIN products B
ON A.product_id = B.product_id
LEFT JOIN orders C
ON A.order_id = C.order_id
GROUP BY C.order_date,A.order_id),

ranked_data AS
(SELECT 
	total_revenue,
	ROW_NUMBER() OVER (ORDER BY total_revenue) AS row_num,
    COUNT(*) OVER () AS total_rows
 FROM order_total
)


-- Calculate the average
SELECT AVG(total_revenue) AS Median
FROM ranked_data
WHERE row_num IN ((total_rows + 1) / 2, (total_rows + 2) / 2);  
  
```
| Median |
|--------|
|112.5000|
 
  
  
## Question 9
*For each order, determine if it was ‘Expensive’ (total over 300), ‘Affordable’ (total over 100), or ‘Cheap’.*
```sql
 WITH order_total AS
(SELECT 
	A.order_id, 
	SUM(A.quantity * B.price) AS total_revenue
FROM order_items A
LEFT JOIN products B
ON A.product_id = B.product_id
LEFT JOIN orders C
ON A.order_id = C.order_id
GROUP BY C.order_date,A.order_id)

SELECT
	order_id,
	total_revenue,
	CASE 
		WHEN total_revenue > 300 THEN 'Expensive'
		WHEN total_revenue > 100 AND total_revenue < 300 THEN 'total over 100'
		ELSE 'Cheap'
	END AS How_Expensive
FROM order_total; 
  
```
| order_id | total_revenue |   How_Expensive   |
|----------|---------------|-------------------|
|    1     |      35       |       Cheap       |
|    2     |      75       |       Cheap       |
|    3     |      50       |       Cheap       |
|    4     |      80       |       Cheap       |
|    5     |      50       |       Cheap       |
|    6     |      55       |       Cheap       |
|    7     |      85       |       Cheap       |
|    8     |     145       | total over 100    |
|    9     |     140       | total over 100    |
|    10    |     285       | total over 100    |
|    11    |     275       | total over 100    |
|    12    |      80       |       Cheap       |
|    13    |     185       | total over 100    |
|    14    |     145       | total over 100    |
|    15    |     225       | total over 100    |
|    16    |     340       |     Expensive     |
                                                    
                                                    
                                                    
## Question 10
*Find customers who have ordered the product with the highest price.*
  
```sql
WITH Highest_Selling AS 
(SELECT 
	C.customer_id,
	A.order_id, 
	A.product_id, 
	B.product_name,
	SUM(A.quantity * B.price) AS total_revenue,
	DENSE_RANK() OVER (ORDER BY SUM(A.quantity * B.price) DESC) AS Highest_sold
FROM order_items A
LEFT JOIN products B
ON A.product_id = B.product_id
LEFT JOIN orders C
ON A.order_id = C.order_id
GROUP BY C.customer_id,A.order_id, A.product_id,B.product_name),

customer AS
(SELECT
	A.customer_id,
	CONCAT(B.first_name, ' ' ,B.last_name) AS customer_fullname
FROM orders A
LEFT JOIN customers B
ON A.customer_id = B.customer_id
GROUP BY A.customer_id, CONCAT(B.first_name, ' ', B.last_name))

-- Final step, is to join with customer table
SELECT 
	X.product_id,
	X.product_name,
	X.total_revenue,
	Y.customer_fullname
FROM Highest_Selling X
LEFT  JOIN customer Y
ON X.customer_id = Y.customer_id
WHERE X.Highest_sold = 1; 
```
|product_id | product_name | total_revenue | customer_fullname|
|----------|--------------|---------------|-------------------|
|13         | Product M    | 210           | Ivy Jones |
|13         | Product M    | 210           | Sophia Thomas |

