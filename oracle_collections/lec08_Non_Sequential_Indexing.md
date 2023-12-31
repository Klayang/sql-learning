###### Non-Sequential Indexing

- Sometimes you simply want to add items to the end of a list. This makes sense:
  
  - If the order in which items are added matters

- But how do you find a specific element in a list? With sequential indexing: 
  
  - Have to scan through the contents to find the match

- And what if you want to find elements in a collection using more than 1 *index*?
  
  - No way around that

    

###### Taking Advantage of Non-Sequential Indexing (1 / 3)

Take a look at the following 2 versions of looking up employee info in a table

```sql
CREATE OR REPLACE FUNCTION GetEmployeeInfo (ID IN PLS_INTEGER)
RETURN EMPLOYEE%ROWTYPE AS
DECLARE
    employeeInfo EMPLOYEE%ROWTYPE;
BEGIN
    SELECT * INTO employeeInfo FROM EMPLOYEE WHERE EMPLOYEEID = ID;
    RETURN employeeInfo; 
END;
```

    

###### Taking Advantage of Non-Sequential Indexing (2 / 3)

```sql
CREATE OR REPLACE TYPE employee_aat IS TABLE OF EMPLOYEE%ROWTYPE
INDEX BY PLS_INTEGER;
```

```sql
employeeCache employ_aat;
```

```sql
BEGIN
    FOR employeeRec in (SELECT * FROM EMPLOYEE)
    LOOP
        employeeCache(employeeRec.EMPLOYEEID) := employeeRec;
    LOOP END;
END;
```

```sql
CREATE OR REPLACE FUNCTION GetEmployeeInfo (ID IN PLS_INTEGER)
RETURN EMPLOYEE%ROWTYPE AS
DECLARE
    employeeInfo EMPLOYEE%ROWTYPE;
BEGIN
    RETURN employeeCache(ID);
END;
```

    

###### Taking Advantage of Non-Sequential Indexing (3 / 3)

- The 1st version scan through the table every time when the function is called

- The 2nd version first stores the whole table into an *associative array*, then return:
  
  - The given row based on the passed-in `ID`, without scanning through the table

- The 2nd version greatly reduces the execution time if a lot of look-ups are enforced
  
  - Due to the cache and it's use of `PGA` rather than `SGA`

    

###### Multiple Indices on A Collection

- Most relational tables have multiple indices to optimize query performance

- Can create multiple collections based on the same table to emulate the same effect

    


