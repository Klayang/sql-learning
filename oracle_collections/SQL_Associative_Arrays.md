###### Introduction to Oracle PL/SQL Associative Arrays

- *Associative arrays* are single dimensional, unbounded, sparse collections: 
  
  - Of homogeneous elements

- An *associative array* is single-dimensional, which means it has: 
  
  - A single column of data in each row

- An *associative array* is unbounded, meaning that it has a predetermined: 
  
  - Limited number of elements

- An *associative array* is sparse because its elements are not sequential
  
  - i.e., it may have gaps between elements

    

###### Declaring An Associative Array Type (1 / 3)

- An *associative array* can be indexed by numbers or characters

- Declaring it is a two-step process. First, you declare an *associative array* type
  
  - Then, you declare an variable of that type

    

###### Declaring An Associative Array Type (2 / 3)

The following shows the syntax for declaring an *associative array* type:

```sql
TYPE asso_arr_type IS TABLE OF datatype [NOT NULL]
INDEX BY index_type;
```

- The `associative_array_type` is the name of the associative array type

- The `datatype` is the data type of the elements in the array

- The `index_type` is the data type of the index used to organize the elements

- Optionally, can specify `NOT NULL` to force every element must have a value

    

###### Declaring An Associative Array Type (3 / 3)

The following declares an *associative array* of `char`s indexed by `char`s

```sql
TYPE t_capital_type IS TABLE OF VARCHAR2(100) INDEX BY VARCHAR2(50);
```

    

###### Declaring An Associative Array Variable

- After having the type, you need to declare a variable of it with the syntax:
  
  ```sql
  asso_arr_var asso_arr_type
  ```

- In our example above, it would be:
  
  ```sql
  t_capital t_capital_type
  ```

    

###### Accessing Associative Array Elements

To access an array element, can use the syntax:

```sql
asso_arr_var(index)
```

Note that `index` can be a number or a character string

    

###### Assigning Associative Array Elements

To assign a value to an element, you use the *assignment operation* (`:=`)

```sql
asso_arr_var(index) := value;
```

    

###### Oracle PL/SQL Associative Array Example

The following annoymous block shows how to declare an array and assign values

```sql
DECLARE
    -- declare an associative array type
    TYPE t_capital_type IS TABLE OF VARCHAR(100) INDEX BY VARCHAR(50);
    -- declare a variable of the type
    t_capital t_capital_type;
BEGIN
    t_capital('USA') := 'Washington, D.C.';
    t_capital('United Kingdom') := 'London';
    t_capital('Japan') := 'Tokyo';

    DBMS_OUTPUT.PUT_LINE(t_capital('USA'));
END;
```

    

###### Associative Array Method (1 / 2)

Associative arrays have methods for Manipulating elements effectively

```sql
asso_arr_var.method(paras);
```

- The method `FIRST` returns the 1st index of the array.  If an array is empty: 
  
  - The `FIRST` method returns `NULL`

- The method `NEXT(n)` returns the index that succeeds the index `n`
  
  - If `n` has no successor, then the `NEXT(n)` returns `NULL`

    

###### Associative Array Method (2 / 2)

The `FIRST` & `NEXT(n)` methods are useful in iterating over the array

```sql
t_capital_index := t_capital.FIRST;
WHILE t_capital_index IS NOT NULL
LOOP
    DBMS_OUTPUT.PUT_LINE(t_capital(t_capital_index));
    t_capital_index := t_capital.NEXT(t_capital_index);
END LOOP;
```

    

###### Putting It All Together

```sql
DECLARE
    -- declare an associative array type
    TYPE t_capital_type IS TABLE OF VARCHAR2(100) INDEX BY VARCHAR2(50);
    -- declare a variable of the t_capital_type
    t_capital t_capital_type;
    -- local variable
    l_country VARCHAR2(50);
BEGIN
    
    t_capital('USA')            := 'Washington, D.C.';
    t_capital('United Kingdom') := 'London';
    t_capital('Japan')          := 'Tokyo';
    
    l_country := t_capital.FIRST;
    
    WHILE l_country IS NOT NULL LOOP
        dbms_output.put_line('The capital of ' || l_country || ' is ' || 
            t_capital(l_country));
        l_country := t_capital.NEXT(l_country);
    END LOOP;
END;
```

![](C:\Users\yangs\AppData\Roaming\marktext\images\2023-12-22-03-57-17-image.png)

    
