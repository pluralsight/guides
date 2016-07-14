When it comes to how to analyze employee data, there are fewer more complex data structures to work with. Probably the biggest challenge with working with an employee data set is how unassuming it may seem to start. Let's take a typical employee table from a Human Resources Information System (HRIS) and see how we can run some common analytical queries against it.

Here is a table structure for how our data is stored. This is typical of many HRIS systems I've seen in my days so we'll stick with this as an example.

## Table Name: emps

| EmployeeID | HireDate   | RehireDate | TermDate   | ManagerID | Name       |
|------------|------------|------------|------------|-----------|------------|
| 1          | 2004-06-17 | 1900-01-01 | 3000-01-01 | -1        | Washington |
| 2          | 2004-07-01 | 1900-01-01 | 3000-01-01 | 1         | Adams      |
| 3          | 2008-05-01 | 1900-01-01 | 2008-07-01 | 2         | Rosevelt   |
| 4          | 2007-01-01 | 2009-01-01 | 2008-01-01 | 3         | Carter     |

### How many people were active on a given day (ex. 2005-07-01)?

If we were answering these questions independently it the following SQL would suffice:

``` sql
select count(1)
from emps
where HireDate <= '2005-07-01'
and TermDate > '2005-07-01'
```

Rinse and repeat this for every date. But what about employees that have been rehired?

First based on this data, we must assume that when a person is rehired, their TermDate remains the same and does not reset to 3000-01-01 as if they were active.

``` sql
select count(1)
from emps
where (HireDate >= '2005-07-01'
and TermDate < '2005-07-01')
or (RehireDate >= '2005-07-01'
and (
    TermDate < '2005-07-01'
  or
    TermDate <= RehireDate
)

```

### The plot thickens

This is all easy enough, but what if we wanted to create a view that has the running total of active employees for every day so we could trend it over time and ask more complex questions like Year over Year (YoY) growth?

Now we need to include another table which has nothing but dates. In Data Warehousing we typically create this as a Date dimension. Here is a basic structure for us to work with.

### Table Name: dates

| Date       | Day | Month | Year |
|------------|-----|-------|------|
| 2005-01-01 | 1   | 5     | 2005 |
| 2005-01-02 | 2   | 5     | 2005 |
| etc...     |     |       |      |


We'll use this as our starting point since it won't be constrained by the employee status at the time or dates in our emp table. From there we'll do a left join to our emp table to get the count of active employees similar to how we did it manually above. Here's the SQL to do this:

``` sql
select
DayDate, count(hires.EmployeeId) as ActiveCount
from 
    dates d
-- get hires
left join 
 (select 'active' as Status, * from emps) hires
on
 ( d.Date >= hires.HireDate and d.Date < hires.TermDate) -- normal active
or
  (d.Date 
      between (case when hires.RehireDate = '1900-01-01' then null else hires.RehireDate end) 
      and hires.TermDate)
where 
  d.Date between '2004-06-17' and now() -- use the function to pull the current date here
```

### Alas, we have trend data!

The results of our query above will give us a daily count of Active employees to trend over time. This is just the first step in analyzing employee data of course, but a solid foundation to start and a good example of why having a date dimension in your data warehouse is important.




