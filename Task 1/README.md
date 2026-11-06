## ERD

![[ERD.png]]

## Schema

```sql
CREATE DATABASE ECOMMERCE;
USE ECOMMERCE;

CREATE TABLE Customer (
    CustomerID INT,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(250) NOT NULL,
    CustomerPassword VARCHAR(255) NOT NULL,
    PRIMARY KEY (CustomerID)
);

CREATE TABLE Category (
    CategoryID INT,
    CategoryName VARCHAR(50),
    PRIMARY KEY (CategoryID)
);

CREATE TABLE Product (
    ProductID INT,
    CategoryID INT,
    ProductName VARCHAR(50),
    ProductDescription TEXT,
    Price DECIMAL(10,2),
    StockQuantity INT,
    PRIMARY KEY (ProductID),
    FOREIGN KEY (CategoryID) REFERENCES Category(CategoryID)
);

CREATE TABLE Orders (
    OrderID INT,
    CustomerID INT,
    OrderDate DATE,
    TotalAmount DECIMAL(10,2),
    PRIMARY KEY (OrderID),
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID)
);

CREATE TABLE OrderDetails (
    OrderDetailID INT,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    PRIMARY KEY (OrderDetailID),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID)
);
```

## Reports

```sql
SELECT OrderDate AS RevenueDate, SUM(TotalAmount) AS TotalRevenue
FROM Orders
GROUP BY OrderDate;
```

```sql
SELECT p.ProductName, SUM(od.UnitPrice) AS TotalAmount
FROM Orders o JOIN OrderDetails od ON o.OrderID = od.OrderID
    JOIN Product p ON p.ProductID = od.ProductID
WHERE o.OrderDate BETWEEN '2023-08-01' AND '2023-08-31'
GROUP BY p.ProductName
ORDER BY TotalAmount DESC;
```

```sql
SELECT c.CustomerID, c.FirstName, c.LastName, SUM(o.TotalAmount) AS TotalOrders
FROM Customer c JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE o.OrderDate BETWEEN '2023-12-01' AND '2023-12-31'
GROUP BY c.CustomerID, c.FirstName, c.LastName
HAVING TotalOrders >= 500
ORDER BY TotalOrders DESC;
```

## Denormalization

Applying a denormalization mechanism on customer and order entities begins with identifying repeated queries that could potentially harm the database performance.

Such harmful queries to consider would be the generation of customers' total spending. To solve this problem it would be a good idea to add a total spending column to the customer table and add each new order total to that column. Of course that would take some time on the insertion of new orders, but the administrator would have to weigh the tradeoff between having this behavior added and whether it's actually worth the added load.
