# A-8-Group-Project-2
Group Name: A8 

Member Names: 
1. Sritha Neeli (Group Leader) 
2. Aman Bhavsar (SQL Writer) 
3. Dylan Foley (Database Designer) 
4. Luzmia Belman (Data Wrangler) 
5. Daniel Choi (Conceptual Modeler) 

## Case Description:

Northline Outfitters is an expanding e-commerce retailer focused on selling student-friendly tech and lifestyle accessories  in the United States and Canada. They buy their merchandise from different vendors and then sell it directly to their customers. Some of their data challenges include mixed format dates, country indicators embedded in identifiers, metric and imperial measurements, customer information embedded in a single text field, inconsistent payment, discount, and tax formats, and duplicate-looking products or variant rows. 

# Conceptual Model



# Data quality assessment
  The data quality assessment of the provided dataset identifies several significant issues that could affect the accuracy of the analysis. First off, there is a lot of missing/incomplete data across the Sales_Dump and Product_Supplier_Master tables, specifically in the customer email, discount, tax, and reorder levels fields, which can lead to inaccurate reporting. Duplicate records are present, especially with repeated SKU values. This could lead to inconsistencies and double-counting during data integration. The dataset also has inconsistent formatting where there is variation in SKU casing ( uppercase vs lowercase) and unstandardized text fields. This could disrupt joins and filtering performance. Moreover, many of the attributes were stored as text instead of being numeric or date-based. This prevents proper calculations. Invalid/inconsistent values are seen, such as mixed representations of return flags, which can cause complications in interpretation. There are structural issues seen in fields such as customer information and shipping details, which are stored in a single column, reducing query efficiency. Lastly, there are inconsistencies between tables, especially in SKU representation, which can weaken data integrity and constrain accurate relationships.

# Data Cleaning Process
To transform this into a production-ready dataset, we followed a structured cleaning workflow designed to enforce consistency and logic:

Columnar Decoupling (Splitting): We dismantled concatenated strings. For customer and vendor info, we identified the spaces or delimiters between names and emails to move them into dedicated columns (First Name, Last Name, Email). This allows the database to treat each piece of information as a unique, searchable attribute.

Character Stripping & Casting: We "scrubbed" the numerical columns by removing all non-numeric characters (such as "$", "units", or "lbs"). Once the text noise was removed, we converted these columns into pure integers or decimals so they could be used for financial totaling.

Case Standardization: We applied a global casing rule across the entire dataset. Payment methods and SKU numbers were forced into uppercase for uniformity, while product descriptions were adjusted to proper noun capitalization to ensure professional-looking customer invoices.

Logic-Based Value Filling: To resolve the Null value ambiguity, we implemented an automated "Default to No" rule. Any empty cell in a status-related column (like Return Flags or Discontinued Status) was filled with an "N," ensuring every row had a definitive, actionable value.

Unit Normalization: For the physical dimensions (Weight and Height), we performed mathematical conversions on the raw data. By identifying the unit label (like oz or lbs), we converted those specific values into a single, uniform metric (grams) across the entire sheet, ensuring that an item weighing "1 lb" was no longer numerically smaller than an item weighing "500 g."

Shipping ID Reconstruction: We found that some records in the Orders table were missing ship_id values. Since the project required revenue to be analyzed by country, we reconnected orders to the cleaned Shipping table by matching the row order of the cleaned orders and shipping records. This allowed each order to connect to a shipping country for analysis.

# Queries 

### Query #1: Which products generated the highest total sales revenue, by country?
Code:
```SQL
SELECT 
    s.ship_country,
    p.sku,
    p.product_description,
    p.category,
    SUM(od.line_total) AS total_sales_revenue
FROM OrderDetails od
JOIN Orders o 
    ON od.order_id = o.order_id
JOIN Shipping s 
    ON o.ship_id = s.ship_id
JOIN Products p 
    ON od.sku = p.sku
GROUP BY 
    s.ship_country,
    p.sku,
    p.product_description,
    p.category
ORDER BY 
    s.ship_country ASC,
    total_sales_revenue DESC;
```
Query Results:
<img width="553" height="637" alt="Screenshot 2026-04-24 at 6 21 55 PM" src="https://github.com/user-attachments/assets/70269a00-05e1-428c-9811-6eee5d5cacdd" />

