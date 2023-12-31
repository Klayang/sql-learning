###### Introduction (1 / 3)

Here's an example of the use of a streaming table function:

```sql
INSERT INTO tickers
SELECT * FROM TABLE(doubled(CURSOR(SELECT * FROM stocks)))
```

    

###### Introduction (2 / 3)

What's going on here? Let's take it step by step

- `SELECT * FROM stocks`: Get all the rows from the table `stocks`

- `CURSOR ()`: Create a cursor variable that points to the result set
  
  - Pass that cursor variable to the table function `doubled`

- `doubled ()`: The table function performs its transformation 
  
  - And returns a nested table of object type instances

- `SELECT * FROM TABLE(...)`: Convert the collection into a set of rows

- `INSERT INTO tickers`: Insert those rows into the tickers table

    

###### Introduction (3 / 3)

You may perform more than 1 transformation as part of the streaming  process

- No probelm, can certainly *string together* multiple invocations of table functions

- All the code to implement and demonstrate this statement follows in this tutorial
  
  <img src="file:///C:/Users/yangs/AppData/Roaming/marktext/images/2023-12-22-06-49-09-image.png" title="" alt="" width="510">

    

###### Setting up Tables for Transformation (1 / 4)

We will start with the table `stocks`

```sql
CREATE TABLE stocks
(
   ticker        VARCHAR2 (20),
   trade_date    DATE,
   open_price    NUMBER,
   close_price   NUMBER
)
```

    

###### Setting up Tables for Transformation (2 / 4)

Let's load it up with some optimistic data

```sql
BEGIN
   FOR indx IN 1 .. 1000
   LOOP
      INSERT INTO stocks VALUES ('STK'||indx, SYSDATE, indx, indx + 15);
   END LOOP;
   COMMIT;
END;
```

    

###### Setting up Tables for Transformation (3 / 4)

Transformation: for each row in the table `stock`, generate 2 rows: 

For the table `tickers` (one row each for the open and close prices)

```sql
CREATE TABLE tickers
(
   ticker      VARCHAR2 (20),
   pricedate   DATE,
   pricetype   VARCHAR2 (1),
   price       NUMBER
)
```

    

###### Setting up Tables for Transformation (4 / 4)

Can use `insert all` to copy data from `stocks` to `tickers`

```sql
INSERT ALL
   INTO tickers (ticker, pricedate, pricetype, price) 
        values (ticker, trade_date, 'O', open_price)
   INTO tickers (ticker, pricedate, pricetype, price) 
        values (ticker, trade_date, 'C', close_price)
SELECT * FROM stocks
```

However, in this tutorial, we'll use table function to implement it

    

###### Types & Package for Table Function (1 / 2)

First we need an *object type* that looks like the table `tickers`

```sql
CREATE TYPE ticker_ot AUTHID DEFINER IS OBJECT 
(
   ticker VARCHAR2 (20),
   pricedate DATE,
   pricetype VARCHAR2 (1),
   price NUMBER
);
```

```sql
CREATE TYPE tickers_nt AS TABLE OF ticker_ot;
```

    

###### Types & Package for Table Function (2 / 2)

Since we'll use the *table function* in a streaming process

- Also need to define a strong `REF CURSOR` type that will be used as:

- The datatype of the parameter accepting the dataset inside the SQL
  
  ```sql
  CREATE OR REPLACE PACKAGE stock_pkg AUTHID DEFINER
  IS
     TYPE stocks_rc IS REF CURSOR RETURN stocks%ROWTYPE;
     TYPE tickers_rc IS REF CURSOR RETURN tickers%ROWTYPE;
  END stock_pkg;
  ```

    

###### Define the Table Function (1 / 4)

The main distinction with *streaming* & regular table functions is that: 

- At least one parameter to that function is a cursor variable

- Generally, the flow within a streaming table function is:
  
  - Fetch a row from the cursor variable
  
  - Apply the transformation to each row
  
  - Put the transformed data into the collection
  
  - Return the collection when done

    

