# Model-Car-Database-Analysis-using-MySQL-Workbench
This project is dedicated to performing data analysis within the framework of the imaginary organization, Mint Classics. In this scenario, I will be taking on the role of a beginner data analyst, supporting the company in resolving challenges related to inventory management and storage facilities. The primary concern involves evaluating whether to shut down one of the current storage facilities.

**Tools Used: -** MySQL Workbench 8.0 CE

**Tasks: -**
1.	Importing the classic car Model Database.
2.	Understanding the Database and ERD (Entity Relationship Diagram).
3.	Understanding the business issue.
4.	Preprocessing the tables for analysis.
5.	Formulating the queries for in-depth analysis.
6.	Crafting conclusion and recommendations.

**Entity Relationship Diagram of Mint classic Database: -**

![image](https://github.com/him100gupta/Model-Car-Database-Analysis-using-MySQL-Workbench/assets/29143253/1fcaa5db-b87c-4f23-8ecb-1a158d9eee8d)

**Approach: -**
Initiated the analysis by examining fundamental metrics related to the warehouses, followed by an assessment of each warehouse's performance metrics, including total sales, average sales amounts, and the percentage of successful shipment rates. The objective was to identify warehouses that may be approaching closure. Subsequently, delved into evaluating the influence of products on warehouse performance and investigated whether employee performance and customers have an impact on warehouse efficiency.

**The following will be analyzed in this project: -**

1. What is the total quantity of product in each inventory?
   ``` SQL
   SELECT w.warehouseCode, w.warehouseName, SUM(p.quantityInStock) as Total_Inventory
   from warehouses w
   join products p on w.warehouseCode = p.warehouseCode
   group by w.warehouseCode, w.warehouseName
   order by Total_Inventory desc;
   ```
2. What is the product category count and their product line percentage?
   ``` SQL
   select productLine, warehouseCode, count(productLine) as ProductCatCount, 
   cast(count(productLine)*100/ (select count(productLine) from products)as decimal(10,2)) as CategoryPercentage
   from products
   group by productLine, warehouseCode
   order by CategoryPercentage desc;
   ```
3. What is the overall quantity of products shipped from each warehouse?
   ``` SQL

   ```
5. What’s the total sales amount and average sales amount of each warehouse?
   ``` SQL

   ```
6. Are warehouses making enough profit through sales?
   ``` SQL

   ```
7. Are the products effectively shipped from the warehouse?
   ``` SQL

   ```
8. Was there any drastic change in product demand across the years?
   ``` SQL

   ```
9. Are there any product lines showing exceptional performance or underperforming?
   ``` SQL

   ```
10. What product line yields the highest profit for the company?
    ``` SQL

    ```
11. Was there any product that was in demand but was out of stock?
    ``` SQL

    ```
12. Are there products with high inventory but low sales?
    ``` SQL

    ```
13. Who are the customers making the most significant contributions to sales?
    ``` SQL

    ```
14. What is the purchasing trend among customers? Are there customers who did not place any orders within the specified timeframe?
18. Do customers have outstanding balances?
19. Are there customers with credit issues that need to be addressed?
20. How are employees performing based on their overall sales?
21. Are employees effectively able to secure deals with clients?

## Should Mint Classics consider the closure of one of their storage facilities?

I strongly oppose the closure of any warehouses. Upon thorough examination of the Mint Classic database, it's evident that the East warehouse outperforms others, particularly in sales, shipment, and efficiency metrics. Notably, products from the East warehouse command an average sales price of $108.46, surpassing the average prices of products from other warehouses, which are below $100. Moreover, the demand for Classic Cars products, housed in the East inventory, is notably high compared to other lines, except for Vintage Cars. However, the profit margins generated by each warehouse and product line, except for Trains, are nearly identical which is approximately 40%.

This analysis underscores the significant influence of various factors on warehouse performance, with notable contributors being product cost and demand. Product demand is primarily driven by customer behavior, whereas product pricing is controlled by the company. Therefore, it can be inferred that optimizing warehouse performance necessitates a holistic consideration of these factors.

Given these insights, closing any storage facility could potentially disrupt operations and hinder overall performance. Therefore, Mint Classics should maintain its existing storage infrastructure, leveraging the strengths of warehouses to drive continued success and profitability.
