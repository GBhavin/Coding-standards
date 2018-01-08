# Usage
## Structure
### Preferred Formalisms
- Use BETWEEN where possible instead of combining multiple statements with AND.
- Use IN() instead of multiple OR clauses.
- Where a value needs to be interpreted before leaving the database use the CASE expression. CASE statements can be nested to form more complex logical structures.
- Avoid the use of UNION clauses and temporary tables where possible. If the schema can be optimised to remove the reliance on these features then it most likely should be.

``` sql
SELECT CASE postcode
       WHEN 'BN1' THEN 'Brighton'
       WHEN 'EH1' THEN 'Edinburgh'
       END AS city
  FROM officeLocations
 WHERE country = 'United Kingdom'
   AND openingTime BETWEEN 8 AND 9
   AND postcode IN ('EH1', 'BN1', 'NN1', 'KW1')
```
### Data Types
- Where possible do not use vendor specific data types—these are not portable and may not be available in older versions of the same vendor’s software.
- Only use REAL or FLOAT types where it is strictly necessary for floating point mathematics otherwise prefer INT and DECIMAL at all times.

### Default Values
- The default value must be the same type as the column — if a column is declared a DECIMAL do not provide an INTEGER default value.
- Default values must follow the data type declaration and come before any NOT NULL statement.

### Constraints and Keys
Constraints and their subset, keys, are a very important component of any database definition. They can quickly become very difficult to read and reason about though so it is important that a standard set of guidelines are followed.

#### Choosing Keys
Deciding the column(s) that will form the keys in the definition should be a carefully considered activity as it will effect performance and data integrity.

- The key should be unique to some degree.
- Consistency in terms of data type for the value across the schema and a lower likelihood of this changing in the future.
- Can the value be validated against a standard format (such as one published by ISO)? Encouraging conformity to point 2.
- Keeping the key as simple as possible whilst not being scared to use compound keys where necessary.

It is a reasoned and considered balancing act to be performed at the definition of a database. Should requirements evolve in the future it is possible to make changes to the definitions to keep them up to date.

#### Defining Constraints
Once the keys are decided it is possible to define them in the system using constraints along with field value validation.

##### General
- Tables must have at least one key to be complete and useful.
- Constraints should be given a custom name excepting UNIQUE, PRIMARY KEY and FOREIGN KEY where the database vendor will generally supply sufficiently intelligible names automatically.

##### Layout and Order
- Specify the primary key first right after the CREATE TABLE statement.
- Constraints should be defined directly beneath the column they correspond to. Indent the constraint so that it aligns to the right of the column name.
- If it is a multi-column constraint then consider putting it as close to both column definitions as possible and where this is difficult as a last resort include them at the end of the CREATE TABLE definition.
- If it is a table level constraint that applies to the entire table then it should also appear at the end.
- Use alphabetical order where ON DELETE comes before ON UPDATE.
- If it make senses to do so align each aspect of the query on the same character position. For example all NOT NULL definitions could start at the same character position. This is not hard and fast, but it certainly makes the code much easier to scan and read.

##### Validation
- Use LIKE and Regular Expression constraints to ensure the integrity of strings where the format is known.
- Where the ultimate range of a numerical value is known it must be written as a range CHECK() to prevent incorrect values entering the database. In the least it should check that the value is greater than zero in most cases.
- CHECK() constraints should be kept in separate clauses to ease debugging.

``` sql
CREATE TABLE employees (
    PRIMARY KEY (employeeId),
    employeeId      INT       NOT NULL,
    firstName     NVARCHAR(100) NOT NULL,
    passwordMethod TINYINT       NOT NULL,
                   CONSTRAINT CK_passwordMethodRange
                   CHECK(passwordMethod >= 1 AND passwordMethod < 3)
);
``` 

