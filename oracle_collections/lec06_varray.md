###### What is a Varray?

```plsql
CREATE OR REPLACE TYPE list_of_names_t IS VARRAY(5) OF NUMBER;
```

- A `VARRAY` (*variable size array*) is a type of collection with an upper bound on its size

- The upper limit is set when the type is defined, but can also be adjusted at runtime

- Other than that, quite similar to *nested table* (initialize, extend, etc)

    

###### Characteristics of Varray

- Trying to extend the upper limit of the `VARRAY` would cause errors

- `VARRAY`s are always dense. Cannot `DELETE` but can only `TRIM` from the `VARRAY`

- Can be defined as a schema-level type and used as a relational type column type

    

###### Varray as Column Type

`VARRAY`s can also serve up as column in a table. Unlike a *nested table*:

- You do not provide a `STORE AS` clause

- The order of elements in the column is preserved

    

###### Change Upper Limit on Varray

Can change the upper limit of an `VARRAY` at runtime with an `ALTER TYPE`

```plsql
ALTER TYPE my_varray_t MODIFY LIMIT 100 INVALIDATE
```

```plsql
BEGIN
    EXECUTE IMMEDIATE 'ALTER TYPE my_varray_t MODIFY LIMIT 100 CASCADE';
END;
```

Note the change only takes effect when the current block executes and the new starts

    


