**The following queries were used to analyse the Business issues: - **

1. What is the total quantity of product in each inventory?
   ``` SQL
   SELECT w.warehouseCode, w.warehouseName, SUM(p.quantityInStock) as Total_Inventory
   from warehouses w
    join products p on w.warehouseCode = p.warehouseCode
   group by w.warehouseCode, w.warehouseName
   order by Total_Inventory desc;
   ```
   This query allows you to observe the overall inventory for each warehouse.
2. What is the product category count and their product line percentage?
   ``` SQL
   select productLine, warehouseCode, count(productLine) as ProductCatCount, 
   cast(count(productLine)*100/ (select count(productLine) from products)as decimal(10,2)) as CategoryPercentage
   from products
   group by productLine, warehouseCode
   order by CategoryPercentage desc;
   ```
   This query provides information on the product count within various product lines and the respective percentage distribution of products in each line.
3. What is the overall quantity of products shipped from each warehouse?
   ``` SQL
   Select warehouseName, warehouseCode, TotalquantityShipped
   from (Select w.warehouseName, w.warehouseCode, sum(od.quantityOrdered) as TotalquantityShipped
   from orderdetails od 
    join products p on od.productCode = p.productCode
    join warehouses w on p.warehouseCode = w.warehouseCode 
   Group by warehouseName, warehouseCode) as TotalquantityOrdered
   order by TotalquantityShipped desc;
   ```
   This query provides information about the number of products shipped from each warehouse.
4. What’s the total sales amount and average sales amount of each warehouse?
   ``` SQL
   Select warehouseName, warehouseCode, TotalSalesAmount, AverageSales
   from (
         Select w.warehouseName, w.warehouseCode,
         sum(CASE WHEN o.status IN ("Shipped","Resolved") THEN od.quantityOrdered * od.priceEach ELSE 0 END) as TotalSalesAmount,
         Cast(sum(CASE WHEN o.status IN ("Shipped","Resolved" THEN od.quantityOrdered * od.priceEach ELSE 0 END) / sum(CASE WHEN o.status IN ("Shipped","Resolved") THEN od.quantityOrdered  ELSE 0 END)as decimal(10,2))
         as AverageSales
         from warehouses w 
          join products p on p.warehouseCode = w.warehouseCode 
          join orderdetails od on p.productCode = od.productCode
          join orders o on od.orderNumber = o.orderNumber
         Group by warehouseName, warehouseCode) profit
   order by AverageSales desc;
   ```
   This query provides insights into the total and average sales for each warehouse. 
5. Are warehouses making enough profit through sales?
   ``` SQL
   Select warehouseName, warehouseCode, 
   cast(((TotalSalesAmount - TotalBuyAmount)/TotalSalesAmount)*100 as decimal(10,2)) as profit_margin
   from (
         Select w.warehouseName, w.warehouseCode , 
         sum( od.quantityOrdered * od.priceEach) as TotalSalesAmount,
         sum(od.quantityOrdered * p.buyPrice) as TotalBuyAmount
         from warehouses w 
          join products p on p.warehouseCode = w.warehouseCode 
          join orderdetails od on p.productCode = od.productCode
          join orders o on od.orderNumber = o.orderNumber
         where o.status = "Shipped" or o.status = "Resolved"
         Group by warehouseName, warehouseCode) profit
   order by profit_margin desc;
   ```
   This query offers insights into the profit margin generated by each warehouse through product sales.
6. Are the products effectively shipped from the warehouse?
   ``` SQL
   select w.warehouseName,w.warehouseCode,
   CAST((Sum( CASE WHEN o.status = "Shipped" or o.status = "Resolved" THEN 1 ElSE 0 END)/COUNT(o.orderNumber))  * 100 as decimal(10,2)) as SuccessfulShipmentPer
   from warehouses w
    join products p on w.warehouseCode = p.warehouseCode
    join orderdetails od on p.productCode = od.productCode
    join orders o on od.orderNumber = o.orderNumber
   group by w.warehouseCode, w.warehouseName
   order by SuccessfulShipmentPer desc;
   ```
   This query provides insights into the success rate of product shipment for each warehouse.
