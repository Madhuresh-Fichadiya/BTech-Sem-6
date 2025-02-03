## Consider Following Data Set
```csharp
public class Student
{
    public int StudentID { get; set; }
    public string Name { get; set; }
    public int Marks { get; set; }
    public string Grade { get; set; }
}

public class Employee
{
    public int EmployeeID { get; set; }
    public string Name { get; set; }
    public double Salary { get; set; }
    public string Department { get; set; }
    public int Experience { get; set; }
}

public class Product
{
    public int ProductID { get; set; }
    public string Name { get; set; }
    public double Price { get; set; }
    public int Stock { get; set; }
    public string Category { get; set; }
}
var students = new List<Student>
{
    new Student { StudentID = 1, Name = "Alice", Marks = 85, Grade = "A" },
    new Student { StudentID = 2, Name = "Bob", Marks = 70, Grade = "B" },
    new Student { StudentID = 3, Name = "Charlie", Marks = 60, Grade = "C" },
    new Student { StudentID = 4, Name = "David", Marks = 95, Grade = "A" },
    new Student { StudentID = 5, Name = "Eve", Marks = 45, Grade = "D" }
};

var employees = new List<Employee>
{
    new Employee { EmployeeID = 1, Name = "John", Salary = 60000, Department = "IT", Experience = 5 },
    new Employee { EmployeeID = 2, Name = "Sara", Salary = 45000, Department = "HR", Experience = 3 },
    new Employee { EmployeeID = 3, Name = "Mike", Salary = 70000, Department = "Finance", Experience = 7 },
    new Employee { EmployeeID = 4, Name = "Anna", Salary = 90000, Department = "IT", Experience = 10 }
};

var products = new List<Product>
{
    new Product { ProductID = 1, Name = "Laptop", Price = 1000, Stock = 10, Category = "Electronics" },
    new Product { ProductID = 2, Name = "Phone", Price = 500, Stock = 5, Category = "Electronics" },
    new Product { ProductID = 3, Name = "Shirt", Price = 30, Stock = 50, Category = "Clothing" },
    new Product { ProductID = 4, Name = "Pants", Price = 40, Stock = 30, Category = "Clothing" }
};
```

## Perform Following Queries
1. Get a list of employee names.
2. Get names and their respective departments.
3. Get names of employees earning more than 5000.
4. Get the names in uppercase.
5. Get employee IDs along with salaries as strings.
6. Get all unique skills.
7. Get employees who know "C#".
8. Get department-wise skill sets.
9. Get employees older than 30.
10. Get permanent employees.
11. Get employees earning between 4000 and 6000.
12. Get employees whose names start with 'J'.
13. Get all integer-type salaries (assuming list contains mixed data).
14. Get all boolean properties.
15. Get the total salary expense.
16. Find the highest salary.
17. Find the lowest salary.
18. Find the average salary.
19. Group employees by department.
20. Find the count of employees in each department.
21. Get the highest salary in each department.
22. Get employees sorted by salary in descending order.
23. Get employees sorted first by department, then by salary.
24. Find the employee with the highest salary.
25. Find the second-highest salary.
26. Get all employees with names having at least one vowel.
27. Get department-wise average salary excluding lowest salary.
28. Get department-wise average salary excluding lowest salary.
29. Get employees whose names are palindromes.
30. Get department with the highest number of employees.
