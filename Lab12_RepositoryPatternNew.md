# Repository Pattern with Asynchronous Methods in ASP.NET Core Web API

## Flow of Web API Using Repository Pattern
![image](https://github.com/user-attachments/assets/733b6b29-556a-4919-bdd1-68eb992b3542)

### Agenda

- What is Repository Pattern?
- Why should we use it?
- Creating Web API Project.
- Prerequisites

## What is Repository Pattern?

The repository pattern is a software design pattern that acts as an abstraction layer between your data access layer and the business logic layer in an ASP.NET Core Web API. It hides the details of how exactly the data is saved or retrieved from the underlying data source. The details of how the data is stored and retrieved are in the respective repository. This means your business logic doesn’t care whether it’s talking to SQL Server, Oracle, or even a mock object for testing purposes.

## Why should we use it?

By using the repository pattern, we are promoting a more loosely coupled approach to access our data from the database. This leads to more clean code and makes it easier to test.

### Benefits

- **Loose Coupling**: By separating data access concerns, your code becomes more loosely coupled, which makes it easier to maintain, test, and modify in the future.
- **Reusability**: You can create generic repository interfaces and concrete repository classes that implement them. This allows you to reuse the same basic functionality for different data entities.
- **Testability**: Since the repository pattern abstracts the data access layer, you can easily create mock repositories for unit testing your business logic without needing a real database connection.

## Creating Web API Project

### Setting Up the Project

1. Create ASP.NET Core Web API Project

### Installing Dependencies

Install the following dependencies via NuGet Package Manager:

- Microsoft.EntityFrameworkCore
- Microsoft.EntityFrameworkCore.SqlServer
- Microsoft.EntityFrameworkCore.Design
- Microsoft.EntityFrameworkCore.Tools

### Defining Entity Models

Create models to represent your database tables. Follow these steps:

1. Right-click on the solution and choose "Add" > "New Folder".
2. Name the folder as "Models".
3. Inside this folder, create files for entity (e.g.,`DepartmentModel.cs`).

**Example Entity Models**:

```csharp
public class DepartmentModel
{
    [Key]
    public int DepartmentId { get; set; }
    public string DepartmentName { get; set; }
}
```

### Adding DbContext

Create a folder named "Data" and add `AppDbContext.cs`.

**AppDbContext.cs**:

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<DepartmentModel> Departments { get; set; }
}
```

In your `appsettings.json` file, add the connection string:

```json
"ConnectionStrings": {
    "DBConnection": "Server=//Your SQL Server name;Initial Catalog=//Your Db;Trusted_Connection=True;Encrypt=False"
}
```

In `Program.cs`, configure the DbContext:

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DBConnection")));
```

### Create and Execute Database Migrations

Use these commands to create and execute migrations:

```bash
Add-Migration InitialCreate
Update-Database
```

## Implementing Repository Pattern

### Create Repository Interface

Create a folder named "Repository" and then create a subfolder "Interface" within it. Add interfaces `IDepartmentRepository.cs`.

**Example Interface**:

```csharp
 public interface IDepartmentRepository
 {
     Task<IEnumerable<DepartmentModel>> GetDepartments();
     Task<DepartmentModel> GetDepartment(int departmentId);
     Task<bool> DeleteDepartment(int departmentId);
     Task<DepartmentModel> UpdateDepartment(DepartmentModel model);
     Task<DepartmentModel> AddDepartment(DepartmentModel model);
 }
```

### Create Repository Class

Inside the "Repository" folder, create a subfolder "Services". Add class `DepartmentRepository.cs`.

**Example Implementation for DepartmentRepository**:

