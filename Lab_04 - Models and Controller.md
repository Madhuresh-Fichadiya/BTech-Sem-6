# Controllers

```csharp
public async Task<IActionResult> Index()
{
    var dashboardData = new Dashboard
    {
        Counts = new List<DashboardCounts>(),
        RecentOrders = new List<RecentOrder>(),
        RecentProducts = new List<RecentProduct>(),
        TopCustomers = new List<TopCustomer>(),
        TopSellingProducts = new List<TopSellingProduct>(),
        NavigationLinks = new List<QuickLinks>()
    };

    using (var connection = new SqlConnection(this._configuration.GetConnectionString("myConnectionString")))
    {
        await connection.OpenAsync();

        using (var command = new SqlCommand("usp_GetDashboardData", connection))
        {
            command.CommandType = CommandType.StoredProcedure;

            using (var reader = await command.ExecuteReaderAsync())
            {
                if (reader.HasRows)
                {
                    // Fetch counts
                    while (await reader.ReadAsync())
                    {
                        dashboardData.Counts.Add(new DashboardCounts
                        {
                            Metric = reader["Metric"].ToString(),
                            Value = Convert.ToInt32(reader["Value"])
                        });
                    }

                    // Fetch recent orders
                    if (await reader.NextResultAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            dashboardData.RecentOrders.Add(new RecentOrder
                            {
                                OrderID = Convert.ToInt32(reader["OrderID"]),
                                CustomerName = reader["CustomerName"].ToString(),
                                OrderDate = Convert.ToDateTime(reader["OrderDate"]),
                                Status = reader["Status"].ToString()
                            });
                        }
                    }

                    // Fetch recent products
                    if (await reader.NextResultAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            dashboardData.RecentProducts.Add(new RecentProduct
                            {
                                ProductID = Convert.ToInt32(reader["ProductID"]),
                                ProductName = reader["ProductName"].ToString(),
                                Category = reader["Category"].ToString(),
                                AddedDate = Convert.ToDateTime(reader["AddedDate"]),
                                StockQuantity = Convert.ToInt32(reader["StockQuantity"])
                            });
                        }
                    }

                    // Fetch top customers
                    if (await reader.NextResultAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            dashboardData.TopCustomers.Add(new TopCustomer
                            {
                                CustomerName = reader["CustomerName"].ToString(),
                                TotalOrders = Convert.ToInt32(reader["TotalOrders"]),
                                Email = reader["Email"].ToString()
                            });
                        }
                    }

                    // Fetch top selling products
                    if (await reader.NextResultAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            dashboardData.TopSellingProducts.Add(new TopSellingProduct
                            {
                                ProductName = reader["ProductName"].ToString(),
                                TotalSoldQuantity = Convert.ToInt32(reader["TotalSoldQuantity"]),
                                Category = reader["Category"].ToString()
                            });
                        }
                    }
                }
            }
        }
    }

    dashboardData.NavigationLinks = new List<QuickLinks> { 
        new QuickLinks {ActionMethodName = "Index", ControllerName="HomeMaster", LinkName="Dashboard" },
        new QuickLinks {ActionMethodName = "Privacy", ControllerName="HomeMaster", LinkName="Privacy" },
        new QuickLinks {ActionMethodName = "Index", ControllerName="Country", LinkName="Country" },
        new QuickLinks {ActionMethodName = "Index", ControllerName="State", LinkName="State" },
        new QuickLinks {ActionMethodName = "Index", ControllerName="City", LinkName="City" }
    };

    var model = new Dashboard
    {
        Counts = dashboardData.Counts,
        RecentOrders = dashboardData.RecentOrders,
        RecentProducts = dashboardData.RecentProducts,
        TopCustomers = dashboardData.TopCustomers,
        TopSellingProducts = dashboardData.TopSellingProducts,
        NavigationLinks = dashboardData.NavigationLinks
    };

    return View("Dashboard", model);
}
```

# Models
```csharp
public class DashboardCounts
{
    public string Metric { get; set; }
    public int Value { get; set; }
}

public class RecentOrder
{
    public int OrderID { get; set; }
    public string CustomerName { get; set; }
    public DateTime OrderDate { get; set; }
    public string Status { get; set; }
}

public class RecentProduct
{
    public int ProductID { get; set; }
    public string ProductName { get; set; }
    public string Category { get; set; }
    public DateTime AddedDate { get; set; }
    public int StockQuantity { get; set; }
}

public class TopCustomer
{
    public string CustomerName { get; set; }
    public int TotalOrders { get; set; }
    public string Email { get; set; }
}

public class TopSellingProduct
{
    public string ProductName { get; set; }
    public int TotalSoldQuantity { get; set; }
    public string Category { get; set; }
}

public class QuickLinks
{ 
    public string ActionMethodName { get; set; }
    public string ControllerName { get; set; }
    public string LinkName { get; set; }
}

public class Dashboard
{
    public List<DashboardCounts> Counts { get; set; }
    public List<RecentOrder> RecentOrders { get; set; }
    public List<RecentProduct> RecentProducts { get; set; }
    public List<TopCustomer> TopCustomers { get; set; }
    public List<TopSellingProduct> TopSellingProducts { get; set; }
    public List<QuickLinks> NavigationLinks { get; set; }
}
```
