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
