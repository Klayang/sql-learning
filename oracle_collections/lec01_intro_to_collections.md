###### What is A Collection?

- A collection is *an ordered group of elements, all of the same type*

- It's similar to single-dimensional arrays in other programming languages
  
  - With lots of subtle differences as well

    

###### Why Use Collections?

- Generally, to manipulate lists of information (why not just use relational tables?)
  
  - Collection manipulation is much faster than modifying tables with SQL

- Collections enable other features of `PL/SQL`
  
  - e.g., `BULK LOAD` & `FORALL` use them to boost performance of multi-row sql
  
  - Serve up complex datasets of information to non-`PL/SQL` environment
    
    - By using *table function*s

    

###### What Makes Collections So Fast?

- There are generally 2 types of memory for oracle data:
  
  - System or instance-level memory (`SGA`)
  
  - Process or session-level memory (`PGA`)

- Memory for collections is allocated from the `PGA` (*Process Global Area*)
  
  - Accessing `PGA` memory is quicker than doing that to `SGA`

- Collections represent a clear tradeoff: use more memory (per session) 
  
  - To improve performance (i.e., reduce CPU time)
  
  - But definitely should take an eye on your `PGA` memory consumption

    

###### Different Types of Collections

- 3 types of collections: *associative array*, *nested table*, *varray* (varying arrays)

- *Associative array* is a `PL/SQL`-only datatype

- *Nested table*s & *varray*s can be used within `PL/SQL` blocks
  
  - And also from within the `SQL` layer

    

###### Glossary of Terms

Helpful to be familiar with the following terms

- `Element`: a collection is made up of 1 or more elements, all of the same type
  
  - Also referred to as `row`

- `Index value`: the *location* in the collection in which an *element* is found
  
  - Also referred as a `row number`
  
  - Can be an integer, or string (associative arrays only)

- `Dense` vs. `Sparse`: if every index value between the lowest and highest
  
  - Has a defined *element* (the *sparse* one would have *gap*s)


