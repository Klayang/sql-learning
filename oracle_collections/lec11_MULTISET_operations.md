###### Check for Equality & Inequality

Can use `=` & `<>` to compare the contents of 2 *nested table*s

```sql
DECLARE
    TYPE strings_t IS TABLE OF VARCHAR2(100);
    strings1 strings_t := strings_t('aaa', 'bbb', 'ccc');
    strings2 strings_t := strings_t('bbb', 'aaa', NULL);
BEGIN
    IF strings1 = strings2
        THEN DBMS_OUTPUT.PUT_LINE('YES');
    ELSIF strings1 <> strings2
        THEN DBMS_OUTPUT.PUT_LINE('NO');
    ELSE DBMS_OUTPUT.PUT_LINE('NULL');
    END IF;
END;
```

But watch out! `NULL`s can direct the comparison out of the 2 ways

    

###### Nested Table Duplicates - Detection & Removal

Use `SET` operator to work with distincts, and test if values in *nested table* are distinct

```sql
DECLARE
    TYPE strings_t IS TABLE OF VARCHAR2(100);
    strings1 strings_t := strings_t('aaa', 'bbb', 'aaa');
    isSet BOOLEAN;
BEGIN
    strings1 := SET(strings1);
    isSet := strings1 IS A SET;
    DBMS_OUTPUT.PUT_LINE(strings1.COUNT);
    IF isSet
        THEN DBMS_OUTPUT.PUT_LINE('TRUE');
    END IF;
END;
```

    

###### Determine If Value Is in Nested Table

- Use the `MEMBER OF` syntax to determine if a value is in the *nested table*

- The implementation in *SQL* layer is slow, but fast in *PL/SQL*

    

###### Does One Nested Table Contain Another?

The `SUBMULTISET OF` operator tests if all the elements in 1 *nested table* are in another

```sql
DECLARE
    strings1 strings_t := strings_t('a', 'b');
    strings2 strings_t := strings_t('a', 'c', 'b');
BEGIN
    IF strings1 IS SUBMULTISET OF strings2
        THEN dbms_output.put_line('TRUE');
    END IF;
END;
```

    

###### UNION 2 Nested Tables Together

Use `MULTISET UNION` to join together the contents of 2 *netsted table*s

- Duplicates are preserved unless you include `DISTINCT`, which is opposite to:
  
  - `UNION` & `UNION ALL` to tables

- The resulting collection is either empty or sequentially filled from index value 1
  
  - No need to initialize or extend the 1st

    

###### INTERSECT & EXCEPT

- Use `MULTISET INTERSECT` or `MULTISET EXCEPT` 

- Different semantics but pretty close to `UNION` syntatically

    

# 
