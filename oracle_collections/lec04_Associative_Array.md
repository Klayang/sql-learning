###### What is An Associative Array?

```plsql
DECLARE
TYPE list_of_names_t IS TABLE OF employees.last_name%TYPE 
INDEX BY PLS_INTEGER;
```

- An *associative array* is a variable declared from an *associative array type*

- From the *PL/SQL* user guide:
  
  - An unbouned set of key-value pairs
  
  - Each key is unique & serves as subscript of the element that holds value
  
  - Can access elements without knowing their positions in the array
    
    - And without traversing the array

    

###### Associative Array Background

- It was the first collection type available in *PL/SQL*

- First introduced in Oracle 7 as a *PL/SQL table* (where the `TABLE OF` comes from)

- Renamed in Oracle 8 to *index-by table* when *nested table*s & *varray*s were added

- In 9.2, renamed to *associative array* with the advent of string indexing

- Can only be used in *PL/SQL* blocks

    

###### Characteristics of Associative Arrays

- The datatype following `IS TABLE OF` can be almost any valid *PL/SQL* type
  
  - We'll cover some exceptions in detail very soon

- `INDEX BY` type can be `INTEGER` or `STRING`
  
  - But the index value can never be `NULL`

- *Assciative array*s can be sparse
  
  - Can populate elements in non-consecutive index values
  
  - Easily used to emulate *primary key*s or *unique indices*

    

###### Associative Array of Records Example (1 / 4)

Very easy to emulate a *relational table* inside one's *PL/SQL* code

```plsql
DECLARE
   TYPE employees_aat IS TABLE OF employees%ROWTYPE INDEX BY PLS_INTEGER;
   l_employees employees_aat;
BEGIN
   FOR employee_rec in (SELECT * FROM employees)
   LOOP
       l_employees(l_employees.COUNT + 1) := employee_rec;
   END LOOP;
END;
```

    

###### Associative Array of Records Example (2 / 4)

Emulate the table by `CURSOR`

```sql
DECLARE
    CURSOR all_emps_cur IS SELECT * FROM employees;
    -- same employees_aat & l_employees
BEGIN
    FOR employee_rec in all_emps_cur
    LOOP
        l_employees(all_emps_cur%ROWCOUNT) := employee_rec;
    END LOOP;
END;
```

    

###### Associative Array of Records Example (3 / 4)

Emulate the table by `ID` (a `int` primary key)

```sql
DECLARE
    TYPE employees_by_id IS TABLE OF employees%ROWTYPE 
        INDEX BY PLS_INTEGER;
    l_employees employees_by_id;
BEGIN
    FOR employee_rec in (SELECT * FROM employees)
    LOOP
        l_employees(employee_rec.id) := employee_rec;
    END LOOP;
END;
```

    

###### Associative Array of Records Example (4 / 4)

Emulate the table by `NAME` (a `STRING` primary key)

```sql
DECLARE
    TYPE employees_by_name IS TABLE OF employees%ROWTYPE 
        INDEX BY employees.name%TYPE;
    l_employees employees_by_name;
BEGIN
    FOR employee_rec IN (SELECT * FROM employees)
    LOOP
        l_employees(employee_rec.name) = employee_rec;
    END LOOP;
END;
```

    

###### Valid Table of Datatypes

- Can create an *associative array* of almost any *PL/SQL* or *SQL* datatypes
  
  - Any scalar types, including `BOOLEAN`
  
  - Collection of object types
  
  - Collection of other collections

- The restrictions are:
  
  - Cannot have a `TABLE OF` *cursor* variables or exceptions

    

###### Valid Index Values

Since an `INT` could range from -2<sup>31</sup> to 2<sup>31</sup> - 1

- That's the binary for integer indices (more than 4 billion values!)

- String indices can have any value, only restricted by that total # (4 billion)

    

###### 