7. Was there any drastic change in product demand across the years?
   ``` SQL
   SELECT p.productCode, p.productName,
    SUM(case when YEAR(o.orderDate) = 2003 then od.quantityOrdered else 0 end) as quantityOrderedIn_2003,
    cast(case when COUNT(case when YEAR(o.orderDate) = 2003 then od.priceEach else null end) > 0
         then AVG(case when YEAR(o.orderDate) = 2003 then od.priceEach end) else 0 end as decimal(10,2)) as AveragePrice_2003,
         
    SUM(case when YEAR(o.orderDate) = 2004 then od.quantityOrdered else 0 end) as quantityOrderedIn_2004,
    cast(case when COUNT(case when YEAR(o.orderDate) = 2004 then od.priceEach else null end) > 0
         then AVG(case when YEAR(o.orderDate) = 2004 then od.priceEach end) else 0 end as decimal(10,2)) as AveragePrice_2004,
         
    SUM(case when YEAR(o.orderDate) = 2005 then od.quantityOrdered else 0 end) as quantityOrderedIn_2005,
    cast(case when COUNT(case when YEAR(o.orderDate) = 2005 then od.priceEach else null end) > 0
         then AVG(case when YEAR(o.orderDate) = 2005 then od.priceEach end) else 0 end as decimal(10,2)) as AveragePrice_2005
   FROM products p 
    INNER JOIN orderdetails od ON p.productCode = od.productCode
    INNER JOIN orders o ON od.orderNumber = o.orderNumber
   GROUP BY p.productCode, p.productName
   ORDER BY p.productCode;
   ```
   This query provides insights of the products demand across the years.
8. Are there any product lines showing exceptional performance or underperforming?
   ``` SQL
   Select  p.warehouseCode, pl.productLine,
      sum(CASE WHEN o.status IN ("Shipped","Resolved") THEN od.quantityOrdered * od.priceEach ELSE 0 END) as TotalSales,
	   Cast(sum(CASE WHEN o.status IN ("Shipped","Resolved")  THEN od.quantityOrdered * od.priceEach ELSE 0 END)
      / sum(CASE WHEN o.status IN ("Shipped","Resolved")  THEN od.quantityOrdered ELSE 0 END) as decimal(10,2))  as AverageSales,
      cast(sum(CASE WHEN o.status IN ("Shipped","Resolved") THEN od.quantityOrdered  ELSE 0 END)as decimal(10,2)) as totalQuantityOrdered
   from productlines pl 
    join products p on pl.productLine = p.productLine
    join orderdetails od on p.productCode = od.productCode
    join orders o on od.orderNumber = o.orderNumber
   Group by p.warehouseCode, pl.productLine;
   ```
   This query gives insights about the total sales and average sales of each product line.
9. What product line yields the highest profit for the company?
    ``` SQL
   select warehouseCode, productLine, cast(((TotalSales - TotalSales1)/TotalSales)*100 as decimal(10,2)) as profitMargin
   from(
        Select  p.warehouseCode, pl.productLine, sum( od.quantityOrdered * od.priceEach ) as TotalSales, sum(od.quantityOrdered * p.buyPrice) as TotalSales1
        from productlines pl 
         join products p on pl.productLine = p.productLine
         join orderdetails od on p.productCode = od.productCode
         join orders o on od.orderNumber = o.orderNumber
        WHERE o.status = 'Shipped' or o.status = 'Resolved'
        Group by p.warehouseCode, pl.productLine) as profit
   order by profitMargin desc;
    ```
    This query offers insights into the profit margin produced by each product line.
10. Was there any product that was in demand but was out of stock?
    ``` SQL
    select productCode, productName,  productLine, warehouseCode, quantityInStock, quantityOrdered,
    (quantityInStock - quantityOrdered) as InventoryShortage
    from (
          Select p.productName, p.quantityInStock,od.productCode,p.warehouseCode,
          sum(od.quantityOrdered) as quantityOrdered, pl.productLine
          from productlines pl 
           join products p on pl.productLine = p.productLine
           join orderdetails od on p.productCode = od.productCode
          group by p.productName, p.quantityInStock, od.productCode , pl.productLine) AS InventoryShortage
    where (quantityInStock - quantityOrdered) < 0
    order by productLine, InventoryShortage asc ;
    ```
    This query fetches details about products that experienced demand but were unavailable in stock.
