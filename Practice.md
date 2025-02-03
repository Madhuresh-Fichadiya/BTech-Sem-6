## Consider Following Data Set
```csharp
public class Employee
{
    public int EmployeeID { get; set; }
    public string Name { get; set; }
    public string Department { get; set; }
    public double Salary { get; set; }
    public List<string> Skills { get; set; }
    public int Age { get; set; }
    public bool IsPermanent { get; set; }
}
var employees = new List<Employee>
{
    new Employee { EmployeeID = 1, Name = "John", Department = "IT", Salary = 5000, Skills = new List<string> { "C#", "SQL" }, Age = 30, IsPermanent = true },
    new Employee { EmployeeID = 2, Name = "Emma", Department = "HR", Salary = 4000, Skills = new List<string> { "Communication", "Management" }, Age = 28, IsPermanent = false },
    new Employee { EmployeeID = 3, Name = "Michael", Department = "IT", Salary = 7000, Skills = new List<string> { "Java", "Python" }, Age = 35, IsPermanent = true },
    new Employee { EmployeeID = 4, Name = "Sophia", Department = "Finance", Salary = 6000, Skills = new List<string> { "Accounting", "Excel" }, Age = 32, IsPermanent = true },
    new Employee { EmployeeID = 5, Name = "Daniel", Department = "IT", Salary = 5500, Skills = new List<string> { "C#", "JavaScript" }, Age = 27, IsPermanent = false },
    new Employee { EmployeeID = 6, Name = "Olivia", Department = "Marketing", Salary = 4800, Skills = new List<string> { "SEO", "Advertising" }, Age = 29, IsPermanent = true },
    new Employee { EmployeeID = 7, Name = "David", Department = "IT", Salary = 6500, Skills = new List<string> { "Python", "Machine Learning" }, Age = 40, IsPermanent = true },
    new Employee { EmployeeID = 8, Name = "Liam", Department = "HR", Salary = 4200, Skills = new List<string> { "Public Relations", "Recruiting" }, Age = 26, IsPermanent = false },
    new Employee { EmployeeID = 9, Name = "Isabella", Department = "Finance", Salary = 5800, Skills = new List<string> { "Budgeting", "Taxation" }, Age = 31, IsPermanent = true },
    new Employee { EmployeeID = 10, Name = "Ethan", Department = "Marketing", Salary = 5200, Skills = new List<string> { "Social Media", "Copywriting" }, Age = 33, IsPermanent = false }
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
