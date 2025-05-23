## SQL Queries on AdventureWorks Dataset
### Create View
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
### Show all the records where salary is higher than the average salary (using variable)
```sql
declare @avg_salary  int
select @avg_salary = AVG(salary) from dbo.employees

select * from dbo.employees where salary > @avg_salary
```
### Employees with Salaries Higher Than Their Departmental Average 
```sql
With CTE as (

Select 
BusinessEntityID , FirstName , LastName ,salary ,  Department_name ,
AVG(salary) over(partition by department_name) as avg_salary_by_department
from dbo.employees )

select BusinessEntityID , 
	   FirstName , 
	   LastName ,
	   salary ,  
	   Department_name 
from cte where salary > avg_salary_by_department
order by Department_name
```
### Retrieve the BirthDate, Full Name, and Age of employees whose birthday is today.
```sql
Select emp.BusinessEntityID , 
		emp.BirthDate , 
		CONCAT(P.FirstName , ' ', P.MiddleName, ' ',p.LastName) as 'Full Name', 
		Datediff(year,BirthDate,cast(GETDATE() as date)) as Age
from HumanResources.Employee emp
cross join Person.Person p
where emp.BusinessEntityID = p.BusinessEntityID
and month(emp.BirthDate) = month(GETDATE())
and DAY(emp.BirthDate) = DAY(GETDATE())
```
### Create a procedure with two parameters Mobile No. and Account No. in order to get the purchase history of Customer
```sql
Create procedure customer_purchase_history @phone_number varchar(15) , @AccountNumber varchar(15)
as
begin
Select 
	c.[AccountNumber] , 
	CONCAT(p.FirstName ,' ' , p.LastName) as Customer_name ,
	sales.OrderDate,
	pp.PhoneNumber  ,
	prod.name as product
	from Sales.Customer c
	left join Person.Person p on p.BusinessEntityID = c.PersonID
	left join Person.PersonPhone pp on p.BusinessEntityID = pp.BusinessEntityID
	left join Sales.SalesOrderHeader sales on sales.CustomerID = c.CustomerID
	left join Sales.SalesOrderDetail salesdetail on salesdetail.SalesOrderID = sales.SalesOrderID
	left join Production.Product prod on prod.ProductID = salesdetail.productId
where p.BusinessEntityID = c.PersonID
and pp.PhoneNumber = @phone_number and c.AccountNumber = @AccountNumber
end
```

### Running Total With Percentage
```sql
Select orderdate , 
		Sales,
		SUM(sales) over() as overall_sales,
		Round(cast(Sales as float) * 100 / SUM(sales) over(),2) as percentage_of_overall_sales , 

		SUM(sales) over(partition by year(orderdate)) as overall_sales_by_year,
		Round(cast(Sales as float) * 100 / SUM(sales) over(partition by year(orderdate)),2) as percentage_of_overall_sales_by_year,

		SUM(sales) over(order by orderdate) as commulative_Sales,
		Round(SUM(cast(sales as float)) over(order by orderdate)*100 / SUM(sales)over() ,2) as commulative_sales_percentage ,

		SUM(sales) over(partition by year(orderdate) order by orderdate) as commulative_Sales_by_year,
		Round(SUM(cast(sales as float)) over(partition by year(orderdate) order by orderdate)*100 / SUM(sales)over(partition by year(orderdate)) ,2) as commulative_sales_percentage_by_year 

from dbo.Sales
order by ORDERDATE
```
### Calculation of Moving Average and Rolling Average 
```sql
Select orderdate , 
		Sales,
		AVG(sales) over(order by orderdate rows between 2 preceding and current row) as moving_avg,
		AVG(Sales) over(order by orderdate rows between 1 preceding and 1 following) as rolling_avg
from dbo.Sales
order by ORDERDATE
```
### Month over month Change in Sales
```sql
with CTE as (
Select 
	YEAR(orderdate) as _year,
	month(orderdate) as _month , 
	Sum(Sales) as sales
From dbo.Sales
group by 
	YEAR(orderdate),
	month(orderdate)) 

select * , 
Sales - LAG(sales) over(order by _year , _month ) as Month_over_Month_Change
from CTE
order by _year, _month
```





