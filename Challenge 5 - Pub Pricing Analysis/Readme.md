Create the Table Pubs
```sql
CREATE TABLE pubs (
pub_id INT PRIMARY KEY,
pub_name VARCHAR(50),
city VARCHAR(50),
state VARCHAR(50),
country VARCHAR(50)
);
```

Create the 'beverages' table
```sql
CREATE TABLE beverages (
beverage_id INT PRIMARY KEY,
beverage_name VARCHAR(50),
category VARCHAR(50),
alcohol_content FLOAT,
price_per_unit DECIMAL(8, 2)
);
```
Create the 'sales' table
```sql
CREATE TABLE sales (
sale_id INT PRIMARY KEY,
pub_id INT,
beverage_id INT,
quantity INT,
transaction_date DATE,
FOREIGN KEY (pub_id) REFERENCES pubs(pub_id),
FOREIGN KEY (beverage_id) REFERENCES beverages(beverage_id)
);
```
Create the 'ratings' table 
```sql
CREATE TABLE ratings ( rating_id INT PRIMARY KEY,
 pub_id INT,
customer_name VARCHAR(50),
 rating FLOAT,
review TEXT, FOREIGN KEY (pub_id) REFERENCES pubs(pub_id) );
```

Insert sample data into the 'pubs' table
```sql
INSERT INTO pubs (pub_id, pub_name, city, state, country)
VALUES
(1, 'The Red Lion', 'London', 'England', 'United Kingdom'),
(2, 'The Dubliner', 'Dublin', 'Dublin', 'Ireland'),
(3, 'The Cheers Bar', 'Boston', 'Massachusetts', 'United States'),
(4, 'La Cerveceria', 'Barcelona', 'Catalonia', 'Spain');
```

Insert sample data into the 'beverages' table
```sql
INSERT INTO beverages (beverage_id, beverage_name, category, alcohol_content, price_per_unit)
VALUES
(1, 'Guinness', 'Beer', 4.2, 5.99),
(2, 'Jameson', 'Whiskey', 40.0, 29.99),
(3, 'Mojito', 'Cocktail', 12.0, 8.99),
(4, 'Chardonnay', 'Wine', 13.5, 12.99),
(5, 'IPA', 'Beer', 6.8, 4.99),
(6, 'Tequila', 'Spirit', 38.0, 24.99);
```
Insert sample data into the 'sales' table
```sql
INSERT INTO sales (sale_id, pub_id, beverage_id, quantity, transaction_date)
VALUES
(1, 1, 1, 10, '2023-05-01'),
(2, 1, 2, 5, '2023-05-01'),
(3, 2, 1, 8, '2023-05-01'),
(4, 3, 3, 12, '2023-05-02'),
(5, 4, 4, 3, '2023-05-02'),
(6, 4, 6, 6, '2023-05-03'),
(7, 2, 3, 6, '2023-05-03'),
(8, 3, 1, 15, '2023-05-03'),
(9, 3, 4, 7, '2023-05-03'),
(10, 4, 1, 10, '2023-05-04'),
(11, 1, 3, 5, '2023-05-06'),
(12, 2, 2, 3, '2023-05-09'),
(13, 2, 5, 9, '2023-05-09'),
(14, 3, 6, 4, '2023-05-09'),
(15, 4, 3, 7, '2023-05-09'),
(16, 4, 4, 2, '2023-05-09'),
(17, 1, 4, 6, '2023-05-11'),
(18, 1, 6, 8, '2023-05-11'),
(19, 2, 1, 12, '2023-05-12'),
(20, 3, 5, 5, '2023-05-13');
```
Insert sample data into the 'ratings' table
```sql
INSERT INTO ratings (rating_id, pub_id, customer_name, rating, review)
VALUES
(1, 1, 'John Smith', 4.5, 'Great pub with a wide selection of beers.'),
(2, 1, 'Emma Johnson', 4.8, 'Excellent service and cozy atmosphere.'),
(3, 2, 'Michael Brown', 4.2, 'Authentic atmosphere and great beers.'),
(4, 3, 'Sophia Davis', 4.6, 'The cocktails were amazing! Will definitely come back.'),
(5, 4, 'Oliver Wilson', 4.9, 'The wine selection here is outstanding.'),
(6, 4, 'Isabella Moore', 4.3, 'Had a great time trying different spirits.'),
(7, 1, 'Sophia Davis', 4.7, 'Loved the pub food! Great ambiance.'),
(8, 2, 'Ethan Johnson', 4.5, 'A good place to hang out with friends.'),
(9, 2, 'Olivia Taylor', 4.1, 'The whiskey tasting experience was fantastic.'),
(10, 3, 'William Miller', 4.4, 'Friendly staff and live music on weekends.');
```
--1.) How many pubs are located in each country?
```sql
SELECT COUNT(pub_id) AS pubs
FROM pubs;
```

