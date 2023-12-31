###### Defining Collection Types (1 / )

- Before you can manipulate a collection variable, you need a collection type
  
  - On which to declare the variable

- Oracle pre-defines several collection types in various supplied packages
  
  - `DBMS_SQL`: dynamic sql-specific types, generic types (strings, nums, etc)
  
  - `DBMS_OUTPUT`: list of strings

    

###### Defining Collection Types (2 / )

Declaration of all collections starts with the `TYPE` statement

- Associative arrays:
  
  ```sql
  TYPE coll_name IS TABLE OF element_type INDEX BY index_type;
  ```

- Nested table:
  
  ```sql
  TYPE coll_name IS TABLE OF element_type;
  ```

- Varray:
  
  ```sql
  TYPE coll_name IS VARRAY(limit) OF element_type;
  ```

    

###### Scope for Collection Types

- You can define the collection types in:
  
  - Local block - can be used only in that block
  
  - Package - available for use by any session with authority on that package
  
  - Schema - in the *SQL* layer, only for *nested table*s & *varray*s

- Avoid *reinventing* collection types in many places of your code
  
  - They're excellent candidates for shared code elements

    

###### Local Collection Types

```sql
DECLARE 
TYPE strings_t IS TABLE OF VARCHAR2(100);
```

```sql
PROCEDURE my_proc IS
TYPE strings_t IS TABLE OF VARCHAR2(100);
```

- `TYPE` statement in declaration section of a block (anoymous, nested, subprogram)

- It's defined & destroyed each time the block is executed

- You should avoid local types, for its likelihood to cause redundancy in your code

    

###### Package Level Types

```sql
PACKAGE my_types IS
TYPE strings_t IS TABLE OF VARCHAR2(100);
```

- When defined in a package specification, it becomes "globally" available
  
  - Any session with `EXECUTE` authority on the package can use it

- If it's defined in the package body, can only be used by: 
  
  - Subprograms of the pakcage

    

###### Schema Level Types

```sql
CRAETE OR REPLACE TYPE strings_t IS TABLE OF VARCHAR2(100);
GRANT EXECUTE ON strings_t TO PUBLIC;
```

- Defined in the schema, independent of any `PL/SQL` program unit

- Collections of this type can be directly referenced inside *SQL* statements

- Can only be used with *nested table*s & *varray*s

    

###### Declaring Collection Variables

Once you've defined the type, can declare a variable of that type

```sql
CREATE OR REPLACE TYPE hire_dates_t IS TABLE OF DATE;
```

```sql
CREATE OR REPLACE PACKAGE my_types IS 
    TYPE strings_t IS TABLE OF VARCHAR2(100);
END my_types;
```

```sql
DECLARE
l_names my_types.strings_t;
l_dates hire_dates_t;
l_dates HR.hire_dates_t;
l_strings DBMS_SQL.varchar2_table;
```

    

###### Conclusions

- Declare new collection types with the keywork `TYPE`
  
  - We'll explore each of the 3 types in detail later

- You can declare a type at the pakacge level, or the schema level
  
  - Best to avoid locally-declared types

    




