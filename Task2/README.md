# Denormalization

```sql
CREATE TABLE CategoryHierarchy (
	ParentCategoryID INT,
	CategoryName VARCHAR(50),
	ChildCategoryID INT,
	
	FOREIGN KEY (ParentCategoryID) REFERENCES Category(CategoryID),
	FOREIGN KEY (ChildCategoryID) REFERENCES Category(CategoryID),
	PRIMARY KEY (ParentCategoryID, ChildCategoryID)
);

INSERT INTO CategoryHierarchy (ParentCategoryID, CategoryName, ChildCategoryID)
	SELECT
	    C1.CategoryID AS ParentCategoryID,
	    C1.CategoryName AS CategoryName,
	    C1.CategoryID as ChildCategoryID
	FROM Category C1
	WHERE C1.ParentCategoryID IS NULL
	
	UNION ALL
	
	SELECT
	    C2.ParentCategoryID,
	    C1.CategoryName || ' > ' || C2.CategoryName,
	    C2.CategoryID as ChildCategoryID
	FROM Category C1
		INNER JOIN Category C2 ON C1.CategoryID = C2.ParentCategoryID
	WHERE C1.ParentCategoryID IS NULL
	
	UNION ALL
	
	SELECT
	    C3.ParentCategoryID,
	    C1.CategoryName || ' > ' || C2.CategoryName || ' > ' || C3.CategoryName,
		C3.CategoryID as ChildCategoryID
	FROM Category C1
		INNER JOIN Category C2 ON C1.CategoryID = C2.ParentCategoryID
		INNER JOIN Category C3 ON C2.CategoryID = C3.ParentCategoryID
	WHERE C1.ParentCategoryID IS NULL
	ORDER BY ParentCategoryID, ChildCategoryID , CategoryName;
```

# Reports

- Write a SQL query to search for all products with the word "camera" in either the product name or description.

```sql
SELECT ProductID, ProductName, ProductDescription, Price, StockQuantity
FROM Product
WHERE ProductName LIKE '%camera%'
   OR ProductDescription LIKE '%camera%';
```

-  Can you design a query to suggest popular products in the same category for the same author, excluding the Purchased product from the recommendations?
(Assuming CustomerID = 8 and ProductID = 20)

```sql
SELECT C.CategoryID, C.CategoryName, P.ProductID, P.ProductName, SUM(OD.Quantity) AS SoldQuantity
FROM Orders O 
    JOIN OrderDetails OD ON O.OrderID = OD.OrderID
    JOIN Product P ON P.ProductID = OD.ProductID
    JOIN Category C ON C.CategoryID = P.CategoryID
WHERE C.CategoryID = (SELECT CategoryID FROM Product WHERE ProductID = 20)
    AND P.ProductID <> 20
    AND P.ProductID NOT IN (
        SELECT OD.ProductID
        FROM Orders O JOIN OrderDetails OD ON O.OrderID = OD.OrderID
        WHERE O.CustomerID = 8
    )
GROUP BY C.CategoryID, C.CategoryName, P.ProductID, P.ProductName
ORDER BY SoldQuantity DESC;
```