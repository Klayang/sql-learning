###### What is A Table Function? (1 / 2)

- A *table function* is a function that can be invoked inside the `FROM` clause

- They return collections, which can then be transformed with the `TABLE` clause 
  
  - Into a dataset of rows & columns

    

###### What is A Table Function? (2 / 2)

- To call a function from within the `FROM` clause of a query, you need to:

- Define the `RETURN` datatype of the function to be a collection type 
  
  - Usually a *nested table* or *varray`*, but sometimes can also be *associative array* 
  
  - The type must be defined at schema level (`CREATE TYPE`) or in package spec
    
    - Cannot be defined within the `DECLARE` block of the function

    

###### What is A Table Function (3 / 3)

- All parameters to the function are of mode `IN` & have SQL-compatible datatypes

- Embed the call to the function inside the `TABLE` clause. For example: 
  
  ```sql
  FUNCTION f() ...
  SELECT * FROM TABLE(f());
  ```

    

###### Create Table Function

- First, let's create the nested table type to be returned by the function
  
  ```sql
  CREATE OR REPLACE TYPE strings_t IS TABLE OF VARCHAR2 (100);
  ```

- Then create a function that returns an array of random strings
  
  ```sql
  CREATE OR REPLACE FUNCTION random_strings (count_in IN INTEGER)
     RETURN strings_t
  IS
     l_strings   strings_t := strings_t ();
  BEGIN
     l_strings.EXTEND (count_in);
  
     FOR indx IN 1 .. count_in
     LOOP
        l_strings (indx) := DBMS_RANDOM.string ('u', 10);
     END LOOP;
  
     RETURN l_strings;
  END;
  ```

- Finally, demonstrate that the function works
  
  ```sql
  DECLARE
     l_strings   strings_t := random_strings (5);
  BEGIN
     FOR indx IN 1 .. l_strings.COUNT
     LOOP
        DBMS_OUTPUT.put_line (l_strings (indx));
     END LOOP;
  END;
  ```

    

###### Use Table Function in Table Clause (1 / 2)

- You can (and should) give that `TABLE` clause a table alias
  
  ```sql
  SELECT rs.COLUMN_VALUE my_string FROM TABLE (random_strings (5)) rs
  ```

- Starting in Oracle 12c, also use *named notation* when invoking the function
  
  ```sql
  SELECT COLUMN_VALUE my_string FROM 
  TABLE (random_strings (count_in => 5))
  ```

- Can also call built-in functions for the value returned by the table function
  
  ```sql
  SELECT SUM (LENGTH (COLUMN_VALUE)) total_length,
         AVG (LENGTH (COLUMN_VALUE)) average_length
    FROM TABLE (random_strings (5))
  ```

- On 12.1 and higher, don't bother with the `TABLE` clause
  
  ```sql
  SELECT rs.COLUMN_VALUE no_table FROM random_strings (5) rs
  ```

    

###### Use Table Function in Table Clause (2 / 2)

Notice that Oracle Database automatically uses the string `COLUMN_VALUE`:

- As the name of the single column returned by the table function

- Can verify this by executing the statements in the last section

    

###### Blending Table Function & In-Table Data

- Can use the *table function* same as any other dataset in a `SELECT` statement
  
  ```sql
  SELECT COLUMN_VALUE last_name FROM TABLE (random_strings (10)) rs
  UNION ALL
  SELECT e.last_name FROM hr.employees e WHERE e.department_id = 100
  ```

- Can also use it in a `IN` clause
  
  ```sql
  BEGIN
     FOR rec IN (SELECT COLUMN_VALUE str FROM TABLE (random_strs (5)))
     LOOP
        DBMS_OUTPUT.put_line (rec.str);
     END LOOP;
  END;
  ```

    

###### Left Correlations & Table Functions (1 / 3)

A *left correlation join* occurs when:

- Passing as an argument to your *table function* a column value from a table
  
  - Referenced to the left in the `TABLE` clause

- The func will be called *for each row* in the table that provides the parameter
  
  - This could cause some performance issues

    

###### Left Correlations & Table Functions (2 / 3)

```sql
CREATE TABLE things
(
   thing_id     NUMBER,
   thing_name   VARCHAR2 (100)
)
```

```sql
BEGIN
   INSERT INTO things VALUES (1, 'Thing 1');
   INSERT INTO things VALUES (2, 'Thing 2');
   COMMIT;
END;
```

```sql
CREATE OR REPLACE TYPE numbers_t IS TABLE OF NUMBER
```

    

###### Left Correlations & Table Functions (3 / 3)

```sql
CREATE OR REPLACE FUNCTION more_numbers (id_in IN NUMBER)
   RETURN numbers_t
AS 
    l_numbers numbers_t := numbers_t();
BEGIN
    l_numbers.extend(id_in * 5);
    
    FOR indx IN 1 .. id_in * 5
    LOOP
        l_numbers (indx) := indx;
    END LOOP;

    RETURN l_numbers;
