---
title:  "Useful SQL Commands"
search: true
toc: true
categories: 
  - Jekyll
last_modified_at: 2020-05-10T04:06:00-05:00
---

# Relational Databases

[Course url in udacity](https://classroom.udacity.com/courses/ud198)
---
![](/assets/images/posts/rdb/table_structure.png)

---
## LESSON 03 - SQL AGGREGATIONS

COUNT, MIN, MAX, SUM, AVG
1. All the columns selected and not aggregated should always be in the GROUP BY clause.

2. GROUP BY clause should come in between WHERE clause and ORDER BY clause

3. GROUP BY and ORDER BY can be used with multiple columns in the same query

4. DISTINCT can be used when you don't want to use aggregations

5. It’s worth noting that using DISTINCT, particularly in aggregations, can slow
your queries down quite a bit.

6. WHERE clause does not allow you to filter on aggregate columns - therefore
we need to use HAVING clause for filtering on aggregated columns - usually HAVING
clause comes after GROUP BY clause.

7. HAVING is the “clean” way to filter a query that has been aggregated, but this is also commonly done using a subquery. Essentially, any time you want to perform a WHERE on an element of your query that was created by an aggregate, you need to use HAVING instead.

8. HAVING is the “clean” way to filter a query that has been aggregated, but this is also commonly done using a subquery. Essentially, any time you want to perform a WHERE on an element of your query that was created by an aggregate, you need to use HAVING instead.

### Lesson 3 - section 15

```sql
SELECT a.name account_name, o.occurred_at date, o.total_amt_usd total
	FROM orders AS o
    JOIN accounts AS a ON a.id = o.account_id
ORDER BY o.occurred_at DESC
LIMIT 2;
------------------------------------------------------
SELECT a.name account_name, SUM(o.total)
	FROM orders AS o
    JOIN accounts AS a ON a.id = o.account_id
GROUP BY account_name
ORDER BY account_name
LIMIT 2;
-------------------------------------------------------
SELECT a.name, MIN(total_amt_usd) smallest_order
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY a.name
ORDER BY smallest_order;

```

### Lesson 03 - section 18 - example queries for AVG and COUNT functionality

```sql
SELECT a.name, AVG(o.standard_amt_usd) avg_stand, AVG(o.gloss_amt_usd) avg_gloss, AVG(o.poster_amt_usd) avg_post
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY a.name;
------------------------------------------------------
SELECT r.name, w.channel, COUNT(*) num_events
FROM accounts a
JOIN web_events w
ON a.id = w.account_id
JOIN sales_reps s
ON s.id = a.sales_rep_id
JOIN region r
ON r.id = s.region_id
GROUP BY r.name, w.channel
ORDER BY num_events DESC;

```

### Lesson 03 - section 21 - using DISTINCT

```SQL
SELECT s.id, s.name, COUNT(*) num_accounts
FROM accounts a
JOIN sales_reps s
ON s.id = a.sales_rep_id
GROUP BY s.id, s.name
ORDER BY num_accounts;
--------------------------------------------------
SELECT DISTINCT id, name
FROM sales_reps;
```

### Lesson 03 - section 23 - using HAVING

<img src="/assets/images/posts/rdb/sql_where_and_having_clauses.png" width=800 >

```sql
SELECT s.id, s.name, COUNT(*) num_accounts
FROM accounts a
JOIN sales_reps s
ON s.id = a.sales_rep_id
GROUP BY s.id, s.name
HAVING COUNT(*) > 5
ORDER BY num_accounts;
--------------------------------------------------------------
SELECT COUNT(*) num_reps_above5
FROM(SELECT s.id, s.name, COUNT(*) num_accounts
     FROM accounts a
     JOIN sales_reps s
     ON s.id = a.sales_rep_id
     GROUP BY s.id, s.name
     HAVING COUNT(*) > 5
     ORDER BY num_accounts) AS Table1;
```

### Lesson 03 - section 25 - using date functionality

* DATE_TRUNC('hour', occurred_at)
* DATE_TRUNC('day', occurred_at)
* DATE_PART('second', occurred_at)
* DATE_PART('month', date)
* DATE_PART('year', date)
* DATE_PART('dow', date)

```sql
SELECT DATE_PART('year', occurred_at) ord_year,  SUM(total_amt_usd) total_spent
FROM orders
GROUP BY 1
ORDER BY 2 DESC;
-------------------------------------------------------------------
SELECT DATE_PART('month', occurred_at) ord_month, SUM(total_amt_usd) total_spent
FROM orders
WHERE occurred_at BETWEEN '2014-01-01' AND '2017-01-01'
GROUP BY 1
ORDER BY 2 DESC;
-------------------------------------------------------------------
SELECT DATE_TRUNC('month', o.occurred_at) ord_date, SUM(o.gloss_amt_usd) tot_spent
FROM orders o
JOIN accounts a
ON a.id = o.account_id
WHERE a.name = 'Walmart'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

### Lesson 03 - section 30 - using CASE statements

* The CASE statement always goes in the SELECT clause.

* CASE must include the following components: WHEN, THEN, and END. ELSE is an optional component to catch cases that didn’t meet any of the other previous CASE conditions.

* You can make any conditional statement using any conditional operator (like WHERE) between WHEN and THEN. This includes stringing together multiple conditional statements using AND and OR.

* You can include multiple WHEN statements, as well as an ELSE statement again, to deal with any unaddressed conditions.

```sql
-- handing division by zero
SELECT account_id,
		CASE 	WHEN standard_qty = 0 OR standard_qty IS NULL THEN 0
        			ELSE standard_amt_usd/standard_qty
		END AS unit_price
FROM orders
LIMIT 10;
--------------------------------------------------------
SELECT a.name, SUM(total_amt_usd) total_spent,
     CASE WHEN SUM(total_amt_usd) > 200000 THEN 'top'
     WHEN  SUM(total_amt_usd) > 100000 THEN 'middle'
     ELSE 'low' END AS customer_level
FROM orders o
JOIN accounts a
ON o.account_id = a.id
WHERE occurred_at > '2015-12-31'
GROUP BY 1
ORDER BY 2 DESC;
-------------------------------------------------------
SELECT s.name, COUNT(*), SUM(o.total_amt_usd) total_spent,
     CASE WHEN COUNT(*) > 200 OR SUM(o.total_amt_usd) > 750000 THEN 'top'
     WHEN COUNT(*) > 150 OR SUM(o.total_amt_usd) > 500000 THEN 'middle'
     ELSE 'low' END AS sales_rep_level
FROM orders o
JOIN accounts a
ON o.account_id = a.id
JOIN sales_reps s
ON s.id = a.sales_rep_id
GROUP BY s.name
ORDER BY 3 DESC;
----------------------------------------------------------
SELECT
    count(created_at) as count,
    CASE
        WHEN hour(created_at) = 0 THEN '00'
        WHEN hour(created_at) = 1 THEN '01'
        WHEN hour(created_at) = 2 THEN '02'
        WHEN hour(created_at) = 3 THEN '03'
        WHEN hour(created_at) = 4 THEN '04'
        WHEN hour(created_at) = 5 THEN '05'
        WHEN hour(created_at) = 6 THEN '06'
        WHEN hour(created_at) = 7 THEN '07'
        WHEN hour(created_at) = 8 THEN '08'
        WHEN hour(created_at) = 9 THEN '09'
        WHEN hour(created_at) = 10 THEN '10'
        WHEN hour(created_at) = 11 THEN '11'
        WHEN hour(created_at) = 12 THEN '12'
        WHEN hour(created_at) = 13 THEN '13'
        WHEN hour(created_at) = 14 THEN '14'
        WHEN hour(created_at) = 15 THEN '15'
        WHEN hour(created_at) = 16 THEN '16'
        WHEN hour(created_at) = 17 THEN '17'
        WHEN hour(created_at) = 18 THEN '18'
        WHEN hour(created_at) = 19 THEN '19'
        WHEN hour(created_at) = 20 THEN '20'
        WHEN hour(created_at) = 21 THEN '21'
        WHEN hour(created_at) = 22 THEN '22'
        WHEN hour(created_at) = 23 THEN '23'
    END AS intervals
FROM
    sms_message
WHERE
    DAY(created_at) = 02
    AND MONTH(created_at) = 05
    AND YEAR(created_at) = 2020
GROUP BY intervals;

```

## LESSON 04 - SQL SUB-QUERIES AND TEMPORARY TABLES

Note that you should not include an alias when you write a subquery in a conditional statement. This is because the subquery is treated as an individual value (or set of values in the IN case) rather than as a table.

Also, notice the query here compared a single value. If we returned an entire column IN would need to be used to perform a logical argument. If we are returning an entire table, then we must use an ALIAS for the table, and perform additional logic on the entire table.

```sql
SELECT channel, AVG(events) AS average_events
FROM (SELECT DATE_TRUNC('day',occurred_at) AS day,
             channel, COUNT(*) as events
      FROM web_events
      GROUP BY 1,2) sub
GROUP BY channel
ORDER BY 2 DESC;
-----------------------------------------------------------------------
SELECT AVG(standard_qty) avg_std, AVG(gloss_qty) avg_gls, AVG(poster_qty) avg_pst
FROM orders
WHERE DATE_TRUNC('month', occurred_at) =
     (SELECT DATE_TRUNC('month', MIN(occurred_at)) FROM orders);
-----------------------------------------------------------------------
SELECT SUM(total_amt_usd)
FROM orders
WHERE DATE_TRUNC('month', occurred_at) =
      (SELECT DATE_TRUNC('month', MIN(occurred_at)) FROM orders);

```

```SQL
SELECT t3.rep_name, t3.region_name, t3.total_amt
FROM(SELECT region_name, MAX(total_amt) total_amt
     FROM(SELECT s.name rep_name, r.name region_name, SUM(o.total_amt_usd) total_amt
             FROM sales_reps s
             JOIN accounts a
             ON a.sales_rep_id = s.id
             JOIN orders o
             ON o.account_id = a.id
             JOIN region r
             ON r.id = s.region_id
             GROUP BY 1, 2) t1
     GROUP BY 1) t2
JOIN (SELECT s.name rep_name, r.name region_name, SUM(o.total_amt_usd) total_amt
     FROM sales_reps s
     JOIN accounts a
     ON a.sales_rep_id = s.id
     JOIN orders o
     ON o.account_id = a.id
     JOIN region r
     ON r.id = s.region_id
     GROUP BY 1,2
     ORDER BY 3 DESC) t3
ON t3.region_name = t2.region_name AND t3.total_amt = t2.total_amt;
```

WITH statement for Common Table Expressions (CTE)
make sure to define the CTEs at the top of the queries and without a semi-colon

```sql
-- sql query with sub QUERIES
SELECT channel, AVG(events) AS average_events
FROM (SELECT DATE_TRUNC('day',occurred_at) AS day,
             channel, COUNT(*) as events
      FROM web_events
      GROUP BY 1,2) sub
GROUP BY channel
ORDER BY 2 DESC;

-- same query can be written using WITH / common table expressions to
-- reduce the same query
WITH events AS (
          SELECT DATE_TRUNC('day',occurred_at) AS day,
                        channel, COUNT(*) as events
          FROM web_events
          GROUP BY 1,2)
------------------------------------------------------------
SELECT channel, AVG(events) AS average_events
FROM events
GROUP BY channel
ORDER BY 2 DESC;

```

## LESSON 05 - SQL DATA CLEANING

* LEFT, RIGHT, LENGTH
* LEFT("string", 2) returns st
* RIGHT("string", 2) returns ng
* LENGTH("string") returns 6
* RIGHT("string", LENGTH("string") - 4) returns ng

* POS returns a position of a given string - POSITION(',' IN city_state)

* STRPOS is same as POS but the way the command is written is different -
STRPOS(city_state, ',') returns the comma position in the city_state column

* both POS and STRPOS functions are case sensitive - we can use LOWER or UPPER
commands to format the strings to either upper or lower case letters

* LEFT(city_state, POS(',' IN city_state)) will return cincinnati from cincinnati, oh by first finding the location of the comma and then taking the left side string
from the comma position.


```sql

SELECT RIGHT(website, 3) AS domain, COUNT(*) num_companies
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;
---------------------------------------------------------------
SELECT LEFT(UPPER(name), 1) AS first_letter, COUNT(*) num_companies
FROM accounts
GROUP BY 1
ORDER BY 2 DESC;

/*
There are 350 company names that start with a letter and 1 that starts with a number. This gives a ratio of 350/351 that are company names that start with a letter or 99.7%.
*/
SELECT SUM(num) nums, SUM(letter) letters
FROM (SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9')
                       THEN 1 ELSE 0 END AS num,
         CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9')
                       THEN 0 ELSE 1 END AS letter
      FROM accounts) t1;
-------------------------------------------------------------
SELECT SUM(num) nums, SUM(letter) letters
FROM (SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('0','1','2','3','4','5','6','7','8','9')
                       THEN 1 ELSE 0 END AS num,
         CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U')
                       THEN 1 ELSE 0 END AS letter
      FROM accounts) t1;

```

### CONCAT is used to concatenate string values

usage is like CONCAT(first_name, ' ', last_name) AS full_name or we can even use 2 pipe characters
to concatenate strings like first_name || ' ' || last_name AS full_name

```sql

WITH t1 AS (
 SELECT LEFT(primary_poc,     STRPOS(primary_poc, ' ') -1 ) first_name,  RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name, name
 FROM accounts)
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', name, '.com')
FROM t1;
----------------------------------------
WITH t1 AS (
 SELECT LEFT(primary_poc,     STRPOS(primary_poc, ' ') -1 ) first_name,  RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name, name
 FROM accounts)
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', REPLACE(name, ' ', ''), '.com')
FROM  t1;
----------------------------------------
WITH t1 AS (
 SELECT LEFT(primary_poc,     STRPOS(primary_poc, ' ') -1 ) first_name,  RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name, name
 FROM accounts)
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', name, '.com'), LEFT(LOWER(first_name), 1) || RIGHT(LOWER(first_name), 1) || LEFT(LOWER(last_name), 1) || RIGHT(LOWER(last_name), 1) || LENGTH(first_name) || LENGTH(last_name) || REPLACE(UPPER(name), ' ', '')
FROM t1;

```

### CAST and TO_DATE function usage

![](/assets/images/posts/rdb/casting_and_date_formatting.png)

```sql
SELECT date orig_date, (SUBSTR(date, 7, 4) || '-' || LEFT(date, 2) || '-' || SUBSTR(date, 4, 2)) new_date
FROM sf_crime_data;

SELECT date orig_date, (SUBSTR(date, 7, 4) || '-' || LEFT(date, 2) || '-' || SUBSTR(date, 4, 2))::DATE new_date
FROM sf_crime_data;

```

### COALESCE

COALESCE is used to fill null values with some other values
usage is like COALESCE(column_name, 'null_value_replacement')

```sql

SELECT COALESCE(a.id, a.id) filled_id, a.name, a.website, a.lat, a.long, a.primary_poc, a.sales_rep_id, COALESCE(o.account_id, a.id) account_id, o.occurred_at, COALESCE(o.standard_qty, 0) standard_qty, COALESCE(o.gloss_qty,0) gloss_qty, COALESCE(o.poster_qty,0) poster_qty, COALESCE(o.total,0) total, COALESCE(o.standard_amt_usd,0) standard_amt_usd, COALESCE(o.gloss_amt_usd,0) gloss_amt_usd, COALESCE(o.poster_amt_usd,0) poster_amt_usd, COALESCE(o.total_amt_usd,0) total_amt_usd
FROM accounts a
LEFT JOIN orders o
ON a.id = o.account_id
WHERE o.total IS NULL;

```

## LESSON 06 - SQL WINDOW FUNCTIONS

It allows us to compare one row to another row without any JOINS
for example - create a running total, whether one row was created earlier than
the previous row and classify it based on your finding