--2.) What is the total sales amount for each pub, including the beverage price and quantity sold?
```sql
SELECT pub_id,SUM(beverages.price_per_unit * sales.quantity) AS total_sales_amount
FROM beverages LEFT JOIN sales
ON beverages.beverage_id = sales.beverage_id
GROUP BY pub_id;
```

--3.) Which pub has the highest average rating?
```sql
SELECT top 1 pub_name,ROUND(AVG(rating),1) AS average_rating
FROM ratings
LEFT JOIN pubs
ON ratings.pub_id = pubs.pub_id
GROUP BY pub_name
ORDER BY average_rating DESC;
```

--4.) What are the top 5 beverages by sales quantity across all pubs?
```sql
SELECT top 5 beverages.beverage_id, beverages.beverage_name,
SUM(beverages.price_per_unit * sales.quantity) AS total_sales_amount
FROM beverages
LEFT JOIN sales
ON beverages.beverage_id = sales.beverage_id
GROUP BY beverages.beverage_id, beverages.beverage_name
ORDER BY total_sales_amount DESC;
```

--5.) How many sales transactions occurred on each date?
```sql
SELECT transaction_date,
COUNT(sale_id) AS transactions
FROM sales
GROUP BY transaction_date
ORDER BY transactions DESC;
```
--6.) Find the name of someone that had cocktails and which pub they had it in.
```sql
SELECT customer_name,pub_name,category,ratings.pub_id
FROM sales
LEFT JOIN beverages
ON sales.beverage_id = beverages.beverage_id
LEFT JOIN ratings
ON sales.pub_id = ratings.pub_id
LEFT JOIN pubs
ON ratings.pub_id = pubs.pub_id
WHERE category = 'Cocktail' AND review LIKE '%cocktail%'
GROUP BY customer_name, pub_name,category,ratings.pub_id;
```
--7.) What is the average price per unit for each category of beverages, excluding the category 'Spirit'?
```sql
SELECT category,
ROUND(AVG(price_per_unit),0) AS average_price_per_unit
FROM beverages
WHERE category <> 'Spirit'
GROUP BY category;
```
--8.) Which pubs have a rating higher than the average rating of all pubs?

```sql
SELECT p.pub_id, p.pub_name,
AVG(r.rating) AS average_rating
FROM ratings r
JOIN pubs p ON r.pub_id = p.pub_id
GROUP BY p.pub_id, p.pub_name
HAVING
AVG(r.rating) > (SELECT AVG(rating) FROM ratings);
```


--9.) For each country, what is the average price per unit of beverages in each category, 
--and what is the overall average price per unit of beverages across all categories?

```sql
SELECT p.country,b.category, AVG(b.price_per_unit) AS average_price_per_unit
FROM beverages b
INNER JOIN sales s ON b.beverage_id = s.beverage_id
INNER JOIN pubs p ON s.pub_id = p.pub_id
GROUP BY p.country, b.category
UNION ALL
SELECT 'All Countries' AS country,
'All Categories' AS category,
AVG(price_per_unit) AS average_price_per_unit
FROM beverages;
```
--10.) For each pub, what is the percentage contribution of each category
--of beverages to the total sales amount, and what is the pub's overall sales amount?

```sql
SELECT
s.pub_id,
b.category,
SUM(b.price_per_unit * s.quantity) AS category_sales,
SUM(b.price_per_unit * s.quantity) / t.total_sales * 100 AS category_percentage,
t.total_sales
FROM
sales s
JOIN
beverages b ON s.beverage_id = b.beverage_id
JOIN
(SELECT pub_id, SUM(price_per_unit * quantity) AS total_sales
FROM sales
JOIN beverages ON sales.beverage_id = beverages.beverage_id
GROUP BY pub_id) t ON s.pub_id = t.pub_id
GROUP BY
s.pub_id, b.category, t.total_sales
ORDER BY pub_id;
```
