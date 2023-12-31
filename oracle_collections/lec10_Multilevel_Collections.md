###### Multilevel Collections

- A multilevel collection type's element is directly or indirectly: 
  
  - Another collection

- Usages for multilevel collection:
  
  - Model normalized data structures in *PL/SQL* collections
  
  - Emulate multidimensional arrays

    

###### String Tracker Example

We want to record what variable names have been used in a given file

```sql
TYPE used_aat IS TABLE OF BOOLEAN INDEX BY VARCHAR2(100); -- always true
```

```sql
TYPE list_rt IS RECORD(description VARCHAR(100), varNames used_aat);
```

```sql
TYPE list_of_names IS TABLE OF list_rt INDEX BY VARCHAR2(100);
```

```sql
g_list_of_names list_of_names;
```

```sql
CREATE OR REPLACE PROCEDURE mark_as_used(file_name IN VARCHAR2(100),
var_name IN varchar2(100)) IS
BEGIN
    g_list_of_names(file_name).varNames(var_name) = TRUE;
END;
```

    CREATE OR REPLACE FUNCTION string_in_use(file_name IN VARCHAR2(100),
    var_name IN varchar2(100)) RETURN BOOLEAN IS
    BEGIN
        IF g_list_of_names.EXISTS(file_name)
            THEN RETURN g_list_of_names(file_name).varNames.EXISTS(var_name);
        ELSE
            RETURN FALSE;
        END IF;
    END;
