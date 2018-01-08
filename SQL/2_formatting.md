# Formatting
## Whitespace and Symbols
### Indenting and Spacing
- An indent is two spaces.
- Use a single space after a comma (e.g. between function arguments).
- There is no space after the leading parenthesis/bracket or before the closing parenthesis/bracket.
- There is no space between a function name and the leading parenthesis, but there is a space before the leading parenthesis of a flow-control statement.
- Use a single space to surround all infix operators.
- Spaces should be used to line up the code so that the root keywords all end on the same character boundary. This forms a river down the middle making it easy for the readers eye to scan over the code and separate the keywords from the implementation detail. Rivers are bad in typography, but helpful here.
``` sql
/* Bad - No river used so visibility is harder. */
SELECT COUNT(*) AS Expenses, AVG(net) AS net, AVG(vat) AS vat, AVG(total) AS total FROM savedexpenses
WHERE staff > 0 OR others > 0 GROUP BY staff, others ORDER BY Expenses DESC


/* Good - River is used to increase visibility. */ 
SELECT COUNT(*) AS Expenses
	,AVG(net) AS net
	,AVG(vat) AS vat
	,AVG(total) AS total
FROM savedexpenses
WHERE staff > 0
	OR others > 0
GROUP BY staff
	,others
ORDER BY Expenses DESC
```
#### Joins
Joins should be indented to the other side of the river and grouped with a new line where necessary.
``` sql 
SELECT r.lastName
  FROM riders AS r
       INNER JOIN bikes AS b
       ON r.bikeVinNumber = b.vinNumber
          AND b.engines > 2

       INNER JOIN crew AS c
       ON r.crewChiefLastName = c.lastName
          AND c.chief = 1;
```
#### Subqueries
Subqueries should also be aligned to the right side of the river and then laid out using the same style as any other query. Sometimes it will make sense to have the closing parenthesis on a new line at the same character position as its opening partner—this is especially true where you have nested subqueries.

``` sql
SELECT r.lastName,
       (SELECT MAX(YEAR(championshipDate))
          FROM champions AS c
         WHERE c.lastName = r.lastName
           AND c.confirmed = 'Y') AS lastChampionshipYear
  FROM riders AS r
 WHERE r.lastName IN
       (SELECT c.lastName
          FROM champions AS c
         WHERE YEAR(championshipDate) > '2008'
           AND c.confirmed = 'Y');
```
### Blank Lines

``` 
TODO - transaction/block/script - it needs to be more precise
```
In the following list, the phrase “surrounding SQL” refers to a line consisting of more than just the begining or ending of an SQL transaction. That is, no new line is required at the beginning or end of an SQL transaction. Always place an empty line in the following places:


- Before operators (AND, OR, NOT).
- After semicolons to separate queries for easier reading
after each keyword definition
- After a comma when separating multiple columns into logical groups to separate code into related sections, which helps to ease the readability of large chunks of code.
- Keeping all the keywords aligned to the righthand side and the values left aligned creates a uniform gap down the middle of query. It makes it much easier to scan the query definition over quickly too.

``` sql
/* Example 1 */
INSERT INTO albums (title, releaseDate, recordingDate)
VALUES ('Charcoal Lane', '1990-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000'),
       ('The New Danger', '2008-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000');

/* Example 2 */
UPDATE albums
   SET releaseDate = '1990-01-01 01:01:01.00000'
 WHERE title = 'The New Danger';
 
/* Example 3 */
SELECT a.title,
       a.releaseDate, a.recordingDate, a.productionDate -- grouped dates together
  FROM albums AS a
 WHERE a.title = 'Charcoal Lane'
    OR a.title = 'The New Danger';
```