END;
```

```sql
BEGIN
    FOR rec in (SELECT th.thingName, t.COLUMN_VALUE thing_num
                FROM things th, TABLE(more_numbers(th.thing_id)) t)
    LOOP
        DBMS_OUTPUT.put_line ('more numbers ' || rec.thing_number);
    END LOOP;
END;
```

    

###### Where Table Functions Can be Defined (1 / 2)

Can be at schema level or package spec, not in a nested or private subprogram

```sql
DECLARE
   FUNCTION nested_strings (count_in IN INTEGER) RETURN strings_t
   IS
   BEGIN
      RETURN strings_t ('abc');
   END;
BEGIN
   FOR rec IN (SELECT * FROM TABLE (nested_strings()))
   LOOP
      DBMS_OUTPUT.PUT_LINE (rec.COLUMN_VALUE);
   END LOOP;
END;
```

`PLS-00231: function 'NESTED_STRINGS' may not be used in SQL`

    

###### Where Table Functions Can be Defined (2 / 2)

New to 12.1, can use the `WITH` clause to define functions inside a `SELECT`

```sql
WITH 
  FUNCTION strings RETURN strings_t 
  IS 
  BEGIN 
     RETURN strings_t ('abc'); 
  END; 
SELECT COLUMN_VALUE my_string FROM TABLE (strings)
```

    

###### Valid Collection Types for Table Functions (1 / 2)

2 things to notice about the collection types used in `RETURN` clause of a *table function*

- It must be declared so that the SQL engine can resolve a reference to it

- It must be SQL-compatible. You cannot, e.g., return a collection of `Boolean`s

    

###### Valid Collection Types for Table Functions (2 / 2)

Types defined within a package spec can only be used with *pipelined table function*s 

    

###### Returning Multiple Columns

- We've shown how to query from a *table function* that returns a collection of scalars

- However, sometimes you need to pass back rows consisting of more than 1 value

    

###### Just Use %ROWTYPE? (1 / )

You may use the `%ROWTYPE` as an attribute for the nested table type

- This won't work. say we want our *table function* to return rows to fit in this table
  
  ```sql
  CREATE TABLE animals
  (
     name VARCHAR2 (10),
     species VARCHAR2 (20),
     date_of_birth DATE
  )
  ```

- The most straightforward way for would be to do something like this:
  
  ```sql
  CREATE TYPE animals_nt IS TABLE OF animals%ROWTYPE;
  CREATE OR REPLACE FUNCTION lots_of_animals RETURN animals_nt;
  ```

    

###### Just Use %ROWTYPE? (2 / 2)

*PL/SQL* is a language that offers procedural *extensions* to *SQL*

- *PL/SQL* knows all bout *SQL*, but *SQL* doesn't know *PL/SQL*-specific constructs

- `%ROWTYPE` is not a part of *SQL* & `CREATE TYPE` statement is a *SQL* statement

- So what's a developer supposed to do? Use object types!

    

###### Object Type Mimics Table (1 / 2)

- Your *table function*'s collection type must be an object type whose attributes:
  
  - Look like the columns of the dataset you want

- First we create an object type with attributes that match the table
  
  ```sql
  CREATE TYPE animal_ot IS OBJECT
  (
     name VARCHAR2 (10),
     species VARCHAR2 (20),
     date_of_birth DATE
  );
  ```

- Then we create a nested table of this type:
  
  ```sql
  CREATE TYPE animals_nt IS TABLE OF animal_ot;
  ```

    

###### Object Type Mimics Table (2 / 2)

In the code below, we define a function accepting 2 object types & return a collection

```sql
CREATE OR REPLACE FUNCTION animal_fam(dad_in IN animal_ot, mom_in IN ...)
RETURN animals_nt
AS
    l_family animals_nt := animals_nt(dad_in, mom_in);
BEGIN
    FOR indx in 1 .. 10
    LOOP
        l_family.extend;
        l_family(l_family.last) := animal_ot('BABY' || indx, 
            mom_in.species, 
            ADD_MONTHS (SYSDATE, -1 * DBMS_RANDOM.VALUE (1, 6)));
    END LOOP;
    return l_family;
END;
```

    

###### Use the Table Function

- In the `SELECT`, can reference names of the attributes as names of the columns
  
  ```sql
  SELECT name, species, date_of_birth
  FROM TABLE (
      animal_family (animal_ot ('Hoppy', 'RABBIT', SYSDATE - 500),
                     animal_ot ('Hippy', 'RABBIT', SYSDATE - 300)))
  ```

- Here's an example of taking the result from the function & inserted into the table
  
      SELECT name, species, date_of_birth
      FROM TABLE (
          animal_family (animal_ot ('Bob',   'KANGAROO', SYSDATE - 1000),
                         animal_ot ('Sally', 'KANGAROO', SYSDATE - 700)))