###### Define the Table Function (2 / 4)

Now let's see how this pattern unfolds in the `doubled` function here

```sql
CREATE OR REPLACE FUNCTION doubled (rows_in IN stock_pkg.stocks_rc)
   RETURN tickers_nt
IS
   TYPE stocks_nt IS TABLE OF stocks%ROWTYPE INDEX BY PLS_INTEGER;
   l_stocks    stocks_nt;
   l_doubled   tickers_nt := tickers_nt ();
BEGIN
   LOOP
      FETCH rows_in BULK COLLECT INTO l_stocks LIMIT 100;
      EXIT WHEN l_stocks.COUNT = 0;

      FOR l_row IN 1 .. l_stocks.COUNT
      LOOP
         l_doubled.EXTEND;
         l_doubled (l_doubled.LAST) :=
            ticker_ot (l_stocks (l_row).ticker,
                       l_stocks (l_row).trade_date,
                       'O',
                       l_stocks (l_row).open_price);

         l_doubled.EXTEND;
         l_doubled (l_doubled.LAST) :=
            ticker_ot (l_stocks (l_row).ticker,
                       l_stocks (l_row).trade_date,
                       'C',
                       l_stocks (l_row).close_price);
      END LOOP;
   END LOOP;
   CLOSE rows_in;

   RETURN l_doubled;
END;
```

    

###### Define the Table Function (3 / 4)

Here's the explanation of the streaming *table function* above:

- Line 1: Use the `REF CURSOR` type defined in the package for the rows passed in
  
  - Since we are selecting from table `stocks`, we use type `stocks_rc`

- Line 2: Return an array, each of whose elements looks like a row in table `tickers`

- Line 5: Declare an associative array to hold rows fetched from `rows_in` cursor

- Line 6: Define an associative array to hold results that will be returned

- Line 8: Start up a simple loop to fetch rows from the cursor. It's already open:
  
  - The `CURSOR()` expression takes care of that

- Lines 9: Use `BULK COLLECT` feature to retrieve up to 100 rows with each fetch
  
  - Do this to avoid row-by-row processing, which is not efficient enough
  
  - Exit the loop when the associative array is empty

- Line 12: For each element in the array (row from the `cursor` variable)....

- Lines 14-26: Use `EXTEND` to add elements at the end of the nested table 
  
  - Then call the object type constructor to create 2 new rows
  
  - And put them in the new last index value of the collection

- Line 31: Close the `cursor` variable, now that all rows have been fetched
  
  - Note: this is optional. When using `CURSOR()` to pass in the result set:
  
  - The `cursor` will be closed automatically when the function terminates

- Line 33: Send the nested table back to the `SELECT` statement for streaming

    

###### Define the Table Function (4 / 4)

Regarding `FETCH-BULK` `LIMIT`, here we use 100

- If you are processing a number of rows & want to squeeze better performance 
  
  - You might try a larger `LIMIT` value

- However, this will consume more *Process Global Area* memory. At some point:
  
  - Your code will slow down due to excessive memory consumption

- Can pass `LIMIT` as parameter to be able to modify performance-memory profile
  
  ```sql
  CREATE OR REPLACE FUNCTION doubled (
     rows_in stock_mgr.stocks_rc, limit_in IN INTEGER DEFAULT 100)
  ...
     FETCH rows_in BULK COLLECT INTO l_stocks LIMIT limit_in;
  ```

    

###### Summary

- Streaming table functions play a crucial role in data warehouse ETLs operations
  
  - *ETL* stands for *extract-transform-load*

- Oracle Database makes building such functions easy by its implementation of:
  
  - PL/SQL cursor variables and the `CURSOR()` expression

- Remember the collection constructed & returned by a streaming table function: 
  
  - Will consume `PGA` memory, so very large data sets:
  
  - Passed in to the function via the cursor variable could result in errors
