## Step 1: Create Table schema as per requirement
```csharp
CREATE TABLE [dbo].[Bills](
	[BillID] [int] IDENTITY(1,1) NOT NULL,
	[OrderID] [int] NOT NULL,
	[BillDate] [datetime] NULL DEFAULT (getdate()),
	[TotalAmount] [decimal](10, 2) NOT NULL,
PRIMARY KEY CLUSTERED 
(
	[BillID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

CREATE TABLE [dbo].[Customers](
	[CustomerID] [int] IDENTITY(1,1) NOT NULL,
	[CustomerName] [nvarchar](255) NOT NULL,
	[Email] [nvarchar](255) NOT NULL,
	[Phone] [nvarchar](20) NULL,
	[Address] [nvarchar](500) NULL,
	[CreatedAt] [datetime] NULL DEFAULT (getdate()),
PRIMARY KEY CLUSTERED 
(
	[CustomerID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

CREATE TABLE [dbo].[OrderItems](
	[OrderItemID] [int] IDENTITY(1,1) NOT NULL,
	[OrderID] [int] NOT NULL,
	[ProductID] [int] NOT NULL,
	[Quantity] [int] NOT NULL,
	[Price] [decimal](10, 2) NOT NULL,
PRIMARY KEY CLUSTERED 
(
	[OrderItemID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

CREATE TABLE [dbo].[Orders](
	[OrderID] [int] IDENTITY(1,1) NOT NULL,
	[CustomerID] [int] NOT NULL,
	[OrderDate] [datetime] NULL DEFAULT (getdate()),
	[Status] [nvarchar](50) NULL DEFAULT ('Pending'),
	[TotalAmount] [decimal](10, 2) NULL,
PRIMARY KEY CLUSTERED 
(
	[OrderID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

CREATE TABLE [dbo].[Products](
	[ProductID] [int] IDENTITY(1,1) NOT NULL,
	[ProductName] [nvarchar](255) NOT NULL,
	[Category] [nvarchar](255) NULL,
	[Price] [decimal](10, 2) NOT NULL,
	[StockQuantity] [int] NOT NULL,
	[AddedDate] [datetime] NULL DEFAULT (getdate()),
PRIMARY KEY CLUSTERED 
(
	[ProductID] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]
```
## Step 2: Create Stored Procedure for Dashboard as follows
```sql
ALTER PROCEDURE [dbo].[usp_GetDashboardData]
AS
BEGIN
    -- Enable NOCOUNT for better performance
    SET NOCOUNT ON;
-- SET NOCOUNT ON: Suppresses the message from being returned. This prevents the sending of DONEINPROC messages to the client for each
-- statement in a stored procedure.
-- SET NOCOUNT OFF: Includes the message in the result set. 
    -- Temporary tables for organized data fetching
	CREATE TABLE #Counts (
        Metric NVARCHAR(255),
        Value INT
		);

    CREATE TABLE #RecentOrders (
        OrderID INT,
        CustomerName NVARCHAR(255),
        OrderDate DATETIME,
        Status NVARCHAR(50)
    );

    CREATE TABLE #RecentProducts (
        ProductID INT,
        ProductName NVARCHAR(255),
        Category NVARCHAR(255),
        AddedDate DATETIME,
        StockQuantity INT
    );

    CREATE TABLE #TopCustomers (
        CustomerName NVARCHAR(255),
        TotalOrders INT,
        Email NVARCHAR(255)
    );

    CREATE TABLE #TopSellingProducts (
        ProductName NVARCHAR(255),
        TotalSoldQuantity INT,
        Category NVARCHAR(255)
    );

    ---- Step 1: Get Counts
    --
	INSERT INTO #Counts
        SELECT 'TotalCustomers', COUNT(*) FROM Customers
    INSERT INTO #Counts
	    SELECT 'TotalProducts', COUNT(*) FROM Products
	INSERT INTO #Counts
		SELECT 'TotalOrders',COUNT(*) FROM Orders
	INSERT INTO #Counts
		SELECT 'TotalBills',COUNT(*) FROM Bills
		
    --    (SELECT COUNT(*) FROM Customers) AS TotalCustomers,
    --    (SELECT COUNT(*) FROM Products) AS TotalProducts,
    --    (SELECT COUNT(*) FROM Orders) AS TotalOrders,
    --    (SELECT COUNT(*) FROM Bills) AS TotalBills;

    -- Step 2: Get Recent 10 Orders
    INSERT INTO #RecentOrders
    SELECT TOP 10
        O.OrderID,
        C.CustomerName,
        O.OrderDate,
        O.Status
    FROM Orders O
    INNER JOIN Customers C ON O.CustomerID = C.CustomerID
    ORDER BY O.OrderDate DESC;

    -- Step 3: Get Recent 10 Newly Added Products
    INSERT INTO #RecentProducts
    SELECT TOP 10
        ProductID,
        ProductName,
        Category,
        AddedDate,
        StockQuantity
    FROM Products
    ORDER BY AddedDate DESC;

    -- Step 4: Get Top 10 Customers by Order Count
    INSERT INTO #TopCustomers
    SELECT TOP 10
        C.CustomerName,
        COUNT(O.OrderID) AS TotalOrders,
        C.Email
    FROM Orders O
    INNER JOIN Customers C ON O.CustomerID = C.CustomerID
    GROUP BY C.CustomerName, C.Email
    ORDER BY COUNT(O.OrderID) DESC;

    -- Step 5: Get Top 10 Selling Products
    INSERT INTO #TopSellingProducts
    SELECT TOP 10
        P.ProductName,
        SUM(OI.Quantity) AS TotalSoldQuantity,
        P.Category
    FROM OrderItems OI
    INNER JOIN Products P ON OI.ProductID = P.ProductID
    GROUP BY P.ProductName, P.Category
    ORDER BY SUM(OI.Quantity) DESC;

    -- Output Results
    -- Output Counts
    SELECT * FROM #Counts;

    -- Output Recent Orders
    SELECT * FROM #RecentOrders;

    -- Output Recent Products
    SELECT * FROM #RecentProducts;

    -- Output Top Customers
    SELECT * FROM #TopCustomers;

    -- Output Top Selling Products
    SELECT * FROM #TopSellingProducts;

    -- Cleanup Temporary Tables
    DROP TABLE #RecentOrders;
    DROP TABLE #RecentProducts;
    DROP TABLE #TopCustomers;
    DROP TABLE #TopSellingProducts;
END;
```
