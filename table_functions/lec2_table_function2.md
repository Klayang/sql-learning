###### Returning Multiple Columns

- We've shown how to query from a *table function* that returns a collection of scalars

- However, sometimes you need to pass back rows consisting of more than 1 value

    

###### Just Use %ROWTYPE? (1 / 2)

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
  
  ```sql
  SELECT name, species, date_of_birth
  FROM TABLE (
      animal_family (animal_ot ('Bob',   'KANGAROO', SYSDATE - 1000),
                     animal_ot ('Sally', 'KANGAROO', SYSDATE - 700)))
  ```
