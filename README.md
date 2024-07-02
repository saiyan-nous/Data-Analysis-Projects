# Maryland-City-Project
Maryland City has been purchasing lots of bicycle products and parts across the world for the citizens' usage. Each product is classified and tracked based on order date, delivery due date and shipment, as these are essential to provide the citizens the procurement information.

The purchasing information is stored in Category, Product, ProductCategory, and Purchase Trans.

## Queries and Results

### Question1: 

Produce a SQL Statement that joins all the tables to get the product name, product number and PurchaseTrans

    SELECT PT.TransID, PT.OrderID, CONVERT(Date,PT.OrderDate) OrderDate, CONVERT(date,PT.ShipDate) ShipDate, CONVERT(date,PT.DueDate) DueDate,
		PT.Supplier, PT.Country,
		PT.StateProvince, PT.City, PT.PostalCode, PT.CarrierTrackingNumber, P.ProductName, P.ProductNumber,PT.Class, PT.Color, PT.Employee,
		PT.OrderQty, PT.UnitPrice, PT.UnitPriceDiscount
        FROM PurchaseTrans PT
        INNER JOIN Product P ON P.ProductID = PT.ProductID

![Question1 Union Query](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/21aff9fc-f6e0-403b-a8b7-6088cbbb705e)


### Question2: 
Top 10 products ordered in each country on specific date with product detail including product category and net amount

    WITH CTE AS (
				SELECT ROW_NUMBER() OVER(partition by Country order by AGGNetPurchaseAmount desc) RankID, a.OrderDate, a.Country, a.ProductName,
				a.CategoryName, FORMAT(isnull(a.AGGNetPurchaseAmount,0),'C') AGGNetPurchaseAmount
				FROM (
						SELECT CONVERT(Date,PT.OrderDate) OrderDate, PT.Country, P.ProductName, C.CategoryName,
										SUM((PT.UnitPrice*PT.OrderQty)-((PT.UnitPrice*PT.OrderQty)*PT.UnitPriceDiscount)) AGGNetPurchaseAmount
						FROM PurchaseTrans PT
						INNER JOIN Product P ON P.ProductID = PT.ProductID
						INNER JOIN ProductCategory PC ON PC.ProductID = P.ProductID
						INNER JOIN Category C ON C.CategoryID = PC.CategoryID
						GROUP BY PT.OrderDate, PT.Country, P.ProductName, C.CategoryName
					) a
			)
            
            SELECT * FROM CTE
            WHERE RankID <= 10

![Q2 Top10 each country](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/a4a7edd9-ca4e-4f18-a235-34367fe673a0)


### Question3: 
List of suppliers that offer discount amount on purchased products filtered by Product Category

    SELECT PT.Supplier, C.CategoryName
    FROM PurchaseTrans PT
    INNER JOIN Product P ON P.ProductID = PT.ProductID
    INNER JOIN ProductCategory PC ON PC.ProductID = P.ProductID
    INNER JOIN Category C ON C.CategoryID = PC.CategoryID
    WHERE UnitPriceDiscount > 0
    GROUP BY PT.Supplier, C.CategoryName

![Q3 Supplier List](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/ddcceda1-e936-4298-a8e1-70f505f427d1)


### Question4:
Create KPI with Low, Medium, High indicator based on Net Purchase Amount with the following
conditions:
- If the Net Purchase Amount is less than $10,000 then “Low”
- If the Net Purchase Amount between $10,000 and $20,000 then “Medium”
- If the Net Purchase Amount greater than $20,000 then “HIGH”

        SELECT PT.Country, C.CategoryName, P.ProductName, 
		FORMAT(isnull(SUM((PT.UnitPrice*PT.OrderQty)-((PT.UnitPrice*PT.OrderQty)*PT.UnitPriceDiscount)),0),'C') AGGNetPurchaseAmount,
		CASE
			WHEN SUM((PT.UnitPrice*PT.OrderQty)-((PT.UnitPrice*PT.OrderQty)*PT.UnitPriceDiscount)) < 10000 THEN 'Low'
			WHEN SUM((PT.UnitPrice*PT.OrderQty)-((PT.UnitPrice*PT.OrderQty)*PT.UnitPriceDiscount)) between 10000 and 20000 THEN 'Medium'
			WHEN SUM((PT.UnitPrice*PT.OrderQty)-((PT.UnitPrice*PT.OrderQty)*PT.UnitPriceDiscount)) > 20000 THEN 'High'

			END as KPI
            FROM PurchaseTrans PT
            INNER JOIN Product P ON P.ProductID = PT.ProductID
            INNER JOIN ProductCategory PC ON PC.ProductID = P.ProductID
            INNER JOIN Category C ON C.CategoryID = PC.CategoryID
            GROUP BY PT.Country, C.CategoryName, P.ProductName

![Q4 KPI Indicator](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/ea15f734-4d76-4602-bf53-fd7a738c10ff)

## Visualization

![Purchase Analysis Visualization](https://github.com/saiyan-nous/Data-Analysis-Projects/assets/105250935/ed51d38f-6ad8-4ed5-887c-3942d9a40deb)
