# Joins--SQL


Joins- SQL
-----Create 2 tables:
------The Stg_Emp table contains all new & current employees

CREATE TABLE	StgEmp (EmpID INT, EmpName VARCHAR(50))

INSERT INTO		StgEmp (EmpID,EmpName)

VALUES	       (1,'John Doe'),
		       (2,'Jane Doe'),
		       (3,'Sally Mae')
--The Emp_List table contains some current & all former employees

CREATE TABLE	EmpList( EmpID INT, EmpName VARCHAR (50))

INSERT INTO		EmpList ( EmpID,EmpName)
VALUES			(1,'John Doe'  ),
				(2,'Jane Doe'), 
				(5,'Peggy Sue')
--ASSIGNMENT: Write two scripts using JOIN to:
/*Show the Company’s new employee = Only Show Sally Mae
Show the Company’s former employee – Only Show Peggy Sue
*/

--I created a FULL JOIN to combine ALL Employees- New, Current and Former.

--Script 1
SELECT StgEmp.EmpName AS NewEmployee ,EmpList.EmpName AS FormerEmployee
FROM StgEmp
FULL JOIN EmpList
ON StgEmp.EmpID=EmpList.EmpID
-- In a Full Join, all columns combines but where there is no matching data, it appears'IS NULL'. Isolating the NULLS will show the Unique data i.e New and Former Employees Only.

--Script 2 - Isolate the Unique rows which will highlight the New and Former Employee

SELECT    StgEmp.EmpName AS NewEmployee ,EmpList.EmpName AS FormerEmployee
FROM	  StgEmp
FULL JOIN EmpList 
ON		   StgEmp.EmpID=EmpList.EmpID
WHERE	  StgEmp.EmpName IS NULL OR EmpList.EmpName IS NULL
/* LAB TWO : ADVENTUREWORKS

1.How many Sales Orders (Headers) used Vista credit cards in June 2012
*/
SELECT *
FROM [AdventureWorks2019].[Sales].[SalesOrderHeader]

SELECT	         a.OrderDate, b.CardType,COUNT (a.SalesOrderID) AS NoOfOrders
FROM	         [AdventureWorks2019].[Sales].[SalesOrderHeader] a
LEFT OUTER JOIN  [AdventureWorks2019].[Sales].[CreditCard] b
ON	             a.CreditCardID= b.CreditCardID
WHERE	         b.CardType ='Vista' AND a.OrderDate BETWEEN '2012-06-01' AND '2012-06-30'
GROUP BY         b.CardType,a.OrderDate,a.SalesOrderID
ORDER BY		 a.OrderDate 
-- ANSWER IS 98

--2. Store the answer to Q1. in a variable.

DECLARE @NumOfVistaSales INT

SET		@NumOfVistaSales = (
							SELECT	         a.OrderDate, b.CardType,COUNT (a.SalesOrderID) AS NoOfOrders
	                        FROM	         [AdventureWorks2019].[Sales].[SalesOrderHeader] a
	                        LEFT OUTER JOIN  [AdventureWorks2019].[Sales].[CreditCard] b
	                        ON	             a.CreditCardID= b.CreditCardID
	                        WHERE	         b.CardType ='Vista'  AND a.OrderDate BETWEEN'@StartDate' AND '@EndDate'
	                        GROUP BY         b.CardType,a.OrderDate,a.SalesOrderID

							)

SELECT	@NumOfVistaSales AS NumOfVistaSales

GO
--3. Create a UDF that accepts start date and end date. The function will return
--the number of Sales Orders (Using Vista credit cards) that took place between
--the start date and end date entered by the user.

CREATE FUNCTION fx_getVistaCCSales

				(@StartDate Date, @EndDate Date)

RETURNS INT

AS

BEGIN

   DECLARE @NoOfVistaSales INT =
                    ( 
                    SELECT	         a.OrderDate, b.CardType,COUNT (a.SalesOrderID) AS NoOfOrders
                    FROM	         [AdventureWorks2019].[Sales].[SalesOrderHeader] a
                    LEFT OUTER JOIN  [AdventureWorks2019].[Sales].[CreditCard] b
                    ON	             a.CreditCardID= b.CreditCardID
                    WHERE           b.CardType ='Vista'  AND a.OrderDate BETWEEN '@StartDate' AND '@EndDate'
                    GROUP BY         b.CardType,a.OrderDate,a.SalesOrderID

			         )
   RETURN @NoOfVistaSales

