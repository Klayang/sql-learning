###### String-Indexed Associative Arrays

Even if an *associative array* is string indexed, can still call it via an `int`

```sql
DECLARE
    TYPE nums_t IS TABLE OF VARCHAR(100) INDEX BY VARCHAR(100);
    nums nums_t;
BEGIN
    nums(100) := '1 HUNDRED';
    DBMS_OUTPUT.PUT_LINE(nums(100));
    DBMS_OUTPUT.PUT_LINE(nums('100'));
END;
```

![](C:\Users\yangs\AppData\Roaming\marktext\images\2023-12-25-18-12-02-image.png)
