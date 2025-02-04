# Basic CRUD Operations using EF Core
## Step 1: Create a WebAPI project and add following packages
**- Microsoft.EntityFrameworkCore.SqlServer**

**- Microsoft.EntityFrameworkCore.Tools**

**- System.ComponentModel.DataAnnotations**
## Step 2: Prepare Model classes
- Course- CourseId, CourseName
- Student - StudentId, Name, Enrollment, Semester
- Department- DepartmentId, DepartmentName
```csharp
[Table("Courses")]
public class Course
{
    #region Properties

    [Key]
    public int CourseId { get; set; }

    [Required]
    [StringLength(100)]
    public string CourseName { get; set; }

    #endregion
}
```
```csharp
[Table("Students")]
public class Student
{
    #region Properties

    [Key]
    public int StudentId { get; set; }

    [Required]
    [StringLength(100)]
    public string Name { get; set; }

    [Required]
    [StringLength(20)]
    public string Enrollment { get; set; }

    [Required]
    public int Semester { get; set; }

    #endregion
}
```
```csharp
[Table("Departments")]
    public class Department
    {
        #region Properties

        [Key]
        public int DepartmentId { get; set; }

        [Required]
        [StringLength(100)]
        public string DepartmentName { get; set; }
        #endregion
    }
```
## Step 3: Add Connection string in appSettings.json file
```csharp
"ConnectionStrings": {
    "DefaultConnection": "Your_SQL_Server_Connection_String_Here"
}
```
## Step 4: Register ApplicationDbContext with dependency injection in Program.cs
```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```
## Step 5: Prepare ApplicationDbContext class
Create a Data folder in root directory and add ApplicationDbContext inside Data folder.
```csharp
public class ApplicationDbContext: DbContext
{
    private readonly IConfiguration configuration;
    public ApplicationDbContext(IConfiguration _configuration)
    {
        configuration = _configuration;
    }

    // DbSet properties represent collections of the entities.
    public DbSet<Course> Courses { get; set; }
    public DbSet<Student> Students { get; set; }
    public DbSet<Department> Departments { get; set; }

    // Configure the database connection string.
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // SQL Server connection string.
        optionsBuilder.UseSqlServer(this.configuration.GetConnectionString("Default"));
    }
}
```
## Step 6: Add Migration to generate Database
- run following commands in Package Manager Console
```csharp
//Add-Migration MigrationName
Add-Migration InitialCommit

//Update Database
Update-Database
```

## Step 7: Add Controller and Implement CRUD
```csharp
[ApiController]
[Route("api/[controller]/[action]")]
public class StudentController : ControllerBase
{
    // Inject ApplicationDbContext class dependency
    private readonly ApplicationDbContext _context;
    public StudentController(ApplicationDbContext context)
    {
        _context = context;
    }

    // GET: api/Students
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Student>>> GetStudents()
    {
        return await _context.Students.ToListAsync();
    }

    // GET: api/Students/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Student>> GetStudent(int id)
    {
        var student = await _context.Students.FindAsync(id);

        if (student == null)
        {
            return NotFound();
        }

        return student;
    }

    // PUT: api/Students/5
    [HttpPut("{id}")]
    public async Task<IActionResult> PutStudent(int id, Student student)
    {
        if (id != student.StudentId)
        {
            return BadRequest();
        }

        try
        {
            _context.Update(student);
            await _context.SaveChangesAsync();
        }
        catch (Exception)
        {
            if (!StudentExists(id))
            {
                return NotFound();
            }
            else
            {
                throw;
            }
        }

        return NoContent();
    }

    // POST: api/Students
    [HttpPost]
    public async Task<ActionResult<Student>> AddStudent(Student student)
    {
        _context.Students.Add(student);
        await _context.SaveChangesAsync();

        return CreatedAtAction("GetStudent", new { id = student.StudentId }, student);
    }

    // DELETE: api/Students/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteStudent(int id)
    {
        var student = await _context.Students.FindAsync(id);
        if (student == null)
        {
            return NotFound();
        }

        _context.Students.Remove(student);
        await _context.SaveChangesAsync();

        return NoContent();
    }

    private bool StudentExists(int id)
    {
        return _context.Students.Any(e => e.StudentId == id);
    }
}
```
