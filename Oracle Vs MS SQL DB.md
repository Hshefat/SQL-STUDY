#Oracle vs Ms Sql
```sql
PL/SQL Stored Procedure (Oracle):

CREATE OR REPLACE PROCEDURE calculate_total_spent (
    p_customer_id IN INT,               -- Input parameter: Customer ID
    p_total_spent OUT NUMBER             -- Output parameter: Total amount spent by the customer
) AS
BEGIN
    -- Calculate the total spent by the customer
    SELECT SUM(total_amount)
    INTO p_total_spent
    FROM orders
    WHERE customer_id = p_customer_id;

    -- Handle the case when the customer has no orders
    IF p_total_spent IS NULL THEN
        p_total_spent := 0;
    END IF;

    -- Output the result (this is optional in PL/SQL procedures)
    DBMS_OUTPUT.PUT_LINE('Total amount spent by customer ' || p_customer_id || ': ' || p_total_spent);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        -- If no data is found, set total to 0
        p_total_spent := 0;
    WHEN OTHERS THEN
        -- Handle any other errors
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
        p_total_spent := 0;
END calculate_total_spent;
/

How to Call the Procedure (Oracle):

DECLARE
    total_spent NUMBER(10,2);
BEGIN
    -- Calling the stored procedure for customer ID 101
    calculate_total_spent(101, total_spent);
    -- Display the result
    DBMS_OUTPUT.PUT_LINE('Total Spent: ' || total_spent);
END;
/

2. MS SQL Server T-SQL Stored Procedure

In MS SQL Server, we will create a T-SQL procedure that performs the same task as the Oracle procedure—calculating the total amount spent by a customer.
MS SQL Server Table Structure (Example):

-- MS SQL Server Table: Customers
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100)
);

-- MS SQL Server Table: Orders
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

T-SQL Stored Procedure (MS SQL Server):

CREATE PROCEDURE calculate_total_spent
    @customer_id INT,              -- Input parameter: Customer ID
    @total_spent DECIMAL(10, 2) OUTPUT  -- Output parameter: Total amount spent by the customer
AS
BEGIN
    -- Calculate the total spent by the customer
    SELECT @total_spent = SUM(total_amount)
    FROM orders
    WHERE customer_id = @customer_id;

    -- Handle the case when the customer has no orders (NULL case)
    IF @total_spent IS NULL
    BEGIN
        SET @total_spent = 0;
    END

    -- Output the result (for debugging or information purposes)
    PRINT 'Total amount spent by customer ' + CAST(@customer_id AS VARCHAR) + ': ' + CAST(@total_spent AS VARCHAR);
END;
GO

How to Call the Procedure (MS SQL Server):

DECLARE @total_spent DECIMAL(10,2);
-- Calling the stored procedure for customer ID 101
EXEC calculate_total_spent @customer_id = 101, @total_spent = @total_spent OUTPUT;
-- Display the result
PRINT 'Total Spent: ' + CAST(@total_spent AS VARCHAR);
