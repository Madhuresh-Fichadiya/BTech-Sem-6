# Basic LINQ Operators
## LINQ Queries on Projection operators

```csharp
// 1. Display FirstName of all employees.
var q1 = context.Employee.Select(x => x.FirstName);
foreach (var employee in q1)
{
  Console.WriteLine("\n {0}", employee);
}

// 2. Select ActNo, FirstName and Salary of all employees to a new class and
display it.
var q2 = context.Employee.Select(x => new Employee { AccountNo = x.AccountNo, FirstName = x.FirstName, Salary = x.Salary });
foreach (var employee in q2)
{
  Console.WriteLine("AccountNo: {0}, First Name: {1}, Salary: {2}", employee.AccountNo, employee.FirstName, employee.Salary);
}

// 3. Display data in following format => {Anil} works in {Admin} Department.
var q3 = context.Employee.Select(x => new { ans = x.FirstName + " works in " + x.Department + " Deapartmet" });
foreach (var item in q3)
{
  Console.WriteLine(item.ans);
}

// 4. Select Employee Full Name, Email and Department as anonymous and display it.
var q4 = context.Employee.Select(x => "Name: " + x.FirstName + " " + x.LastName + "\tEmail: " + x.Email + "\tDept: " + x.Department).ToList();
foreach (var item in q4)
  Console.WriteLine(item);

// 5. Display employees with their joining date.
var q5 = context.Employee.Select(x => x.FirstName + " " + x.JoiningDate);
foreach (var item in q5)
  Console.WriteLine(item);
```

## LINQ Queries on Filtering operators
```csharp
// 6. Display employees between age 20 to 30.
var q6 = context.Employee.Where(x => x.Age > 20 && x.Age < 30);
foreach (var item in q6)
  Console.WriteLine(item.FirstName);

// 7. Display female employees.
var q7 = context.Employee.Where(x => x.Gender == "Female");
foreach (var item in q7)
  Console.WriteLine(item.FirstName);

// 8. Display employees with salary more than 35000.
var q8 = context.Employee.Where(x => x.Salary > 35000);
foreach (var item in q8)
  Console.WriteLine(item.FirstName);

// 9. Display employees whose account no is less than 110.
var q9 = context.Employee.Where(x => x.AccountNo < 110);
foreach (var item in q9)
  Console.WriteLine(item.FirstName);

// 10. Display employees who belongs to either Rajkot or Morbi city.
var q10 = context.Employee.Where(x => x.City == "Rajkot" || x.City == "Morbi");
foreach (var item in q10)
  Console.WriteLine(item.FirstName);

// 11. Display employees whose salary is less than 20000.
var q11 = context.Employee.Where(x => x.Salary < 20000);
foreach (var item in q11)
  Console.WriteLine(item.FirstName);

// 12. Display employees whose salary is more than equal to 30000 and less than equal to 60000.
var q12 = context.Employee.Where(x => x.Salary >= 30000 && x.Salary <= 60000);
foreach (var item in q12)
  Console.WriteLine(item.FirstName);

// 13. Display ActNo, FirstName and Amount of employees who belong to Morbi or Ahmedabad or Surat city and Account No greater than 120.
var q13 = context.Employee.Where(x => x.AccountNo > 120 && (x.City == "Morbi" || x.City == "Ahmedabad" || x.City == "Surat"));
foreach (var item in q13)
  Console.WriteLine(item.AccountNo);

// 14. Display male employees of age between 30 to 35 and belongs to Rajkot city.
var q14 = context.Employee.Where(x => (x.Age >= 30 && x.Age <= 35) && x.City == "Rajkot" && x.Gender == "Male");
foreach (var item in q14)
  Console.WriteLine(item.FirstName);

// 15. Display Unique Cities. (use Distinct())
var q15 = context.Employee.Select(x => x.City).Distinct();
foreach (var item in q15)
  Console.WriteLine(item);

// 16. Display employees whose joining is between 15/07/2018 to 16/09/2019.
var q16 = context.Employee.Where(x => x.JoiningDate >= new DateTime(2018, 07, 15) && x.JoiningDate <= new DateTime(2019, 09, 16));
foreach (var item in q16)
  Console.WriteLine(item.FirstName);

// 17. Display female employees who works in Sales department.
var q17 = context.Employee.Where(x => x.Gender == "Female" && x.Department == "Sales");
foreach (var item in q17)
  Console.WriteLine(item.FirstName);

// 18. Display employees with their Age who belong to Rajkot city and more than 35 years old.
var q18 = context.Employee.Where(x => x.City == "Rajkot" && x.Age > 35);
foreach (var item in q18)
  Console.WriteLine(item.FirstName);
```

