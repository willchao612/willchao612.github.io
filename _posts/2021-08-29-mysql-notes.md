---
layout: post
title: 'MySQL Notes'
subtitle: ''
author: 'Will'
header-style: text
lang: en
tags:
  - MySQL
  - 基础
  - 数据库
  - 笔记
---

## Database operations

- List databases

```sql
SHOW DATABASES;
```

- Create database

```sql
CREATE DATABASE <database_name>;
```

- Delete database

```sql
DROP DATABASE <database_name>;
```

- Use database

```sql
USE <database_name>;
```

- Show current database

```sql
SELECT database();
```

## Table operations

- List tables

```sql
SHOW TABLES;
```

- Create table

```sql
CREATE TABLE <table_name>
(
  <column_name> <data_type> <constraint>,
  ...
);
```

- Describe table

```sql
SHOW COLUMNS FROM <table_name>;
DESC <table_name>;
```

- Delete tables

```sql
DROP TABLE <table_name>,...;
```

## CRUD operations

- Insert

```sql
INSERT INTO <table_name>(<column_name>,...) VALUES(<value>,...),...;
INSERT INTO <table_name> SET <column_name>=<value>,...;
```

- Select

```sql
SELECT <column_name> AS <alias_name>,... FROM <table_name> WHERE <condition>;
```

- Update

```sql
UPDATE <table_name> SET <column_name>=<value>,... WHERE <condition>;
```

- Delete

```sql
DELETE FROM <table_name> WHERE <condition>;
```

## Refined selection

- Distinct

```sql
SELECT DISTINCT <column_name> FROM <table_name>;
```

- Sort

```sql
SELECT <column_name> FROM <table_name> ORDER BY {<column_name> | <column_number>};
```

- Limit (index starting from 0)

```sql
SELECT <column_name> FROM <table_name> LIMIT [<start_index>, ]<num_of_entries>;
```

- Conditional

```sql
SELECT <column_name> FROM <table_name> WHERE <condition>;
```

## Aggregate functions

- Count

```sql
SELECT COUNT({* | <column_name>}) FROM <table_name> GROUP BY <column_name> HAVING <condition>;
```

- Min / Max / Sum / Avg

```sql
SELECT {MIN | MAX | SUM | AVG}(<column_name>) FROM <table_name> GROUP BY <column_name> HAVING <condition>;
```

## String functions

- Concatenate

```sql
CONCAT(<column_name>,...);
```

- Slice (index starting from 1)

```sql
SUBSTRING(<column_name>, <start_index>[, <stop_index>]);
```

- Substitute

```sql
REPLACE(<column_name>, <pattern_string>, <replace_string>);
```

- Reverse

```sql
REVERSE(<column_name>);
```

- Length

```sql
CHAR_LENGTH(<column_name>);
```

- Uppercase / Lowercase

```sql
UPPER(<column_name>);
LOWER(<column_name>);
```

## Help functions

- If null default value

```sql
IFNULL(<column_name>, <default_value>)
```

- Round

```sql
ROUND(X)
ROUND(X, D)
```

- Ternery

```sql
IF(<condition>, <yes_value>, <no_value>)
```

## Data types

| Type             | Syntax                          | Description         |
| ---------------- | ------------------------------- | ------------------- |
| Char             | `CHAR(<length>)`                | Fixed length        |
| Varchar          | `VARCHAR(<length>)`             | Variable length     |
| Integer          | `INT`                           |                     |
| Decimal          | `DECIMAL(<precision>, <scale>)` | Precise             |
| Float            | `FLOAT`                         | Approximate         |
| Double           | `DOUBLE`                        | Approximate         |
| Date             | `DATE`                          | YYYY-MM-DD          |
| Time             | `TIME`                          | HH:MM:SS            |
| Datetime         | `DATETIME`                      | YYYY-MM-DD HH:MM:SS |
| Timestamp        | `TIMESTAMP`                     | See example below   |
| Year             | `YEAR`                          | YYYY                |

> `TIMESTAMP` is similar to `DATETIME`, but with smaller range and lower cost.

```sql
CREATE TABLE comments
(
  context VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  changed_at TIMESTAMP DEFAULT NOW() ON UPDATE CURRENT_TIMESTAMP
);
```

## Constraints

| Type          | Effect                                                   |
| ------------- | -------------------------------------------------------- |
| `NOT NULL`    | Ensures that a column cannot be `NULL`                   |
| `UNIQUE`      | Ensures that all values in a column are different        |
| `PRIMARY KEY` | Uniquely identifies every row in a table                 |
| `FOREIGN KEY` | References a primary key from another table or itself    |
| `CHECK`       | Ensures that all values in a column satisify a condition |
| `DEFAULT`     | Default value for a column                               |

