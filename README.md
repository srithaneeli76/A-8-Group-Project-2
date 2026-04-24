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

# Queries 
