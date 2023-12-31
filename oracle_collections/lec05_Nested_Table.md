###### What is A Nested Table?

    CREATE OR REPLACE TYPE list_of_names_t IS TABLE OF VARCHAR2(50);

A *nested table* is a type of collection, which models *an unordered set* of elements

- It's a *multiset*: no inherent order to its elements, duplicates are allowed

- From a practical standpoint, can still access the elements through an `int` index

    

###### Overview of Nested Tables

- The *nested table* was added in Oracle 8 as part of the object model

- A *nested table* type can be defined in *PL/SQL* or a schema-level type (for use in *SQL*)

- Must initialize a *nested table* before using it, and `extend` to make room: 
  
  - For new elements

    

###### Initializing Nested Tables (1 / 2)

Before you can use a *nested table*, it must be initialized

- Initialize it explicitly with the *constructor*, same name as type

- The *constructor* is privided by Oracle and cannot be overloaded

- No need to initialize if it's populated by `BULK COLLECT` query

    

###### Initializing Nested Tables (2 / 2)

Provide a list of values or initialize it as empty

```plsql
DECLARE
    TYPES numbers_t IS TABLE OF number;
    salaries numbers_t := numbers_t();
```

```plsql
DECLARE
    TYPE numbers_t IS TABLE OF number;
    salaries numbers_t := numbers_t(100, 200, 300); -- 3 elements
```

```plsql
DECLARE
    TYPE numbers_t IS TABLE OF number;
    salaries numbers_t;
BEGIN
    salaries := numbers_t();
```

    

###### Characteristics of Nested Tables

- No pre-defined or practical limit on # elements in a *nested table*
  
  - Index values can range from 1 to 2<sup>31</sup> - 1

- Always dense initially, but can become sparse after `DELETE`s

- Can be defined as a schema-level type, and used as a relational table column type

- Multiset operators allow set-level operations on *nested table*s

    

###### Happy Family Example (1 / 2)

```sql
CREATE OR REPLACE TYPE list_of_names_t IS TABLE OF VARCHAR2(50);
```

```sql
DECLARE
    happy_family list_of_names_t := list_of_names_t();
    parents list_of_names_t := list_of_names_t();
    children list_of_names_t := list_of_names_t();
BEGIN
    happy_family.EXTEND(3);
    happy_family(1) := 'Eli';
    happy_family(2) := 'Steven';
    happy_family(3) := 'Chris';
    --
    children.EXTEND;
    children(children.LAST) = 'Chris';
    --
    parents := happy_family MULTISET EXCEPT children;
    FOR l_row IN 1 .. parents.COUNT
    LOOP
        dbms_output.put_line(parents(l_row));
    END LOOP;
END; 
```

    

###### Happy Family Example (2 / 2)

```sql
...
FOR rec IN (SELECT COLUMN_VALUE family_name FROM TABLE(happy_family))
LOOP
    dbms_output.put_line(rec.family_name);
END LOOP;
```

    

###### Nested Table as Column Type

    CREATE TABLE FAMILY
    (
        surname VARCHAR2(50),
        parent_names parent_names_t,
        children_names children_names_t
    );
    NESTED TABLE children_names STORED AS parent_names _tbl;
    NESTED TABLE parent_names STORED AS children_names_tbl;

- If the type is defined at schema level, it can be used as a column type

- Must also provide a `STORE AS` clause

- The order of the elements in the column is not preserved

- See more detail in next lesson `using collections in SQL`
