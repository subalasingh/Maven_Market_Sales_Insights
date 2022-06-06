# ➡️steps I followed in detail:
           
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

3. `Hide all foreign keys in both data tables from Report View, as well as "region_id" from the Stores table`

4. `In the DATA view, completed the following:`
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
   - In the Customers table, added a column named "Current Age"
      - Calculates current customer ages using the "birthdate" column and the TODAY() function --> `Current Age = DateDiff(Customers[birthdate],Today(),year)`
   - In the Customers table, added a column named "Priority"
      - Equals "High" for customers who own homes and have Golden membership cards (otherwise "Standard") -->`Priority = IF(AND(Customers[homeowner] =    "Y",Customers[member_card] = "Golden"),"High","Standard")`
   - In the Customers table, added a column named "Short_Country"
      - Returns the first three characters of the customer country, and converts to all uppercase --> `Short_Country = UPPER(LEFT(Customers[customer_country],3))`
   - In the Customers table, added a column named "House Number"
      - Extracts all characters/numbers before the first space in the "customer_address" column --> `House Number = LEFT(Customers[customer_address],Search(" ",Customers[customer_address])-1)`
   - In the Products table, added a column named "Price_Tier"
      - Equals "High" if the retail price is >$3, "Mid" if the retail price is >$1, and "Low" otherwise --> `Price_Tier = IF(Products[product_retail_price]>3,"High",IF(Products[product_retail_price]>1,"Mid","Low"))`
   - In the Stores table, added a column named "Years_Since_Remodel"
      - Calculates the number of years between the current date (TODAY()) and the last remodel date --> `Years_Since_Remodel = DATEDIFF(Stores[last_remodel_date], TODAY(),YEAR)`

2. `In the REPORT view, added the following measures.`
   - Created new measures named "Quantity Sold" and "Quantity Returned" to calculate the sum of quantity from each data table.
      - `Quantity Sold = SUM('MavenMarket Transactions'[quantity])` `Quantity Returned = SUM(Return_Data[quantity])`
   - Created new measures named "Total Transactions" and "Total Returns" to calculate the count of rows from each data table.
      - `Total Transactions = Countrows(MavenMarket Transactions)` `Total Returns = Countrows(Return_Data)`
   - Created a new measure named "Return Rate" to calculate the ratio of quantity returned to quantity sold (format as %)
      - `Return Rate = [Quantity Returned]/[Quantity Sold]`
   - Created a new measure named "Weekend Transactions" to calculate transactions on weekends.
      - `Weekend1 = RELATED('Calendar'[Weekend])` --> `Weekend Transactions = CALCULATE(COUNT('MavenMarket Transactions'[transaction_date]),'MavenMarket Transactions'[Weekend1] = "Y")` 
   - Created a new measure named "% Weekend Transactions" to calculate weekend transactions as a percentage of total transactions (format as %)
      - `% Weekend Transactions = [Weekend Transactions]/'MavenMarket Transactions'[Total Transactions]`
   - Created a new measure to calculate "Total Revenue" based on transaction quantity and product retail price, and format as $.
      - `Total Revenue = SUMX('MavenMarket Transactions', 'MavenMarket Transactions'[quantity]*'MavenMarket Transactions'[Retail Price])`
   - Created a new measure to calculate "Total Cost" based on transaction quantity and product cost, and format as $.
      - `Total Cost = SUMX('MavenMarket Transactions', 'MavenMarket Transactions'[quantity]*'MavenMarket Transactions'[product cost price])`
   - Created a new measure named "Total Profit" to calculate total revenue minus total cost, and format as $.
      - `Total Profit = 'MavenMarket Transactions'[Total Revenue] - 'MavenMarket Transactions'[Total Cost]`
   - Created a new measure to calculate "Profit Margin" by dividing total profit by total revenue (format as %).
      - `Profit Margin = 'MavenMarket Transactions'[Total Profit]/'MavenMarket Transactions'[Total Revenue]`
   - Created a new measure named "Unique Products" to calculate the number of unique product names in the Products table
      - `Unique Products = DISTINCTCOUNT(Products[product_id])`
   - Created a new measure named "YTD Revenue" to calculate year-to-date total revenue, and format as $.
      - `YTD Revenue = Calculate('MavenMarket Transactions'[Total Revenue], DATESYTD('Calendar'[date]))`
   - Created new measures named  "Last Month Transactions", "Last Month Revenue", "Last Month Profit", and "Last Month Returns"
      - `Last Month Transactions = CALCULATE('MavenMarket Transactions'[Total Transactions],DATEADD('Calendar'[date],-1,MONTH))` 
      - `Last Month Revenue = CALCULATE('MavenMarket Transactions'[Total Revenue],DATEADD('Calendar'[date],-1,MONTH))`
      - `Last Month Profit = CALCULATE('MavenMarket Transactions'[Total Profit],DATEADD('Calendar'[date],-1,MONTH))`
      - `Last Month Returns = CALCULATE(Return_Data[Total Returns],DATEADD('Calendar'[date],-1,MONTH))`
   - Created a new measure named "Revenue Target" based on a 5% lift over the previous month revenue, and format as $
      - `Revenue Target = 'MavenMarket Transactions'[Last Month Revenue]*1.05`
---
                 
                 D. Designed an interactive report to analyze and visualize the data.

1. Inserted a Matrix visual to show `Total Transactions`, `Total Profit`, `Profit Margin`, and `Return Rate` by Product_Brand (on rows).
  - Added conditional formatting to show `data bars` on the Total Transactions column, and `color scales` on Profit Margin and Return Rate.
  - Added a visual level `Top N filter` to only show the top 30 product brands, then sort descending by Total Transactions.

2. Added a KPI Card to show `Total Transactions`, with `Start of Month as the trend axis` and `Last Month Transactions as the target goal`.
  - Updated the title to "Current Month Transactions".
  - Created two more copies: one for `Total Profit (vs. Last month Profit)` and one for `Total Returns (vs. Last Month Returns)`.
  - Change the `Total Returns` to `color coding to "Low is Good"`

3. Added a Map visual to show `Total Transactions by store city`.
  - Added a slicer for store country.
  - Under the "selection controls" menu in the formatting pane, activated the "Show Select All" option.
      - Changed the orientation in the "General" formatting menu to horizontal and resize to create a vertical stack (rather than a list)

4. Added a Treemap visual to break down `Total Transactions by store country`.
  - Pulled in `store_state` and `store_city` beneath store_country in the "Group" field to enable drill-up and drill-down functionality.

5. Added a Column Chart to show `Total Revenue by week` with tittle "Weekly Revenue Trending"
  - Added a report level filter to only show data for 1998

6. Added a Gauge Chart to show `Total Revenue against Revenue Target` with title "Revenue vs. Target
   - Added a visual level Top N filter to show the latest Start of Month

7. Selected the Matrix and activated the `Edit interactions option` (in format pane) to prevent the Treemap from filtering.
---