11. Are there products with high inventory but low sales?
    ``` SQL
    select warehouseCode,warehouseName,productName, productLine,quantityInStock, quantityOrdered, 
    cast((quantityOrdered / (quantityInStock))*100 as decimal(10,2)) as SellThroughRatePer
    from (
          Select w.warehouseName,  W.warehouseCode, p.productName,p.productLine,
          p.quantityInStock, p.productCode,  sum(od.quantityOrdered) as quantityOrdered
          from 
          warehouses w 
           join products p on w.warehouseCode = p.warehouseCode 
           join orderdetails od on p.productCode = od.productCode
          group by p.productCode ) AS sells
    where quantityInStock > quantityOrdered 
    order by SellThroughRatePer desc;
    ```
    This query provides insights into product performance by computing their sell-through rate.
    ```SQL
    select warehouseCode,warehouseName,productName, productLine,quantityInStock, quantityOrdered, cast((quantityOrdered / (quantityInStock))*100 as decimal(10,2)) as SellThroughRatePer
    from (
          Select w.warehouseName,  W.warehouseCode, p.productName,p.productLine,
          p.quantityInStock, p.productCode,  sum(od.quantityOrdered) as quantityOrdered
          from 
          warehouses w 
           join products p on w.warehouseCode = p.warehouseCode 
           join orderdetails od on p.productCode = od.productCode
          group by p.productCode ) AS sells
    where warehouseCode = "a" AND quantityInStock > quantityOrdered 
    order by SellThroughRatePer desc
    limit 5;
    ```
    To assess the product performance from each warehouse, the same query can be employed with a simple modification in the WHERE clause.
12. Who are the customers making the most significant contributions to sales?
    ``` SQL
    select c.customerName, count(distinct o.orderNumber) as NumberOfOrders, sum(od.quantityOrdered) as TotalQuantityBought,
    sum(od.quantityOrdered*od.priceEach) as TotalPurchaseAmount
    from customers c 
     join orders o on c.customerNumber = o.customerNumber
     join orderdetails od on o.orderNumber = od.orderNumber
    group by c.customerName
    order by TotalPurchaseAmount desc;
    ```
    This query gives insights about the customers overall purchase amount.
13. What is the purchasing trend among customers? Are there customers who did not place any orders within the specified timeframe?
    ``` SQL
    select c.customerNumber, c.customerName,
    sum(CASE WHEN YEAR(o.orderDate) = 2003 THEN 1 ELSE 0 END) as quantityOrderedIn_2003,
    sum(CASE WHEN YEAR(o.orderDate) = 2004 THEN 1 ELSE 0 END) as quantityOrderedIn_2004,
    sum(CASE WHEN YEAR(o.orderDate) = 2005 THEN 1 ELSE 0 END) as quantityOrderedIn_2005
    from 
    customers c 
      join orders o on c.customerNumber = o.customerNumber
    group by c.customerNumber, c.customerName
    order by c.customerNumber ;
    ```
    This query gives information about the no. of order customers places each year.
    ``` SQL
    select c.customerNumber, c.customerName,
    sum(CASE WHEN YEAR(o.orderDate) = 2003 THEN 1 ELSE 0 END) as quantityOrderedIn_2003,
    sum(CASE WHEN YEAR(o.orderDate) = 2004 THEN 1 ELSE 0 END) as quantityOrderedIn_2004,
    sum(CASE WHEN YEAR(o.orderDate) = 2005 THEN 1 ELSE 0 END) as quantityOrderedIn_2005
    from 
    customers c 
      join orders o on c.customerNumber = o.customerNumber
    group by c.customerNumber, c.customerName
    having quantityOrderedIn_2005 = 0 
    order by c.customerNumber ;
    ```
    This query furnishes details about customers who were active purchasers until 2004 but did not place any orders further.
