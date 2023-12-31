###### Introduction (1 / 2)

*Pipelined table functions* are *table functions* that:

- Return or pipe rows back to the calling query

- As the function is producing the data in the desired the form

- And before the function has completed all processing

    

###### Introduction (2 / 2)

Let's first analize how unusual the above statement is

- PL/SQL is not a multi-threaded language. When an PL/SQL block is invoked

- Further processing is *on hold* (suspended), until the block returns control
  
  - To the host that invoked the block

- Non-pipelined table functions act in this way. When it's invoked, must wait
  
  - Until a `RETURN` statement is executed to pass back the collection

- The blocking behavior has a negative impact on performance of the `SELECT`
  
  - For very large datasets, this can lead to even errors

    

###### A Very Simple Example (1 / 3)

```sql
CREATE OR REPLACE TYPE strings_t IS TABLE OF VARCHAR2 (100);
```

```sql
CREATE OR REPLACE FUNCTION strings 
   RETURN strings_t PIPELINED
IS
BEGIN
   PIPE ROW ('abc');
   RETURN;
END;
```

```sql
SELECT COLUMN_VALUE my_string FROM TABLE (strings());
```

```sql
SELECT COLUMN_VALUE my_string FROM TABLE (strings());
```

    

###### A Very Simple Example (2 / 3)

What about executing this function in `PL/SQL`?

```sql
DECLARE
    l_strings strings_t := strings_t();
BEGIN
    l_strings := strings();
END;
```

- You'll see this error:
  
  ![](C:\Users\yangs\AppData\Roaming\marktext\images\2023-12-22-08-58-28-image.png)

- This makes a lot of sense. PL/SQL is not a multi-threaded language. It cannot:
  
  - Accept rows *piped* back, before the function terminates execution

    

###### A Very Simple Example (3 / 3)

- Unlike regular *table function*s, when you go with `PIPELINED`:
  
  - You give up the ability to call the function in PL/SQL

- The example above gets across the basic elements of a pipelined table function:
  
  - Add the `PIPELINED` keyword to the function header
  
  - Use `PIPE ROW` to send the data back to the calling `SELECT`
    
    - Instead of adding the data to a local collection
  
  - `RETURN` nothing but control

    

###### Impact of Switch to Pipelined Table Functions (1 / 2)

- Pipelined table functions can help improve performance over the non-pipelined
  
  - And reduce PGA memory consumption

- Remember `doubled` in lec3? We'll create a *pipelined* version of it here
  
  ```sql
  CREATE OR REPLACE FUNCTION doubled_pl(rows_in stock_pkg.stocks_rc)
  RETURN tickets_nt PIPELINED AS
      TYPE stocks_aat IS TABLE OF stocks%ROWTYPE INDEX BY PLS_INTEGER;
      l_stocks stocks_aat;
  BEGIN
      LOOP
          FETCH rows_in BULK COLLECT INTO l_stocks LIMIT 100;
          EXIT WHEN l_stocks.COUNT = 0;
          
          FOR l_row IN 1 .. l_stocks.COUNT
          LOOP
              PIPE ROW(ticker_ot(l_stocks(l_row).ticker, 
                                 l_stocks (l_row).trade_date), 
                                 'O', 
                                 l_stocks (l_row).open_price));
              PIPE ROW(ticker_ot(l_stocks (l_row).ticker,
                                l_stocks (l_row).trade_date,
                                'C',
                                l_stocks (l_row).close_price));
          END LOOP;
      END LOOP;
      RETURN;
  END;
  ```

    

###### Impact of Switch to Pipelined Table Functions (2 / 2)

Let's verify this `PIPELINED` *table function* can be used in a `SELECT`

```sql
INSERT INTO tickers
SELECT * FROM TABLE(doubled_pl(CURSOR(SELECT * FROM stocks)));
```

    

###### The No_Data_Needed Exception (1 / 5)