END
--4. Using the SalesOrderHeader table - Find out how much Revenue (TotalDue) was brought in by the North American Territory Group from 2012 through 2014
--I altered the date range because this version of AdventureWorks2019 has no data pior to 2010.

 SELECT  SUM (TotalDue) AS NorthAmericaRevenue
 FROM    [AdventureWorks2019].[Sales].[SalesOrderHeader] 
 WHERE   TerritoryID BETWEEN 1 and 6
	      AND OrderDate BETWEEN '2012-01-01' AND '2014-12-31'

		  --OR

 SELECT  SUM (TotalDue) AS NorthAmericaRevenue
 FROM    [AdventureWorks2019].[Sales].[SalesOrderHeader] 
 WHERE   TerritoryID BETWEEN 1 and 6
	      AND OrderDate BETWEEN '2002-01-01' AND '2004-12-31'
--5. What is the Sales Tax Rate, StateProvinceCode and CountryRegionCode for Texas?

SELECT		a.TaxRate AS SalesTaxRate,b.StateProvinceCode,c.CountryRegionCode
FROM		Sales.SalesTaxRate a
LEFT JOIN	Person.StateProvince b
ON		    a.StateProvinceID = b.StateProvinceID
LEFT JOIN   Sales.SalesTerritory c
ON			b.CountryRegionCode = c.CountryRegionCode
WHERE		a.Name LIKE '%Texas%' 
--6. Store the information from Q5 in a variable

DECLARE @texas TABLE (TaxRate Money,StateProvinceCode NVARCHAR (10),CountryRegionCode NVARCHAR (10))

INSERT INTO @texas

SELECT a.TaxRate AS SalesTaxRate,b.StateProvinceCode,c.CountryRegionCode
FROM Sales.SalesTaxRate a
LEFT JOIN Person.StateProvince b
ON a.StateProvinceID = b.StateProvinceID
LEFT JOIN Sales.SalesTerritory c
ON b.CountryRegionCode = c.CountryRegionCode
WHERE a.Name LIKE '%Texas%'

SELECT *
FROM @texas

--7. Create a UDF that accepts the State Province and returns the associated.
--Sales Tax Rate, StateProvinceCode and CountryRegionCode.

--HardCode Component
SELECT a.TaxRate AS SalesTaxRate,b.StateProvinceCode,c.CountryRegionCode
FROM Sales.SalesTaxRate a
LEFT JOIN Person.StateProvince b
ON a.StateProvinceID = b.StateProvinceID
LEFT JOIN Sales.SalesTerritory c
ON b.CountryRegionCode = c.CountryRegionCode
WHERE a.Name = @StateProvince

CREATE FUNCTION  fx_getSalesTaxRateProvinceCodeCountryRegionCode

				(@StateProvince VARCHAR(50))

RETURNS TABLE

AS

	  RETURN                         
			  ( 
			  SELECT		a.TaxRate AS SalesTaxRate,b.StateProvinceCode,c.CountryRegionCode
              FROM			Sales.SalesTaxRate a
              LEFT JOIN		Person.StateProvince b
              ON		    a.StateProvinceID = b.StateProvinceID
              LEFT JOIN		Sales.SalesTerritory c
              ON			b.CountryRegionCode = c.CountryRegionCode
              WHERE			a.Name = @StateProvince
				)
END
--8. Show a list of Product Colors. For each Color show and the Total SalesAmount (UnitPrice * OrderQty). Only show Colors with a Total SalesAmount more than $50,000 and eliminate the products that do not have a color.

SELECT 		a.Color, COUNT (b. SalesOrderDetailID) AS Sales,( b.UnitPrice* b.OrderQty) AS TotalSalesAmt
FROM		Production.Product a
LEFT JOIN	Sales.SalesOrderDetail b
ON		    a.ProductID = b.ProductID
WHERE	    a.Color IS NOT NULL
GROUP  BY   a.Color,b.SalesOrderDetailID,b.UnitPrice,b.OrderQty
--9. Create a join using 4 tables in AdventureWorks database.
-- I joined the 4 Tables to show the Employees Dept,StartDate, Shift and their Job Titles.

SELECT	     c.Name AS DeptName,a.StartDate  ,d.Name AS ShiftName,b.JobTitle
FROM		 HumanResources.EmployeeDepartmentHistory a
LEFT JOIN    HumanResources.Employee b
ON           a.BusinessEntityID =b.BusinessEntityID
LEFT JOIN    HumanResources.Department c
ON           a.DepartmentID= c.DepartmentID
LEFT JOIN    HumanResources.Shift  d
ON           a.ShiftID = d.ShiftID
