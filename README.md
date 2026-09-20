# Smart-Dine Analytics 🍽️📊

Food-delivery data, cleaned and turned into decisions.

MySQL → Apache Hop → Databricks → Power BI

---

## About this project

This project takes restaurant order data from a source database, cleans it, stores it in a warehouse, and shows the result in a four-page Power BI report.

I did not start with charts. I started with a simple question: can we trust this file enough to make a decision?

There are four tables. `fact_order_lines` is each item on a bill (30,000 rows). Around it sit customers (1,500), products (50) and locations (15). That shape is a star schema. In Power BI it is just three relationships:

- fact.customer_id → dim_customers.customer_id
- fact.product_id → dim_products.product_id
- fact.location_id → dim_locations.location_id

A line is one item on a bill. A bill is the whole order. The data has about 30,000 items and about 12,000 bills.

---

## Business problem

A delivery brand needs to know:

- which cities bring the most orders
- what the kitchen should stock first
- whether customers wait too long
- whether finance can trust the sales column
- whether VIP customers actually spend more

The raw data could not answer that cleanly. The same city was written as `DELHI`, `New Delhi` and `mumbai`. Some prices were empty. Some were negative. If you sum the sales column as-is, the company total is wrong. If you group by the raw city name, Delhi looks like two different markets.

---

## Tools and why I used them

**MySQL**  
Source system. I loaded the four raw tables here. I did not build the report on top of MySQL.

**Apache Hop**  
Cleaning. I trimmed extra spaces, mapped messy city names to one clean city, tagged every sales row as VALID, MISSING or NEGATIVE, and wrote the 250 bad rows to a reject file. I did not delete those rows. If you delete the mess, nobody can see there was a mess.

**Databricks**  
Warehouse. This is where I ran SQL, joined the four tables, and got the numbers that sit behind the dashboard.

**Power BI**  
The report. Four pages: Demand, Delay, Quality and Decisions.

---

## What I did, step by step

1. Loaded the four tables into MySQL.
2. Built a Hop pipeline that reads those tables.
3. Trimmed text and mapped city names (`DELHI` and `New Delhi` both became Delhi).
4. Flagged sales quality and sent bad rows to a reject file.
5. Loaded the small dimension tables into Databricks from Hop.
6. Loaded the fact table from a cleaned CSV because a live Hop insert of 30,000 rows was too slow on the free warehouse.
7. Answered the business questions with SQL. I had to TRIM join keys because some ids still had spaces.
8. Connected the same four tables in Power BI and built the report.

![Hop pipeline](docs/docs05-hop-pipeline.png)

---

## Problems I hit and how I fixed them

Loading 30,000 fact rows through Hop into Databricks was too slow on the free warehouse. I wrote a cleaned CSV and uploaded that instead.

Databricks preview often showed only 100 rows. I used COUNT(*) after that. The real counts are 1,500 / 50 / 15 / 30,000.

Joins came back empty because some ids still had spaces. TRIM on the keys fixed it.

I also learned not to leave Truncate on, and not to overwrite the wrong table. Both wiped data I then had to load again.

---

## Dashboard

![Demand](docs/01-demand.png)
![Delay](docs/02-delay.png)
![Quality](docs/03-quality.png)
![Decisions](docs/04-decisions.png)

Demand shows where orders come from, what sells, and when in the day.  
Delay shows waiting time.  
Quality shows bad prices and messy city names.  
Decisions is what I would actually tell a manager.

---

## Answers from the dashboard

- 29,750 items we can trust, on 11,643 bills. Average bill about ₹2,025. Sales about ₹23.58 million.
- Delhi, Mumbai and Lucknow take most of the all-day demand.
- At dinner (7–10 pm) Mumbai is first. That is where I would add staff.
- Main Course sells the most plates. Stock that first.
- Average wait looks okay at about 50 minutes. It is not okay when 37% of items take more than an hour.
- Peak hours are not slower than quiet hours. The wait problem is everywhere, not only at dinner.
- VIP bills are only about ₹40 higher than Regular. Regular places about five times more orders. The business is Regular customers.
- 100 prices are missing and 150 are negative. Finance should add VALID sales only.

---

## Short version

I asked if the data was trustworthy first. Then I cleaned cities, flagged bad sales, ran SQL, and built four pages. The charts came last.

SQL · Hop · Databricks · Power BI · data quality
