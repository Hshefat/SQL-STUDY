
#stored Procedure
```sql
Step 1: Tables for Shop Management System

Let's first define the tables. The core tables we'll need for this example are:

    Products Table - To store information about products.
    Sales Table - To track sales transactions.
    Transactions Table - To track any other transaction for earnings, losses, etc.

1. Products Table
 
CREATE TABLE Products (
    ProductID INT PRIMARY KEY IDENTITY(1,1),
    ProductName NVARCHAR(100) NOT NULL,
    Category NVARCHAR(50),
    Price DECIMAL(18, 2) NOT NULL,
    StockQuantity INT NOT NULL,
    TotalEarned DECIMAL(18, 2) DEFAULT 0.00,
    TotalCost DECIMAL(18, 2) DEFAULT 0.00,
    Profit DECIMAL(18, 2) DEFAULT 0.00
);
 
 2. Sales Table

CREATE TABLE Sales (
    SaleID INT PRIMARY KEY IDENTITY(1,1),
    ProductID INT,
    Quantity INT NOT NULL,
    SaleDate DATETIME DEFAULT GETDATE(),
    SalePrice DECIMAL(18, 2) NOT NULL,
    TotalAmount DECIMAL(18, 2) AS (Quantity * SalePrice) PERSISTED,
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);

3. Transactions Table

CREATE TABLE Transactions (
    TransactionID INT PRIMARY KEY IDENTITY(1,1),
    TransactionType NVARCHAR(50),  -- 'Earning', 'Loss', 'Cost', etc.
    Amount DECIMAL(18, 2),
    TransactionDate DATETIME DEFAULT GETDATE(),
    Description NVARCHAR(255)
);


```
#Step 2: Stored Procedures
```sql


Now let's create stored procedures for the operations you requested:
1. Insert Product (Create)

CREATE PROCEDURE InsertProduct
    @ProductName NVARCHAR(100),
    @Category NVARCHAR(50),
    @Price DECIMAL(18, 2),
    @StockQuantity INT
AS
BEGIN
    INSERT INTO Products (ProductName, Category, Price, StockQuantity)
    VALUES (@ProductName, @Category, @Price, @StockQuantity);
END

2. Update Product Stock (Update)

This procedure will increment or decrement the stock quantity when a sale or new product is added:

CREATE PROCEDURE UpdateProductStock
    @ProductID INT,
    @Quantity INT,  -- Positive for new stock, negative for sale
    @TransactionType NVARCHAR(50) -- "Sale" or "Restock"
AS
BEGIN
    DECLARE @CurrentStock INT;
    DECLARE @ProductPrice DECIMAL(18,2);

    -- Get current stock and product price
    SELECT @CurrentStock = StockQuantity, @ProductPrice = Price
    FROM Products WHERE ProductID = @ProductID;

    -- Update stock based on transaction type
    IF @TransactionType = 'Sale'
    BEGIN
        IF @CurrentStock < @Quantity
        BEGIN
            PRINT 'Not enough stock!';
            RETURN;
        END
        UPDATE Products
        SET StockQuantity = StockQuantity - @Quantity
        WHERE ProductID = @ProductID;

        -- Insert sale transaction
        INSERT INTO Sales (ProductID, Quantity, SalePrice)
        VALUES (@ProductID, @Quantity, @ProductPrice);

        -- Update earnings
        UPDATE Products
        SET TotalEarned = TotalEarned + (@Quantity * @ProductPrice),
            Profit = Profit + (@Quantity * (@ProductPrice - @ProductPrice)) -- assuming cost is same as price
        WHERE ProductID = @ProductID;

        -- Insert profit/loss transaction
        INSERT INTO Transactions (TransactionType, Amount, Description)
        VALUES ('Earning', @Quantity * @ProductPrice, 'Sale of Product ' + CAST(@ProductID AS NVARCHAR));

    END
    ELSE IF @TransactionType = 'Restock'
    BEGIN
        UPDATE Products
        SET StockQuantity = StockQuantity + @Quantity
        WHERE ProductID = @ProductID;
    END
END

3. Delete Product (Delete)

CREATE PROCEDURE DeleteProduct
    @ProductID INT
AS
BEGIN
    DELETE FROM Products WHERE ProductID = @ProductID;
END

4. Get Product Details (Read)

CREATE PROCEDURE GetProductDetails
    @ProductID INT
AS
BEGIN
    SELECT * FROM Products WHERE ProductID = @ProductID;
END

5. Get Sales Summary

This stored procedure will provide a summary of earnings, loss, and profit:

CREATE PROCEDURE GetSalesSummary
AS
BEGIN
    SELECT 
        SUM(TotalAmount) AS TotalEarnings,
        SUM(Profit) AS TotalProfit,
        (SUM(TotalAmount) - SUM(Profit)) AS TotalLoss
    FROM Sales;
END



```
 ##3: Example of Using the Stored Procedures
```sql



 

Here’s how you can use the stored procedures:

    To add a new product:

EXEC InsertProduct 'Laptop', 'Electronics', 1000.00, 50;

To restock a product:

EXEC UpdateProductStock @ProductID = 1, @Quantity = 20, @TransactionType = 'Restock';

To make a sale:

EXEC UpdateProductStock @ProductID = 1, @Quantity = 5, @TransactionType = 'Sale';

To delete a product:

EXEC DeleteProduct @ProductID = 1;

To get the product details:

EXEC GetProductDetails @ProductID = 1;

To get a sales summary:

EXEC GetSalesSummary;
