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

## Basics

### Database

```sql
--0 list databases
SHOW DATABASES;

--1 create database
CREATE DATABASE database_name;

--2 delete database
DROP DATABASE database_name;

--3 use database
USE database_name;

--4 show current database
SELECT database();
```

### Table

```sql
--0 list tables
SHOW TABLES;

--1 create table
CREATE TABLE table_name
(
  column_name data_type,
  ...
);

--2 describe table (method #1)
SHOW COLUMNS FROM table_name;

--3 describe table (method #2)
DESC table_name;

--4 delete table(s)
DROP TABLE table_name[,...];
```

### CRUD

```sql
--0 insert into table ([c]reate)
INSERT INTO table_name(column_name,...) VALUES(value,...),...;
-- or there's an alternative way
INSERT INTO table_name SET column_name=value,...;

--1 select columns from table ([r]ead)
SELECT column_name[ AS alias_name],... FROM table_name WHERE condition;

--2 update entry/entries ([u]pdate)
UPDATE table_name SET column_name=value,... WHERE condition;

--3 delete entry/entries ([d]elete)
DELETE FROM table_name[ WHERE condition];
```

### String functions

```sql
--0 concatenate strings
CONCAT(column_name,...);

--1 slice strings - starting from 1
SUBSTRING(column_name, start_index[, stop_index]);

--2 substitute strings
REPLACE(column_name, pattern_string, replace_string);

--3 reverse strings
REVERSE(column_name);

--4 string length
CHAR_LENGTH(column_name);

--5 uppercase/lowercase strings
UPPER(column_name);
LOWER(column_name);
```

### Refined selection

```sql
--0 select distinct data
SELECT DISTINCT column_name FROM table_name;

--1 select sorted data
SELECT column_name FROM table_name ORDER BY {column_name | column_number};

--2 select limited pieces of data - starting from 0
SELECT column_name FROM table_name LIMIT [start_index, ]number;

--3 select data under condition
SELECT column_name FROM table_name WHERE condition;

--4 wildcards
-- %  : matches any number of any char
-- _  : matches one number of any char
-- \% : literal %
-- \_ : literal _
```

### Aggregate functions

```sql
--0 COUNT()
SELECT COUNT(*) FROM table_name[ GROUP BY column_name];

--1 MIN(), MAX()
SELECT {MIN | MAX}(column_name) FROM table_name[ GROUP BY column_name];

--2 SUM(), AVG()
SELECT {SUM | AVG}(column_name) FROM table_name[ GROUP BY column_name];
```

### Data types

```sql
--0 CHAR(length), VARCHAR(length)
-- CHAR    : fixed length
-- VARCHAR : variable length

--1 INT, DECIMAL(precision, scale) - precise
-- INT     : whole number
-- DECIMAL : decimal number

--2 FLOAT, DOUBLE - approximate
-- FLOAT  : 4B with 0-23 precision
-- DOUBLE : 8B with 24-53 precision

--3 DATE, TIME, DATETIME
-- DATE: YYYY-MM-DD
-- TIME: HH:MM:SS
-- DATETIME: YYYY-MM-DD HH:MM:SS
--
-- MySQL-accepted syntax for inserting date/time can be found at
-- https://dev.mysql.com/doc/refman/8.0/en/date-and-time-literals.html
--
-- Use DATE_FORMAT(date, format) to format output
-- Refer to https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html
-- for other date/time functions
  --0 NOW() === CURRENT_TIMESTAMP
  -- NOW() returns current TIMESTAMP
  --
  --1 distance between two dates
  -- DATEDIFF(date_one, date_two) returns (date_one - date_two) in days
  --
  --2 add and subtract
  -- DATE_ADD(date, INTERVAL expr unit) === (date + INTERVAL expr unit)
  -- DATE_SUB(date, INTERVAL expr unit) === (date - INTERVAL expr unit)

--4 TIMESTAMP
-- Similar to DATETIME, but with smaller time range and lower memory cost
-- Use cases include when user create/update a post/comment
--
-- Two snippets for such purposes
CREATE TABLE comments
(
  context VARCHAR(140),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE comments
(
  context VARCHAR(140),
  changed_at TIMESTAMP DEFAULT NOW() ON UPDATE CURRENT_TIMESTAMP
);
```

### Condition

```sql
--0 AND/OR - chainable
SELECT column_name FROM table_name WHERE condition {AND | OR} condition;

--1 [NOT ]LIKE
SELECT column_name FROM table_name WHERE column_name [NOT ]LIKE pattern_string;

--2 [NOT ]BETWEEN...AND... - inclusive
SELECT column_name FROM table_name WHERE column_name [NOT ]BETWEEN min_value AND max_value;
  -- Same as
  SELECT column_name FROM table_name
  WHERE column_name >= min_value AND
        column_name <= max_value;

--3 [NOT ]IN
SELECT column_name FROM table_name WHERE column_name [NOT ]IN (value,...);

--4 CASE statement - no commas
CASE
  WHEN condition THEN value
  [WHEN condition THEN value]
  ...
  [ELSE statement_list]
END

--5 IF statement - often used in FUNCTION or TRIGGER create statement
IF condition THEN statement_list
[ELSEIF condition THEN statement_list]...
[ELSE THEN statement_list]
END IF
```

