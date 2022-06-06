# ➡️steps I followed:

Designed an interactive report to analyze and visualize the data.
             
             A.connected and shaped the source data
             
1. `Updated Power BI options and settings as follows:` 

  - Deselect the "Autodetect new relationships after data is loaded" option in the Data Load tab.
  - Make sure that Locale for import is set to "English (United States)" in the Regional Settings tab.

2. `Connected to the MavenMarket_Customers csv file`

  - Named the table "Customers", and make sure that headers have been promoted.
  - Confirmed that data types are accurate (like: "customer_id" should be whole numbers, and both "customer_acct_num" and "customer_postal_code" should be text).
  - Added a new column named "full_name" to merge the the "first_name" and "last_name" columns, separated by a space.
  - Created a new column named "birth_year" to extract the year from the "birthdate" column, and format as text.
  - Created a conditional column named "has_children" which equals "N" if "total_children" = 0, otherwise "Y".

3. `Connected to the MavenMarket_Products csv file`

  - Named the table "Products" and make sure that headers have been promoted.
  - Confirmed that data types are accurate (like: "product_id" should be whole numbers, "product_sku" should be text), "product_retail_price" and "product_cost" 
    should be decimal numbers).
  - Used the statistics tools to return the number of distinct product brands, followed by distinct product names.
  - Added a calculated column named "discount_price", equal to 90% of the original retail price.
      (`= Table.AddColumn(#"Changed Type", "Multiplication", each [product_retail_price] * 0.9, type number)`)
  - Format as a fixed decimal number, and then used the rounding tool to round to 2 digits
  - Selected "product_brand" and used the `Group By` option to calculate the average retail price by brand, and name the new column "Avg Retail Price"
  - Deleted the last applied step to return the table to its pre-grouped state
  - Replaced "null" values with zeros in both the "recyclable" and "low-fat" columns by using `Replace values` in Transform tab.

4. `Connectd to the MavenMarket_Stores csv file`

   - Named the table "Stores" and make sure that headers have been promoted.
   - Confirmd that data types are accurate (Like: "store_id" and "region_id" should be whole numbers).
   - Added a calculated column named "full_address", by merging "store_city", "store_state", and "store_country", separated by a comma and space 
     (Used concatenation in formula).
   - Added a calculated column named "area_code", by extracting the characters before the dash ("-") in the "store_phone" field .

5. `Connect to the MavenMarket_Regions csv file`

   - Named the table "Regions" and make sure that headers have been promoted.
   - Confirmed that data types are accurate (like: "region_id" should be whole numbers).

6. `Connected to the MavenMarket_Calendar csv file`

   - Named the table "Calendar" and make sure that headers have been promoted.
   - Used the `date tools` in the query editor to add the following columns:
      - Start of Week (starting Sunday)
      - Name of Day
      - Start of Month
      - Name of Month
      - Quarter of Year
      - Year

7. `Connect to the MavenMarket_Returns csv file`

    - Named the table "Return_Data" and make sure that headers have been promoted.
    - Confirmed that data types are accurate (all ID columns and quantity should be whole numbers)

8. `Added a new folder and named "MavenMarket Transactions", containing both the MavenMarket_Transactions_1997 and MavenMarket_Transactions_1998 csv files`

    - Connected to the folder path, and choosed "Edit" (vs. Combine and Edit).
    - Clicked the "Content" column header (double arrow icon) to combine the files, then remove the "Source.Name" column.
    - Named the table "Transaction_Data", and confirm that headers have been promoted.
    - Confirmed that data types are accurate (all ID columns and quantity should be whole numbers).

9. `With the exception of the two data tables, disable "Include in Report Refresh", then Close & Apply.`
---
                  B. Built a relational data model.
                  
1. `In the RELATIONSHIPS view, arranged tables with the lookup tables above the data tables.`

    - Connected Transaction_Data to Customers, Products, and Stores using valid primary/foreign keys. 
    - Connected Transaction_Data to Calendar using both date fields, with an inactive "stock_date" relationship.
    - Connected Return_Data to Products, Calendar, and Stores using valid primary/foreign keys.
    - Connected Stores to Regions as a `snowflake schema`.

2. `Confirmed the following:`

    - All relationships followed one-to-many cardinality, with primary keys (1) on the lookup side and foreign keys (*) on the data side.
    - Filters are all one-way (no two-way filters).
    - Filter context flows "downstream" from lookup tables to data tables.
    - Data tables are connected via shared lookup tables (not directly to each other)

3. Hide all foreign keys in both data tables from Report View, as well as "region_id" from the Stores table

4. In the DATA view, completed the following:
    - Updated all date fields (across all tables) to the "M/d/yyyy" format using the formatting tools in the Modeling tab.
    - Updated "product_retail_price", "product_cost", and "discount_price" to Currency ($ English) format.
    - In the Customers table, categorize "customer_city" as City, "customer_postal_code" as Postal Code, and "customer_country" as Country/Region.
    - In the Stores table, categorize "store_city" as City, "store_state" as State or Province, "store_country" as Country/Region, and "full_address" as Address 
---
                    
                  C. Added required calculated coloums and DAX measures.

1. `In the DATA view, added the following calculated columns:`
   - In the Calendar table, added a column named "Weekend"
      - Equals "Y" for Saturdays or Sundays (otherwise "N")  --> `Weekend = IF(OR('Calendar'[Day Name] = "Sunday",'Calendar'[Day Name] = "Saturday"),"Y","N")`
   - In the Calendar table, added a column named "End of Month"
      - Returns the last date of the current month for each row --> `End of Month = ENDOFMONTH('Calendar'[date])`
