# Order by clause can sort by nulls first or last

Reference: https://www.postgresql.org/docs/current/queries-order.html

Postgres 12 introduced the options `NULLS FIRST` or `NULLS LAST` to the `ORDER BY` clause. This allows you to sort your data and specify whether you want the null values to appear first or last. Here are some examples to illustrate how it works in combination with the `ASC` and `DESC` options.

The query below will sort the age ascending and place any null values first. Bob's age is null, so Bob will be first in the result. Then the remaining persons will be ordered by the age from youngest to oldest. So, the full line up is Bob, followed by Charlie, Alice, and Diane.

Query:
```sql
select `id`, `name`, age from 
(
    VALUES
    (1,'Alice',45),
    (2,'Bob',null),
    (3,'Charlie',44),
    (4,'Diane',46),
)
ORDER BY age ASC NULLS FIRST
```

Output:
```
2, Bob, null
3, Charlie, 44
1, Alice, 45
4, Diane, 46
```

If the age is sorted descending instead, that is `ORDER BY age DESC NULLS FIRST`, Bob would still be first in the list, then the remaining persons would be ordered by age from oldest to youngest.

Query:
```sql
select `id`, `name`, age from 
(
    VALUES
    (1,'Alice',45),
    (2,'Bob',null),
    (3,'Charlie',44),
    (4,'Diane',46),
)
ORDER BY age DESC NULLS FIRST
```

Output:
```
2, Bob, null
4, Diane, 46
1, Alice, 45
3, Charlie, 44
```

`NULLS LAST` option would simple place the null results last.

Query:
```sql
select `id`, `name`, age from 
(
    VALUES
    (1,'Alice',45),
    (2,'Bob',null),
    (3,'Charlie',44),
    (4,'Diane',46),
)
ORDER BY age DESC NULLS FIRST
```

Output:
```
4, Diane, 46
1, Alice, 45
3, Charlie, 44
2, Bob, null
```
