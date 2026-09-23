Sunrise Supermarket – PLSQL Assignment One



Student Name:Jeannette Uwimpaye

Student ID:27745

Tool used: SQLite (command-line, sqlite3.exe on Windows)



&#x20;Business Scenario

Sunrise Supermarket sells products to customers who place orders containing

one or more items. This project models customers, products, orders, and

order items, then analyzes purchasing behavior and sales trends.



&#x20;Schema

\- customers: customer\_id, customer\_name, email, city

\- products: product\_id, product\_name, category, price

\- orders: order\_id, customer\_id, order\_date

\- order\_items: order\_item\_id, order\_id, product\_id, quantity



&#x20;Queries



&#x20;1. Orders with customer name, city, and date (INNER JOIN)

sql

SELECT o.order\_id, c.customer\_name, c.city, o.order\_date

FROM orders o

JOIN customers c ON o.customer\_id = c.customer\_id

ORDER BY o.order\_date;



Screenshot (Question1.png)



&#x20;2. Order items with product details (JOIN)

```sql

SELECT oi.order\_item\_id, oi.order\_id, p.product\_name, p.category, p.price, oi.quantity

FROM order\_items oi

JOIN products p ON oi.product\_id = p.product\_id

ORDER BY oi.order\_id;

```

Shows exactly what was purchased in each order line.

(Q2.png)



&#x20;3. All customers including those with no orders (LEFT JOIN)

```sql

SELECT c.customer\_name, o.order\_id, o.order\_date

FROM customers c

LEFT JOIN orders o ON c.customer\_id = o.customer\_id

ORDER BY c.customer\_name;



Keeps every customer even with no matching orders — reveals Esther Ingabire

has never placed an order.

(Q3.png)



&#x20;4. Customers spending above average (CTE)

```sql

WITH customer\_totals AS (

&#x20;   SELECT c.customer\_id, c.customer\_name,

&#x20;          SUM(oi.quantity \* p.price) AS total\_spent

&#x20;   FROM customers c

&#x20;   JOIN orders o ON c.customer\_id = o.customer\_id

&#x20;   JOIN order\_items oi ON o.order\_id = oi.order\_id

&#x20;   JOIN products p ON oi.product\_id = p.product\_id

&#x20;   GROUP BY c.customer\_id, c.customer\_name

)

SELECT customer\_name, total\_spent

FROM customer\_totals

WHERE total\_spent > (SELECT AVG(total\_spent) FROM customer\_totals)

ORDER BY total\_spent DESC;

```

The CTE computes per-customer totals once; the outer query filters against

the average of that same set.

(Q4.png)



&#x20;5. Customer ranking by spend (RANK window function)

```sql

WITH customer\_totals AS (

&#x20;   SELECT c.customer\_name, SUM(oi.quantity \* p.price) AS total\_spent

&#x20;   FROM customers c

&#x20;   JOIN orders o ON c.customer\_id = o.customer\_id

&#x20;   JOIN order\_items oi ON o.order\_id = oi.order\_id

&#x20;   JOIN products p ON oi.product\_id = p.product\_id

&#x20;   GROUP BY c.customer\_name

)

SELECT customer\_name, total\_spent,

&#x20;      RANK() OVER (ORDER BY total\_spent DESC) AS spend\_rank

FROM customer\_totals;

```

(Q5.png)



&#x20;6. Order sequence per customer (ROW\_NUMBER window function)

```sql

SELECT customer\_id, order\_id, order\_date,

&#x20;      ROW\_NUMBER() OVER (PARTITION BY customer\_id ORDER BY order\_date) AS order\_sequence

FROM orders;

```

(Q6.png)



&#x20;7. Running revenue total (SUM window function)

```sql

WITH order\_revenue AS (

&#x20;   SELECT o.order\_id, o.order\_date, SUM(oi.quantity \* p.price) AS order\_total

&#x20;   FROM orders o

&#x20;   JOIN order\_items oi ON o.order\_id = oi.order\_id

&#x20;   JOIN products p ON oi.product\_id = p.product\_id

&#x20;   GROUP BY o.order\_id, o.order\_date

)

SELECT order\_id, order\_date, order\_total,

&#x20;      SUM(order\_total) OVER (ORDER BY order\_date) AS running\_total

FROM order\_revenue

ORDER BY order\_date;

```

(Q7.png)



&#x20;8. Days between orders per customer (LAG window function)

```sql

SELECT customer\_id, order\_id, order\_date,

&#x20;      julianday(order\_date) - julianday(LAG(order\_date) OVER (PARTITION BY customer\_id ORDER BY order\_date)) AS days\_since\_previous

FROM orders

ORDER BY customer\_id, order\_date;

```

(Q8.png)



Business Interpretation



David and Alice are the top spenders and order every 1–2 weeks, while Esther has never ordered — a target for re-engagement. Revenue climbs steadily from June to July, showing healthy, consistent sales rather than one-off spikes.