```sql
CREATE TABLE <table_name>
(
  <column_name> <data_type> NOT NULL,
  <column_name> <data_type> UNIQUE,
  <column_name> <data_type> PRIMARY KEY,
  <column_name> <data_type> FOREIGN KEY REFERENCES <table_name>(<column_name>),
  <column_name> <data_type> DEFAULT <default_value>,
  <column_name> <data_type> CHECK (<condition>),
  PRIMARY KEY (<column_name>,...),
  FOREIGN KEY (<column_name>) REFERENCES <table_name>(<column_name>),
  CONSTRAINT <constraint_name> {CHECK ... | PRIMARY ... | FOREIGN ... },
  ...
);
```

## Conditionals

- Logical operater

```sql
SELECT <column_name> FROM <table_name> WHERE <condition> {AND | OR} <condition>;
```

- Like

```sql
SELECT <column_name> FROM <table_name> WHERE <column_name> LIKE <pattern_string>;
```

- Between ... and ... (inclusive)

```sql
SELECT <column_name> FROM <table_name> WHERE <column_name> BETWEEN <min_value> AND <max_value>;
```

- In

```sql
SELECT <column_name> FROM <table_name> WHERE <column_name> IN (<value>,...);
```

- Case

```sql
CASE WHEN <condition> THEN <value> ... ELSE <value> END
```

- Wildcards

| Symbol | Match                  |
| ------ | ---------------------- |
| `%`    | any number of any char |
| `_`    | one of any char        |
| `\%`   | literal `%`            |
| `\_`   | literal `_`            |

## Table joins

> Use `<table_name>.<column_name>` if there is ambiguity.

- Inner join

```sql
SELECT <column_name> FROM <table_name> JOIN <table_name> ON <condition>;
```

- Left / Right join

```sql
SELECT <column_name> FROM <table_name> {LEFT | RIGHT} JOIN <table_name> ON <condition>;
```

## Tips

- Use `<column_name> IS NULL` rather than `<column_name> = NULL`
- Use `COUNT(<column_name>)` rather than `COUNT(*)` if `NULL` should not be
   counted
- `ON DELETE CASCADE` ensures a row gets deleted automatically when the row it
   references is deleted

## Examples

- Select users who have liked all photos

```sql
SELECT
  username
FROM
  likes
  INNER JOIN users ON likes.user_id = users.id
GROUP BY
  user_id
HAVING
  COUNT(*) = (
    SELECT
      COUNT(*)
    FROM
      photos
  );
```

- One to many relationship

![One-to-many relationship](https://i.imgur.com/lGL3C4g.png)

```sql
CREATE TABLE students
(
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255)
);

CREATE TABLE papers
(
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255),
  grade INT,
  student_id INT,
  FOREIGN KEY(student_id) REFERENCES students(id) ON DELETE CASCADE
);

SELECT
  name,
  IFNULL(AVG(grade), 0) AS average,
  CASE
    WHEN IFNULL(AVG(grade), 0) >= 75 THEN 'PASSING'
    ELSE 'FAILING'
  END AS passing_status
FROM
  students
  LEFT JOIN papers ON students.id = papers.student_id
GROUP BY
  students.id
ORDER BY
  average DESC;
```

- Many to many relationship

![Many-to-many relationship](https://i.imgur.com/iRFFUV2.png)

```sql
CREATE TABLE reviewers (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255)
);

CREATE TABLE series (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255),
  released_year YEAR(4),
  genre VARCHAR(255)
);

CREATE TABLE reviews (
  id INT AUTO_INCREMENT PRIMARY KEY,
  rating DECIMAL(2, 1),
  series_id INT,
  reviewer_id INT,
  FOREIGN KEY (series_id) REFERENCES series(id),
  FOREIGN KEY (reviewer_id) REFERENCES reviewers(id)
);

SELECT
  title,
  rating,
  name
FROM
  reviews
  JOIN reviewers ON reviews.reviewer_id = reviewers.id
  JOIN series ON reviews.series_id = series.id
ORDER BY
  title,
  rating DESC;
```

## References

- [Date functions](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html)

## Credits

[I took this course by Colt Steele.](https://www.udemy.com/course/the-ultimate-mysql-bootcamp-go-from-sql-beginner-to-expert/)