You may want to terminate the `PIPELINED` *table function* before all rows piped pack

- Oracle will raise `NO_DATA_NEEDED` exception, which will terminate the function
  
  - But will not terminate the `SELECT` that called it

- You do need to explicitly handle this exception if either of the following applies:
  
  - You include an `OTHERS` exception handler in a block including a `PIPE ROW`
  
  - Your code that feeds a `PIPE ROW` statement must be followed by clean-ups
    
    - Typically, the clean-up releases resources that the code no longer needs

    

###### The No_Data_Needed Exception (2 / 5)

Let's explore this in more detail. In this section, 2 rows are piped, but only 1 needed

```sql
CREATE OR REPLACE TYPE strings_t IS TABLE OF VARCHAR2 (100);
```

```sql
CREATE OR REPLACE FUNCTION strings RETURN strings_t PIPELINED IS
BEGIN
   PIPE ROW (1);
   PIPE ROW (2);
   RETURN;
END;
```

```sql
SELECT COLUMN_VALUE my_string
FROM TABLE (strings ())
WHERE ROWNUM < 2
```

    

###### The No_Data_Needed Exception (3 / 5)

Now if we add an `OTHERS` exception:

```sql
CREATE OR REPLACE FUNCTION strings RETURN strings_t PIPELINED IS
BEGIN
   PIPE ROW (1);
   PIPE ROW (2);
   RETURN;
EXCEPTION
   WHEN OTHERS
   THEN
      DBMS_OUTPUT.put_line ('Error: ' || SQLERRM);
      RAISE;
END;
```

- As you can see, the `NO_DATA_NEEDED` error is trapped by that handler
  
  - And the re-raise does not manifest as an error in the `SELECT`

- Problem: the `OTHERS` handler may contain specific cleanup for other failures
  
  - But not for an early termination of data piping

    

###### The No_Data_Needed Exception (4 / 5)

The recommendation is to provide a specific handler for `NO_DATA_NEEDED`

```sql
CREATE OR REPLACE FUNCTION strings RETURN strings_t PIPELINED IS
BEGIN
   PIPE ROW (1);
   PIPE ROW (2);
   RETURN;
EXCEPTION
   WHEN no_data_needed
   THEN
      RAISE;
   WHEN OTHERS
   THEN
      /* Clean up code here! */
      RAISE;
END;
```

    

###### The No_Data_Needed Exception (5 / 5)

The basic takeaway regarding `NO_DATA_NEEDED` is: 

- Don't worry about it, unless you are providing a `WHEN OTHERS` handler: 
  
  - In your `PIPELINED` table function

- In that case, make sure to provide a handler for `NO_DATA_NEEDED`, in which: 
  
  - You will simply re-raise the exception with a `RAISE;` statement

    

###### Package-based Types Implicitly Declared (1 / 2)

With `PIPELINED` *table function*s only:

- Your types can be defined in the spec of a package, as opposed to in the schema

- You can even declare your nested table as a collection of record types!

    

###### Package-based Types Implicitly Declared (2 / 2)

```sql
CREATE TABLE stocks2
(
   ticker        VARCHAR2 (20),
   trade_date    DATE,
   open_price    NUMBER,
   close_price   NUMBER
)
```

```sql
CREATE OR REPLACE PACKAGE pkg
AS
   TYPE stocks_nt IS TABLE OF stocks2%ROWTYPE;

   FUNCTION stock_rows
      RETURN stocks_nt
      PIPELINED;
END;
```

```sql
CREATE OR REPLACE PACKAGE BODY pkg AS
   FUNCTION stock_rows RETURN stocks_nt PIPELINED IS
      l_stock   stocks2%ROWTYPE;
   BEGIN
      l_stock.ticker := 'ORCL';
      l_stock.open_price := 100;
      PIPE ROW (l_stock);
      RETURN;
   END;
END;
```

```sql
SELECT ticker, open_price FROM TABLE (pkg.stock_rows ())
```

    