### Query #2: Which employees handled the largest number of orders, and how do their results compare with other?
Code:
```SQL
SELECT
    e.manager_ref,
    o.employee_id,
    COUNT(o.order_id) AS total_orders_handled
FROM Orders o
JOIN Employee e
    ON o.employee_id = e.employee_id
GROUP BY 
    e.manager_ref,
    o.employee_id
ORDER BY 
    e.manager_ref ASC,
    total_orders_handled DESC;
```
Query Results:
<img width="277" height="217" alt="Screenshot 2026-04-24 at 6 12 49 PM" src="https://github.com/user-attachments/assets/40f7382c-73a5-4281-88bd-aedb97185e3a" />

### Query #3: Which vendors supply products that appear in more than one category?
Code:
```SQL
SELECT
    v.vendor_id,
    v.vendor_name,
    COUNT(DISTINCT p.category) AS number_of_categories,
    GROUP_CONCAT(DISTINCT p.category ORDER BY p.category SEPARATOR ', ') AS categories_supplied
FROM Vendors v
JOIN Supply s
    ON v.vendor_id = s.vendor_id
JOIN Products p
    ON s.sku = p.sku
GROUP BY 
    v.vendor_id,
    v.vendor_name
HAVING 
    COUNT(DISTINCT p.category) > 1
ORDER BY 
    number_of_categories DESC,
    v.vendor_name ASC;
```
Query Results:
<img width="709" height="134" alt="Screenshot 2026-04-24 at 5 30 15 PM" src="https://github.com/user-attachments/assets/b926c49c-e749-4bad-beed-b7cc1e1bfb40" />


### Query #4: Which product categories generate the most revenue and have the highest average order line value?

Business Justification: This helps Northline Outfitters identify which categories bring in the most money and which categories have stronger sales per order line. Management can use this to focus promotions, inventory decisions, and product planning on the strongest categories.

Code:
```SQL
SELECT
    p.category,
    COUNT(DISTINCT od.order_id) AS number_of_orders,
    SUM(od.quantity) AS total_units_sold,
    SUM(od.line_total) AS total_revenue,
    ROUND(AVG(od.line_total), 2) AS average_line_value
FROM OrderDetails od
JOIN Products p
    ON od.sku = p.sku
GROUP BY 
    p.category
ORDER BY 
    total_revenue DESC;
```
Query Results:
<img width="531" height="234" alt="Screenshot 2026-04-24 at 5 57 42 PM" src="https://github.com/user-attachments/assets/42b980fa-0a6e-43ff-b232-93d6522ce0c3" />

### Query #5: Which customers have the highest total spending, and are they loyalty customers, students, or guest checkouts?

Business Justification: This helps the company identify valuable customer groups. Management can use this to decide whether loyalty customers, students, or regular customers are contributing the most revenue.

Code:
```SQL
SELECT
    c.customer_id,
    CONCAT(c.customer_f_nm, ' ', c.customer_l_nm) AS customer_name,
    c.customer_email,
    c.loyalty_customer,
    c.is_student,
    c.guest_checkout,
    COUNT(DISTINCT o.order_id) AS total_orders,
    SUM(od.line_total) AS total_spent
FROM Customers c
JOIN Orders o
    ON c.customer_id = o.customer_id
JOIN OrderDetails od
    ON o.order_id = od.order_id
GROUP BY
    c.customer_id,
    c.customer_f_nm,
    c.customer_l_nm,
    c.customer_email,
    c.loyalty_customer,
    c.is_student,
    c.guest_checkout
ORDER BY
    total_spent DESC;
```
Query Results:
<img width="666" height="254" alt="Screenshot 2026-04-24 at 5 55 16 PM" src="https://github.com/user-attachments/assets/9dc2e626-6e8e-4f9d-8a45-25f30a766107" />

### Query #6: Which discontinued products have still appeared in customer orders, and how much revenue did they generate?

Business Justification: This helps Northline Outfitters identify discontinued products that are still being sold or appearing in orders. If discontinued products continue generating sales, management may need to check inventory records, product availability, or decide whether similar replacement products should be offered.

Code:
```SQL
SELECT
    p.sku,
    p.product_description,
    p.category,
    p.discontinued,
    COUNT(DISTINCT od.order_id) AS number_of_orders,
    SUM(od.quantity) AS total_units_sold,
    SUM(od.line_total) AS total_revenue
FROM Products p
JOIN OrderDetails od
    ON p.sku = od.sku
WHERE 
    p.discontinued = 'Y'
GROUP BY
    p.sku,
    p.product_description,
    p.category,
    p.discontinued
ORDER BY
    total_revenue DESC;
```
Query Results:
<img width="684" height="76" alt="Screenshot 2026-04-24 at 5 59 35 PM" src="https://github.com/user-attachments/assets/a914dd12-16a1-42d1-b30f-3badca0232ee" />