```csharp
 public class DepartmentRepository : IDepartmentRepository
 {
     private readonly AppDbContext appDbContext;

     public DepartmentRepository(AppDbContext appDbContext)
     {
         this.appDbContext = appDbContext;
     }

     public async Task<DepartmentModel> GetDepartment(int departmentId)
     {
         return await appDbContext.Departments
             .FirstOrDefaultAsync(d => d.DepartmentId == departmentId);
     }

     public async Task<IEnumerable<DepartmentModel>> GetDepartments()
     {
         return await appDbContext.Departments.ToListAsync();
     }

     public async Task<DepartmentModel> AddDepartment(DepartmentModel department)
     {
         var result = await appDbContext.Departments.AddAsync(department);

         await appDbContext.SaveChangesAsync();

         return result.Entity;
     }

     public async Task<DepartmentModel> UpdateDepartment(DepartmentModel department)
     {
         var existingDepartment = await appDbContext.Departments
             .FirstOrDefaultAsync(d => d.DepartmentId == department.DepartmentId);

         if (existingDepartment == null) return null;

         existingDepartment.DepartmentName = department.DepartmentName;

         await appDbContext.SaveChangesAsync();

         return existingDepartment;
     }

     public async Task<bool> DeleteDepartment(int departmentId)
     {
         var departmentToDelete = await appDbContext.Departments
             .FirstOrDefaultAsync(d => d.DepartmentId == departmentId);

         if (departmentToDelete == null) return false;

         appDbContext.Departments.Remove(departmentToDelete);

         await appDbContext.SaveChangesAsync();

         return true;
     }
 }
```

### Configure Services in `Program.cs`

Add the following lines to configure the repository services:

```csharp
builder.Services.AddScoped<IDepartmentRepository, DepartmentRepository>();
```

## Creating API Controllers
### DepartmentsController

In the "Controllers" folder, create `DepartmentsController.cs`.

**Example Controller for Departments**:

```csharp
[Route("api/[controller]")]
[ApiController]
public class DepartmentsController : ControllerBase
{
    private readonly IDepartmentRepository departmentRepository;

    public DepartmentsController(IDepartmentRepository departmentRepository)
    {
        this.departmentRepository = departmentRepository;
    }

    [HttpGet]
    public async Task<ActionResult> GetDepartments()
    {
        try
        {
            return Ok(await departmentRepository.GetDepartments());
        }
        catch (Exception)
        {
            return StatusCode(StatusCodes.Status500InternalServerError,
                "Error retrieving data from the database");
        }
    }

    [HttpGet("{id:int}")]
    public async Task<ActionResult<DepartmentModel>> GetDepartment(int id)
    {
        try
        {
            var result = await departmentRepository.GetDepartment(id);
            if (result == null) return NotFound();
            return result;
        }
        catch (Exception)
        {
            return StatusCode(StatusCodes.Status500InternalServerError,
                "Error retrieving data from the database");
        }
    }

    [HttpPost]
    public async Task<ActionResult<DepartmentModel>> CreateDepartment(DepartmentModel department)
    {
        try
        {
            if (department == null)
                return BadRequest("Department cannot be null");

            var createdDepartment = await departmentRepository.AddDepartment(department);
            return CreatedAtAction(nameof(GetDepartment), new { id = createdDepartment.DepartmentId }, createdDepartment);
        }
        catch (Exception)
        {
            return StatusCode(StatusCodes.Status500InternalServerError,
                "Error creating new department record");
        }
    }

    [HttpPut("{id:int}")]
    public async Task<ActionResult<DepartmentModel>> UpdateDepartment(int id, DepartmentModel department)
    {
        try
        {
            if (id != department.DepartmentId)
                return BadRequest("Department ID mismatch");

            var updatedDepartment = await departmentRepository.UpdateDepartment(department);
            if (updatedDepartment == null)
                return NotFound($"Department with Id = {id} not found");

            return updatedDepartment;
        }
        catch (Exception)
        {
            return StatusCode(StatusCodes.Status500InternalServerError,
                "Error updating data");
        }
    }

    [HttpDelete("{id:int}")]
    public async Task<ActionResult> DeleteDepartment(int id)
    {
        try
        {
            var success = await departmentRepository.DeleteDepartment(id);
            if (!success)
                return NotFound($"Department with Id = {id} not found");

            return NoContent();
        }
        catch (Exception)
        {
            return StatusCode(StatusCodes.Status500InternalServerError,
                "Error deleting department");
        }
    }
}
```

## Conclusion

**Repository Pattern Flow**: 

- **Controller** → **Repository** → **EF Core** → **SQL Server**

The repository pattern is an essential design pattern for scalable and maintainable applications, particularly for complex data access scenarios.
