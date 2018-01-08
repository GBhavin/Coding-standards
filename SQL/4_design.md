# Design

## Avoid
- Object oriented design principles do not effectively translate to relational database designs — avoid this pitfall.
- Placing the value in one column and the units in another column. The column should make the units self evident to prevent the requirement to combine columns again later in the application. Use CHECK() to ensure valid data is inserted into the column.
- EAV ([Entity Attribute Value](https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model)) tables — use a specialist product intended for handling such schema-less data instead.
- Splitting up data that should be in one table across many because of arbitrary concerns such as time-based archiving or location in a multi-national organisation. Later queries must then work across multiple tables with UNION rather than just simply querying one table.

## Cursors
- Using a CURSOR is less efficient (normally be a considerable margin) than using an equivelent  set or a join operation and should only be used when no alternative set or join operation is possible. If you have no alternative ensure you optimise the CURSOR where possible by using FAST_FORWARD, READ_ONLY etc.

## Sub Queries
- Use sub queries only when you cannot use a set or join operation for the same result.

## Unions
- Where possible use UNION ALL over UNION if returning duplicates in the result is acceptable.