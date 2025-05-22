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
    dep.Name , 
    p.FirstName , 
    p.LastName , 
    p.PersonType  ,
    emp.salary
    from HumanResources.Employee emp
    left join HumanResources.EmployeeDepartmentHistory emph
    on emp.BusinessEntityID  =  emph.BusinessEntityID
    left join HumanResources.Department dep on emph.DepartmentID = dep.DepartmentID
    left join Person.Person p on emp.BusinessEntityID =  p.BusinessEntityID 
