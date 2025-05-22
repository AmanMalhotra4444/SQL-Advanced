## SQL Queries on AdventureWorks Dataset
### Create an View
 ```sql
  Create or alter view dbo.employees
  as
    select 
    emp.BusinessEntityID , 
    emp.JobTitle , 
    emp.BirthDate , 
    emp.HireDate , 
    emp.MaritalStatus , 
    dep.Name as department_name, 
    p.FirstName , 
    p.LastName , 
    p.PersonType  ,
    emp.salary
    from HumanResources.Employee emp
    left join HumanResources.EmployeeDepartmentHistory emph
    on emp.BusinessEntityID  =  emph.BusinessEntityID
    left join HumanResources.Department dep on emph.DepartmentID = dep.DepartmentID
    left join Person.Person p on emp.BusinessEntityID =  p.BusinessEntityID 
```

###  Select Second Highest Salary by department
```sql
With cte as (
Select 
BusinessEntityID , FirstName , LastName ,salary ,  Department_name , 
Dense_RANK() over (partition by department_name order by salary desc) as _rank
from dbo.employees )

select BusinessEntityID , 
	   FirstName , 
	   LastName ,
	   salary ,  
	   Department_name 
from cte where _rank = 2
order by Department_name
```