14. Do customers have outstanding balances?
    ``` SQL
    select c.customerNumber,c.customerName, c.creditLimit,
    sum(od.quantityOrdered*od.priceEach) as TotalPurchaseAmount,  totalPayments.totalAmount as TotalAmountPaid,
    sum(od.quantityOrdered*od.priceEach) - totalPayments.totalAmount  as DueAmount
    from customers c 
      join orders o on c.customerNumber = o.customerNumber
      join orderdetails od on o.orderNumber = od.orderNumber
      join ( select p.customerNumber, SUM(p.amount) AS totalAmount
	          from payments p
             group by p.customerNumber ) totalPayments ON c.customerNumber = totalPayments.customerNumber
    group by c.customerNumber, c.customerName, c.creditLimit
    having DueAmount > 0
    order by  DueAmount desc;
    ```
    This query offers information regarding customers with outstanding balance.
15. Are there customers with credit issues that need to be addressed?
    ``` SQL
    select c.customerNumber,c.customerName, c.creditLimit,
    sum(od.quantityOrdered*od.priceEach) as TotalPurchaseAmount,  totalPayments.totalAmount as TotalAmountPaid,
    c.creditLimit - (sum(od.quantityOrdered*od.priceEach) - totalPayments.totalAmount)  as CreditLimitCrossed
    from customers c 
      join orders o on c.customerNumber = o.customerNumber
      join orderdetails od on o.orderNumber = od.orderNumber
      join ( select p.customerNumber, SUM(p.amount) AS totalAmount
	          from payments p
             group by p.customerNumber) totalPayments ON c.customerNumber = totalPayments.customerNumber
    group by c.customerNumber, c.customerName, c.creditLimit
    having c.creditLimit - (sum(od.quantityOrdered*od.priceEach) - totalPayments.totalAmount) < 0
    order by  CreditLimitCrossed desc;
    ```
    This query provides insights into customers with outstanding balances who have surpassed their credit limits.
16. How are employees performing based on their overall sales?
    ``` SQL
    Select employeeNumber, fullName, jobTitle, coalesce(Sum( CASE WHEN status = "Shipped" or status = "Resolved" THEN priceEach * quantityOrdered ELSE 0 END),0) as TotalSalesAmount
    from (Select e.employeeNumber, concat(e.firstName, ' ' , e.lastName) as fullName, e.jobTitle, od.priceEach, od.quantityOrdered,o.status
	       from employees e 
	       left join customers c on e.employeeNumber = c.salesRepEmployeeNumber
	       left join orders o on c.customerNumber = o.customerNumber
	       left join orderdetails od on o.orderNumber = od.orderNumber
	       left join products p on  od.productCode = p.productCode
	       left join warehouses w on  p.warehouseCode = w.warehouseCode ) as totalSales
	 group by employeeNumber
	 order by jobTitle, TotalSalesAmount desc;
    ```
    This query helps to obtain the list of employees and their overall sales.
17. Are employees effectively able to secure deals with clients?
    ``` SQL
    Select employeeNumber, fullName, jobTitle, NumberOfDeals, NumberofSuccessfulDeals, CAST((NumberofSuccessfulDeals / NumberOfDeals)  * 100 as decimal(10,2)) as DealEfficiency
    from ( Select e.employeeNumber, concat(e.firstName, ' ' , e.lastName) as fullName, e.jobTitle, COUNT(*) as NumberOfDeals,
	       sum( CASE WHEN o.status = "Shipped" OR o.status = "Resolved" THEN 1 ElSE 0 END) as NumberofSuccessfulDeals
          from orders o 
          left join customers c on o.customerNumber = c.customerNumber
          left join employees e on c.salesRepEmployeeNumber = e.employeeNumber
          group by employeeNumber) as totalSales
    order by DealEfficiency desc;
    ```
    This query offers a glimpse into the success rate of employee deals, considering only orders that were either successfully shipped or encountered an issue that was subsequently resolved.
