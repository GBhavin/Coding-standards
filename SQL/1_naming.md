# Naming

## Characters

* Names contain only alphabetic characters.
* Numbers are allowed only for local variables in tests and then only as a suffix.

## Words

* Use **US-English** (e.g. “color” rather than “colour”).
* Use **English grammar** (e.g. use `ImportableDatabase` instead of `DatabaseImportable`).
* Use only **standard abbreviations** (e.g. “XML” or “SQL”).
* Use **correct capitalization**. If a word is not hyphenated, then it does not need a capital letter in the camel- or Pascal-cased form. For example, “metadata” is written as `Metadata` in Pascal-case, not `MetaData`.
* Use **number names** instead of numbers (e.g. `partTwo` instead of `part2`). The exception been when using standard acronyms known within our domain (e.g. `X509Certificate` or `P11d`).
* Do not use Hungarian notation or any other prefixing notation to "group" types or members.
* Use **whole words** or stick to accepted short forms (e.g. you may use `max` for `maximum` but prefer the suffix `Count` to the prefix `num` or `no`). The following example shows how important naming is to understand a columns purpose. 

``` sql
    SELECT currefnum FROM employees
```

## Semantics

* A name must be semantically **meaningful** in its **scope**.
* A name should be as **short** as possible and always less than 30 characters.
* A name communicates **intent**; prefer `UpdatesAutomatically` to `AutoUpdate`.
* A name reflects **semantics**, not storage details; prefer `InterestRate` to `DecimalRate` and `Rate`.
* A name should include explicit **units** where possible. 

## Case

* Pascal-case capitalizes every word in a name.
* Camel-case capitalizes all but the first word in a name.
* **Acronyms** longer than two letters are in Pascal-case (e.g. `Xml` or `Sql`). Acronyms at the beginning of a camel-case name are always all lowercase (e.g. `html`).
* Keywords should be uppercase.
* Schema objects should be camel-case.

| Object | Case |
| --- | --- |
| Keywords | Uppercase |
| Stored Procedures | Pascal |
| Functions | Pascal |
| User-Defined Types | Pascal |
| Tables | Pascal |
| Columns | Pascal |

``` sql
/* Bad */
select title, forename, surname, mileagetotal from employees
inner join employees as linemanager on (linemanager.employeeid=employees.employeeid) 
and employees.employeeid=@employeeId;


/* Good */
SELECT title
	,forename
	,surname
	,mileageTotal
FROM employees
INNER JOIN employees AS lineManager ON (lineManager.employeeId = employees.employeeId)
	AND employees.employeeId = @employeeId;    
```
## Structure
### Tables
- Should be named after their content.
- Use a collective name.
- `TODO - Avoid, where possible, concatenating two table names together to create the name of a relationship table. Rather than cars_mechanics prefer services.`

### Columns
- Always use the singular name.
- Where possible avoid simply using id as the primary identifier for the table.
- Do not add a column with the same name as its table and vice versa.

### Aliasing and Correlations
- Should relate to the object or expression they are aliasing.
- As a rule of thumb the correlation name should be the first letter of each word in the object’s name.
- If there is already a correlation with the same name then append a number.
- Always include the AS keyword as this makes it easier to read as it is explicit.
- For computed data (SUM() or AVG()) use the name you would give it were it a column defined in the schema.

``` sql
  /* Alias on columns and joins */
  SELECT firstName AS fn
  FROM employees AS e1
  JOIN employees AS e2 (ON e2.employeeId = e1.lineManager);
    
    
  /* Alias on functions */
  SELECT SUM(totalMiles) AS totalMiles
  FROM expenseItems;
```

### Stored Procedures and Functions
- Must be prefixed with a verb.

### Uniform suffixes
- The following suffixes have a universal meaning ensuring the columns can be read and understood easily from SQL code. Use the correct suffix where appropriate.

- Id — a unique identifier such as a column that is a primary key.
- Status — flag value or some other status of any type such as archivedStatus.
- Total — the total or sum of a collection of values.
- Num — denotes the field contains any kind of number.
- Name — signifies a name such as firstName.
- Seq — contains a contiguous sequence of values.
- Date — denotes a column that contains the date of something.
- Count — a count.
- Addr — an address for the record could be physical or intangible such as ipAddr.

### Reserved Keywords
- It is best to avoid the abbreviated keywords and use the full length ones where available (prefer ABSOLUTE to ABS).
- Do not use database server specific keywords where an ANSI SQL keyword already exists performing the same function. This helps to make code more portable.
