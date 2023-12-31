###### Overview

- When you define your collection at schema level, collection declared with that type:
  
  - Can be referenced in the *SQL* layer, either as a column in a table
  
  - Or by selecting from that collection in a `SELECT` statement

- This feature only applies to *nested table*s & *varray*s, since *associative array*s are:
  
  - *PL/SQL*-only structures

- Oracle offers ways to translate between a collection & a relational table format
  
  - `TABLE`: collection -> relational table
  
  - `MULTISET`: relational table -> collection

    

###### Collection Type Definition Example (1 / 2)

```sql
CREATE OR REPLACE TYPE names_t AS TABLE OF VARCHAR2(100);
```

```sql
DECLARE
    l_names names_t := names_t('Shijie', 'Xinyi', 'Hooxing');
BEGIN
    FOR rec IN (SELECT COLUMN_VALUE name FROM TABLE(l_names))
    LOOP
        dbms_output.put_line(rec.name);
    END LOOP;
END;
```

This code runs

    

###### Collection Type Definition Example (2 / 2)

```sql
DECLARE
    TYPE names_t AS TABLE OF VARCHAR2(100);
    l_names names_t := names_t('Shijie', 'Xinyi', 'Hooxing');
BEGIN
    FOR rec IN (SELECT COLUMN_VALUE name FROM TABLE(l_names))
    LOOP
        dbms_output.put_line(rec.name);
    END LOOP;
END;
```

This code fails

    

###### Using the TABLE Operator

- Use `TABLE` to work with data in a collection, as if it were data in a table
  
  - Oracle refers to this as *un-nesting*
  
  - Useful when you'd like to apply *SQL* operations to a collection

- Since Oracle 10, no need to explicitly cast the collection
  
  - Oracle'll figure out the type automatically

    

###### Changing Collection Contents with TABLE

- Can change the contents of a *nested table* column value with `TABLE`
  
  ```sql
  CREATE OR REPLACE parent_names_t IS TABLE OF VARCHAR2(50);
  CREATE OR REPLACE children_names_t IS TABLE OF VARCHAR2(50);
  ```
  
  ```sql
  CREATE TABLE FAMILY
  (
      surname VARCHAR2(50),
      parent_names parent_names_t,
      children_names children_names_t
  )
  NESTED TABLE children_names STORE AS parent_names_tbl
  NESTED TABLE parent_names STORE AS children_names_tbl 
  ```
  
  ```sql
  DECLARE
      parents parent_names_t := parent_names_t('Steven', 'Veva');
      children children_names_t := children_names_t('Chris', 'Eli');
  BEGIN
      INSERT INTO family VALUES('Yang', parents, children);
  END;
  ```
  
  ```sql
  UPDATE TABLE(SELECT children_names FROM FAMILY WHERE surname = 'Yang')
  SET COLUMN_VALUE = 'Shijie' WHERE COLUMN_VALUE = 'Eli';
  ```

- *Varray*s have to be changed *en masse* - the whole *varray* is replaced
  
  - Cannot modify individual elements

    

###### Using the MULTISET Operator

- `MULTISET` is the inverse of `TABLE`, converting a set of *table*s, into a collection

- Can be used to transform relational joins into collections (multiple values per row)
  
  ```sql
  CREATE OR REPLACE TYPE country_tab_t IS TABLE OF VARCHAR2(100);
  ```
  
  ```sql
  DECLARE
      CURSOR bird_curs IS 
      SELECT b.genus, b.species, CAST(MULTISET(
          SELECT bh.country FROM bird_habitats bh 
          WHERE bh.genus = b.genus AND bh.species = b.species
      ) AS country_tab_t) 
      FROM birds b;
      bird_row birds%ROWTYPE;
  BEGIN
      OPEN bird_curs;
      FETCH bird_curs into bird_row;
  END;
  ```

    

###### Conclusions

- A key advantage of *nested table*s & *varray*s is that they can be used in *SQL* layer

- Using `TABLE` operator allows you to apply *SQL* operations to *PL/SQL* data

- Using `MULTISET` can manipulate a result set (*table*) as if it were a collection

    