## LINQ Queries on Aggregare operators
```csharp
//19. Find total salary and display it.
var q19 = context.Employee.Sum(x => x.Salary);
  Console.WriteLine(q19);

//20. Find total number of employees of Admin department who belongs to Rajkot city.
var q20 = context.Employee.Where(x => x.Department == "Admin" && x.City == "Rajkot").Count();
  Console.WriteLine(q20);

//21. Find total salary of Distribution department.
var q21 = context.Employee.Where(x => x.Department == "Distribution").Sum(x => x.Salary);
  Console.WriteLine(q21);

//22. Find average salary of IT department.
var q22 = context.Employee.Where(x => x.Department == "IT").Average(x => x.Salary);
  Console.WriteLine(q22);

//23. Find minimum salary of Customer Relationship department.
var q23 = context.Employee.Where(x => x.Department == "Customer Relationship").Min(x => x.Salary);
  Console.WriteLine(q23);

//24. Find maximum salary of Distribution department who belongs to Baroda city.
var q24 = context.Employee.Where(x => x.Department == "Distribution").Max(x => x.Salary);
  Console.WriteLine(q24);

//25. Find number of employees whose Age is more than 40
var q25 = context.Employee.Where(x => x.Age > 40).Count();
  Console.WriteLine(q25);

//26. Find total female employees working in Ahmedabad city.
var q26 = context.Employee.Where(x => x.City == "Ahmedabad" && x.Gender == "Female").Count();
  Console.WriteLine(q26);

//27. Find total salary of male employees who belongs to Gandhinagar city and works in IT department.
var q27 = context.Employee.Where(x => x.City == "Gandhinagar" && x.Department == "IT").Sum(x => x.Salary);
  Console.WriteLine(q27);

//28. Find average salary of male employees whose age is between 21 to 35.
var q28 = context.Employee.Where(x => x.Gender == "Male" && x.Age >= 21 && x.Age <= 25).Average(x => x.Salary);
  Console.WriteLine(q28);
```

## LINQ Queries on Ordering operators
```csharp
// 29. Display employees by their first name in ascending order.
var q29 = context.Employee.OrderBy(x => x.FirstName);
foreach (var item in q29)
  Console.WriteLine(item.FirstName);

// 30. Display employees by department name in descending order.
var q30 = context.Employee.OrderByDescending(x => x.Department);
foreach (var item in q30)
  Console.WriteLine(item.FirstName);

// 31. Display employees by department name descending and first name in
ascending order.
var q31 = context.Employee.OrderByDescending(x => x.Department).ThenBy(x => x.FirstName);
foreach (var item in q31)
  Console.WriteLine(item.FirstName);

// 32. Display employees by their first name in ascending order and last name in descending order.
var q32 = context.Employee.OrderBy(x => x.FirstName).ThenBy(x => x.LastName);
foreach (var item in q32)
  Console.WriteLine(item.FirstName);

// 33. Display employees by their Joining Date using Reverse() operator.
var q33 = context.Employee.OrderBy(x => x.JoiningDate).Reverse();
foreach (var item in q33)
  Console.WriteLine(item.FirstName);
```
