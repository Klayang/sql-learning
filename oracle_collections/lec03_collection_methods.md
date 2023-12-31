###### Introducing Collection Methods

- Collection methods are functions attached to a collection variable

- It's the first introduction of object-oriented syntax in *PL/SQL*

    

###### Methods That Retrieve Information (Observer)

- `COUNT`: # rows currently defined in the collection

- `EXISTS`: `TRUE` is the specified row is defined

- `FIRST`/`LAST`: lowest/highest index of defined rows

- `NEXT`/`PRIOR`: defined row after/before the specified row

- `LIMIT`: max # elements allowed in a `VARRAY`

    

###### COUNT

```sql
DECLARE
    l_my_list dbms_sql.varchar2_table;
BEGIN
    l_my_list := dbms_sql.varchar2_table();
    dbms_output.put_line(l_my_list.count());
END;
```

![](C:\Users\yangs\AppData\Roaming\marktext\images\2023-12-23-08-07-45-image.png)

    

###### EXISTS (1 / 2)

```sql
DECLARE
    l_my_list dbms_sql.varchar2_table;
BEGIN
    l_my_list := dbms_sql.varchar2_table();
    IF l_my_list.EXISTS(0)
    THEN dbms_output.put_line(l_my_list.count());
    ELSE dbms_output.put_line('we have no row at the index 0');
    END IF;
END;
```

![](C:\Users\yangs\AppData\Roaming\marktext\images\2023-12-23-08-09-45-image.png)

    

###### EXISTS (2 / 2)

Reading an element at an undefined index would cause the `NO_DATA_FOUND` exception

```sql
DECLARE
    l_my_list dbms_sql.varchar2_table;
BEGIN
    l_my_list := dbms_sql.varchar2_table();
    dbms_output.put_line(l_my_list(0));
END;
```

![](C:\Users\yangs\AppData\Roaming\marktext\images\2023-12-23-08-17-23-image.png)

    

###### Navigating through Collections (1 / 2)

One of the most common actions on collections is to loop throught the contents

- You can use `WHILE`, `FOR` or just simple loops to perform this navigation

- The characteristics of your collection will determine what sort of loop to use

    

###### Navigating through Collections (2 / 2)

- `FOR` loop: don't use with sparse collections

- `WHILE` & simple loop: best fit for sparse collection when you want to:
  
  - Conditionally exit

    

###### FOR Loops

```sql
FOR indx IN 1 .. my_collection.COUNT
LOOP
    dbms_output.put_line(my_collection(indx));
END LOOP;
```

```sql
FOR indx IN my_collection.FIRST .. my_collection.LAST
LOOP
    dbms_output.put_line(my_collection(indx));
END LOOP;
```

    

###### WHILE/Simple Loop

```sql
row_indx PLS_INTEGER := my_collection.FIRST;
```

```sql
LOOP
    EXIT WHEN row_indx IS NULL;
    row_indx = my_collection.NEXT(row_indx);
END LOOP;
```

```sql
LOOP
    EXIT WHEN row_indx IS NULL;
    row_indx = my_collection.PRIOR(row_indx);
END LOOP;
```

    

###### Methods that Change the Collection (Mutator)

- `DELETE` deletes one or more rows from the collection

- `EXTEND` adds rows to the end of a *nested table* or *varray*

- `TRIM` removes rows from a `VARRAY`

    

###### DELETE

- Can only be executed on *nested table*s or *associative array*s

- If you try to execute it on a *varray*, `PLS-00306` error would show up

- If a range of index is specified, the low & high values don't need to exist
  
  ```sql
  -- delete all rows
  my_collection.DELETE();
  ```
  
  ```sql
  -- delete the last row
  my_collection.DELETE(my_collection.LAST);
  ```
  
  ```sql
  -- delete a range of rows
  my_collection.DELETE(1321, 32532);
  ```

    

###### EXTEND (1 / 2)

This method is used t make room for new elements

- Can only be used with *nested table*s & *varray*s

- Tell Oracle to add `N` elements to the end of the collection

- *BULK* `EXTEND` is faster than individual `EXTEND`s
  
  - If you know the need of 10,000 elements, do it all at once

- Optional: can specify the value of new element from an exisiting one
  
  - The default is `NULL`

    

###### EXTEND (2 / 2)

```sql
DECLARE
    TYPE employees_aat IS TABLE OF employees%ROW_TYPE;
    l_employees employees_aat := employees_aat();
BEGIN
    -- extend a single row
    l_employees.EXTEND();
    l_employees(1).last_name := 'Yang';

    -- extend 5 more rows, all set to null
    l_employees.EXTEND(5);

    -- extend 10 more rows, all set to be the same as the 1st
    l_employees.EXTEND(10, 1);
END;
```

    

###### TRIM

This method removes elements from end of the collection

- Use `TRIM` only with *varray*s & *nested table*s

- Can `TRIM` one or multiple elements. However, `ORA-06533` shows up: 
  
  - If the amount you trim > `COUNT`

- `TRIM` is the only way to remove elements from a *varray*
  
  - `DELETE` is not supported

    

###### 
