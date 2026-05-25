# 🎬 DVD Rental Enterprise Analytics: From SQL Backend to BI Dashboard

## 📌 Project Overview
This project demonstrates a full-stack Business Intelligence workflow, transforming raw relational database records into an interactive Power BI dashboard. Unlike standard CSV-based projects, this analysis is built on top of a **local PostgreSQL server**, utilizing advanced SQL for data preparation, DAX for dynamic ranking, and Python for statistical distribution visualization.

![Dashboard Preview](dashboard_main.png)
*(Note: Replace `dashboard_main.png` with a high-quality screenshot of your final dashboard)*

## 🛠️ Architecture & Tech Stack
* **Database Management:** PostgreSQL (pgAdmin 4).
* **Data Engineering:** SQL (Complex `JOIN` operations to create an analytical `VIEW`).
* **Business Intelligence:** Power BI (Direct database connection, Data Modeling).
* **Advanced Analytics:** Python (`seaborn`, `matplotlib` for statistical distribution modeling within Power BI).
* **Metrics & Logic:** DAX (Dynamic filtering, Context modification).

---

## ⚙️ Development Workflow & Code Snippets

### 1. Data Engineering (PostgreSQL)
Instead of importing 15 separate raw tables into Power BI, I optimized the data model at the database level by writing a SQL script to join 6 normalized tables (Payments, Rentals, Inventory, Films, Categories, Customers) into a single, flat analytical `VIEW`.

```sql
CREATE OR REPLACE VIEW public.sales_analytics AS
SELECT 
    p.payment_id,
    DATE(p.payment_date) AS payment_date,
    p.amount,
    c.first_name || ' ' || c.last_name AS customer_name,
    f.title AS film_title,
    cat.name AS genre,
    ci.city
FROM payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category cat ON fc.category_id = cat.category_id
JOIN customer c ON p.customer_id = c.customer_id
JOIN address a ON c.address_id = a.address_id
JOIN city ci ON a.city_id = ci.city_id;
```

### 2. Dynamic DAX Ranking (Power BI)
To dynamically track the highest-paying clients regardless of the applied slicers, I implemented a custom DAX measure using the `RANKX` function to establish a "Top 10 Customers" logic.

```dax
Customer Rank = 
RANKX(
    ALL('public sales_analytics'[customer_name]), 
    [Total Revenue], 
    BLANK(), 
    DESC, 
    Dense
)
```

### 3. Statistical Analysis with Python
Power BI's native visuals are limited when it comes to showing data distribution. I integrated a Python script using the `seaborn` library directly into the dashboard to generate a Boxplot, revealing the variance and outliers in payment amounts across different film genres.

```python
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="darkgrid")
plt.figure(figsize=(8, 5))

ax = sns.boxplot(
    x="amount", 
    y="genre", 
    data=dataset, 
    palette="viridis",
    linewidth=1.5
)

plt.title("Distribution of Payment Amounts by Genre")
plt.xlabel("Payment Amount ($)")
plt.ylabel("Genre")
plt.tight_layout()
plt.show()
```

## 💡 Key Business Insights
1. **Top Performing Categories:** The `Sports`, `Sci-Fi`, and `Animation` genres consistently generate the highest revenue volume.
2. **Payment Distribution (Python Insight):** The Python-generated boxplot reveals that while standard rental prices are tightly grouped, certain genres have significant positive outliers (late fees or premium rentals) that positively skew the revenue.
3. **Customer Retention:** A small segment of VIP customers contributes a disproportionately large share of total revenue, highlighting the need for a targeted loyalty program.
