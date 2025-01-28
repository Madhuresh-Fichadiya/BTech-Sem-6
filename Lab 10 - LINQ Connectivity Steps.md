# LINQ Connectivity Steps
## Step 1: Database connectivity with console application
Add following packages in Console Application

**- Microsoft.EntityFrameworkCore.SqlServer**

**- Microsoft.EntityFrameworkCore.Tools**

## Step 2: Add Model class and ApplicationDbContext class files
- Here we have used Employee model, Also Make sure that table name and the class name must be same.
```csharp
//Employee Model class
public class Employee
 {
 [Key]
 public int AccountNo { get; set; }
 public string? FirstName { get; set; }
 public string? LastName { get; set; }
 public string? PhoneNo { get; set; }
 public string? Email { get; set; }
 public decimal? Salary { get; set; }
 public DateTime? JoiningDate { get; set; }
 public int? Age { get; set; }
 public string? City { get; set; }
 public string? Department { get; set; }
 public string? Gender { get; set; }
}
```
- Add ApplicationDbContext class

```csharp
//DbContext class – it is responsible to communicate with database
//DbSet - Represents the collection of all entities in the context, or that can be queried from the database, of a given type.
public class ApplicationDBContext : DbContext
{
 public DbSet<Employee> Employee { get; set; }
 protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
 {
    optionsBuilder.UseSqlServer(@"Data Source=MRF\\SQLEXPRESS;Initial Catalog=StudentRegistration; Trusted_Connection=true;TrustServerCertificate=True;");//change your connection as required
 }
}
```

## Step 3: : In Program.cs file inside main method create ApplicationDBContext object to retrieve data from database.
```csharp
var context = new ApplicationDBContext();
//For example, Display FirstName of all employees.
var q1 = context.Employee.Select(x => x.FirstName);
foreach (var employee in q1)
{
Console.WriteLine("\n {0}", employee);
}
```
