# Dashboard basic design
```csharp
@model Dashboard

@{
    ViewData["Title"] = "Dashboard";
}

@section Head{
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet" />
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js"></script>

}

<div class="container mt-5">
    <h1 class="text-center mb-4">Dashboard</h1>

    <!-- Counts Section -->
    <div class="row mb-4">
        <!-- Recent Orders Section -->
        <div class="col-6">
            <div class="card mb-4">
                <div class="card-header">
                    <h4>Quick Links</h4>
                </div>
                <div class="card-body">
                    <table class="table table-striped">
                        <thead>
                            <tr>
                                <th>Navigate to</th>
                            </tr>
                        </thead>
                        <tbody>
                            @foreach (var link in Model.NavigationLinks)
                            {
                                <tr>
                                    <td><a asp-action="@link.ActionMethodName" asp-controller="@link.ControllerName">@link.LinkName</a></td>
                                </tr>
                            }
                        </tbody>
                    </table>
                </div>
            </div>

        </div>
        <div class="col-6">
            <div class="row">
                <div class="card col-5 m-2">
                    <div class="card-body">
                        <h5 class="card-title">Total Customers</h5><hr />
                        <p class="card-text">@Model.Counts.FirstOrDefault(c => c.Metric == "TotalCustomers")?.Value</p>
                    </div>
                </div>

                <div class="card col-5 m-2">
                    <div class="card-body">
                        <h5 class="card-title">Total Products<hr /></h5>
                        <p class="card-text">@Model.Counts.FirstOrDefault(c => c.Metric == "TotalProducts")?.Value</p>
                    </div>
                </div>

                <div class="card col-5 m-2">
                    <div class="card-body">
                        <h5 class="card-title">Total Orders</h5><hr />
                        <p class="card-text">@Model.Counts.FirstOrDefault(c => c.Metric == "TotalOrders")?.Value</p>
                    </div>
                </div>

                <div class="card col-5 m-2">
                    <div class="card-body">
                        <h5 class="card-title">Total Bills</h5><hr />
                        <p class="card-text">@Model.Counts.FirstOrDefault(c => c.Metric == "TotalBills")?.Value</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<div class="row mb-4">
    <!-- Recent Orders Section -->
    <div class="container col-6">
        <div class="card mb-4">
            <div class="card-header">
                <h4>Recent 10 Orders</h4>
            </div>
            <div class="card-body">
                <table class="table table-striped">
                    <thead>
                        <tr>
                            <th>Order ID</th>
                            <th>Customer</th>
                            <th>Order Date</th>
                            <th>Status</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach (var order in Model.RecentOrders)
                        {
                            <tr>
                                <td>@order.OrderID</td>
                                <td>@order.CustomerName</td>
                                <td>@order.OrderDate.ToString("yyyy-MM-dd")</td>
                                <td>@order.Status</td>
                            </tr>
                        }
                    </tbody>
                </table>
            </div>
        </div>
    </div>
    <div class="container col-6">
        <!-- Recent Products Section -->
        <div class="card mb-4">
            <div class="card-header">
                <h4>Recent 10 Products</h4>
            </div>
            <div class="card-body">
                <table class="table table-striped">
                    <thead>
                        <tr>
                            <th>Product ID</th>
                            <th>Product Name</th>
                            <th>Category</th>
                            <th>Added Date</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach (var product in Model.RecentProducts)
                        {
                            <tr>
                                <td>@product.ProductID</td>
                                <td>@product.ProductName</td>
                                <td>@product.Category</td>
                                <td>@product.AddedDate.ToString("yyyy-MM-dd")</td>
                            </tr>
                        }
                    </tbody>
                </table>
            </div>
        </div>
    </div>
    <div class="container col-6">
        <!-- Top Customers Section -->
        <div class="card mb-4">
            <div class="card-header">
                <h4>Top 10 Customers by Orders</h4>
            </div>
            <div class="card-body">
                <table class="table table-striped">
                    <thead>
                        <tr>
                            <th>Customer</th>
                            <th>Total Orders</th>
                            <th>Email</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach (var customer in Model.TopCustomers)
                        {
                            <tr>
                                <td>@customer.CustomerName</td>
                                <td>@customer.TotalOrders</td>
                                <td>@customer.Email</td>
                            </tr>
                        }
                    </tbody>
                </table>
            </div>
        </div>
    </div>
    <div class="container col-6">
        <!-- Top Selling Products Section -->
        <div class="card mb-4">
            <div class="card-header">
                <h4>Top 10 Selling Products</h4>
            </div>
            <div class="card-body">
                <table class="table table-striped">
                    <thead>
                        <tr>
                            <th>Product Name</th>
                            <th>Sold Quantity</th>
                            <th>Category</th>
                        </tr>
                    </thead>
                    <tbody>
                        @foreach (var product in Model.TopSellingProducts)
                        {
                            <tr>
                                <td>@product.ProductName</td>
                                <td>@product.TotalSoldQuantity</td>
                                <td>@product.Category</td>
                            </tr>
                        }
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</div>
</div>

<!-- Bootstrap JS (optional) -->
```