## Joins

### One to many

![One-to-many relationship](https://i.imgur.com/lGL3C4g.png)
![2-set intersection](https://i.imgur.com/44MDi5R.png)

```sql
--0 using foreign keys

-- *ON DELETE CASCADE* makes sure parent entry gets deleted along with its
-- children dependent on it

CREATE TABLE students (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(20));
CREATE TABLE papers (
  title VARCHAR(50),
  grade INT,
  student_id INT,
  FOREIGN KEY(student_id) REFERENCES students(id) ON DELETE CASCADE
);

-- The condition is usually like: students.id = papers.student_id

--0 inner join - A∩B
-- It doesn't really matter who joins whom in inner join, because column_name
-- could be any in A∪B. Use table_name.column_name if there is ambiguity.
SELECT column_name FROM table_name [INNER ]JOIN table_name ON condition;

--1 left join - A (row number: A)
-- When there are no entries from table A that doesn't have a relation with an
-- entry in table B, [inner join] === [left join]. In this case, when every
-- student has at least one paper, [inner join] === [left join]. (A = A∩B)
SELECT column_name FROM table_name LEFT JOIN table_name ON condition;

--2 right join - B (row number: B)
-- When there are no entries from table B that doesn't have a relation with an
-- entry in table A, [inner join] === [right join]. In this case, when every
-- paper has its owner, [inner join] === [right join]. (B = A∩B)
SELECT column_name FROM table_name RIGHT JOIN table_name ON condition;

--3 order matters
SELECT column_name FROM A LEFT JOIN B ON condition;
-- is the same as:
SELECT column_name FROM B RIGHT JOIN A ON condition;
-- so any right join could be rewritten as left join, which is more conventional

--4 don't panic
-- Joint tables are no different than others, so *GROUP BY/ORDER BY* and other
-- aggregate functions still work fine.

--5 a little trick with NULL
-- To avoid seeing NULLs in the query result, use IFNULL(expr, default_value).
-- PS: Use [column_name IS NULL](✓) instead of [column_name = NULL](✗)

--6 a little trick with COUNT
-- When left join produces NULLs and COUNT is needed, use COUNT(column_name)
-- instead because COUNT(*) will result in 1 even if there is -- no related
-- entry but COUNT(column_name) will result in 0, which in most cases is what we
-- want.

-- A little example:
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

### Many to many

![Many-to-many relationship](https://i.imgur.com/iRFFUV2.png)
![3-set intersection](https://i.imgur.com/J7idXQA.png)

```sql
-- Schema example:

CREATE TABLE reviewers (
  id INT AUTO_INCREMENT PRIMARY KEY,
  fname VARCHAR(100),
  lname VARCHAR(100)
);

CREATE TABLE series (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(100),
  released_year YEAR(4),
  genre VARCHAR(20)
);

CREATE TABLE reviews (
  id INT AUTO_INCREMENT PRIMARY KEY,
  rating DECIMAL(2, 1),
  series_id INT,
  reviewer_id INT,
  FOREIGN KEY (series_id) REFERENCES series(id),
  FOREIGN KEY (reviewer_id) REFERENCES reviewers(id)
);

-- Multi inner joins example (which corresponds to the figure above):

SELECT
  title,
  rating,
  CONCAT(fname, ' ', lname) AS reviewer
FROM
  reviews
  INNER JOIN reviewers ON reviews.reviewer_id = reviewers.id
  INNER JOIN series ON reviews.series_id = series.id
ORDER BY
  title,
  rating DESC;

-- The order of the three tables being joined doesn't really matter.
```

## Triggers

```sql
--0 list triggers
SHOW TRIGGERS;

--1 create trigger
-- Creating a trigger is just like binding an event listener

-- trigger_time  : { BEFORE | AFTER }
-- trigger_event : { INSERT | UPDATE | DELETE }

-- NEW : Newly inserted/updated entry
-- OLD : Newly deleted entry

DELIMITER |
CREATE TRIGGER trigger_time trigger_event ON table_name
  FOR EACH ROW
  BEGIN
    statement_list
  END;
|
DELIMITER ;

--2 delete trigger(s)
DROP TRIGGER trigger_name[,...];

-- Use cases involve validation and logging. There are often alternative ways of
-- serving the purpose, like doing the validation on the client side. Be careful
-- when using triggers because it can cause unexpected problems. Use it only
-- when it is the only feasible way.
```

## Tips

```sql
--0 ROUND(X[, D=0]) => round X to D decimal places

--1 YEAR(N)
-- A 4-digit datatype for years

--2 IF(condition, yes_value, no_value)
-- IF() *function* is different from IF *statement*
-- SQL version of ternary operator

--3 UNIQUE (after datatype)
-- This keyword indicates that a field must not have duplicates.

--4 PRIMARY KEY(column_name, column_name)
-- The pair as a whole should by unique.

--5 WHERE, HAVING
-- When using values generated with *GROUP BY*, WHERE after *GROUP BY* takes no
-- effect, in which case HAVING will serve the purpose.

--6 Sub Queries
-- Use result of another query as value.

-- The following example includes several tricks mentioned above:

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

## Credits

[I took this course by Colt Steele.](https://www.udemy.com/course/the-ultimate-mysql-bootcamp-go-from-sql-beginner-to-expert/)
