# Working with Single Tables

## Module Introduction

Before proceeding with the rest of the course, it is our hope that you have at least a passing familiarity with basic SQL. In particular, you should be moderately comfortable with writing single-table queries.

> If you need a more detailed introduction to SQL, see @maddala22 for a refresher.

This module will serve as a review of foundational ideas from introductory courses, with an emphasis on simple, single-table operations. These include:

- `SELECT` basics
- Filtering, sorting, and aliasing
- Using built-in functions

You don't need to be an expert in any of these to proceed. Simply having seen them before, and being willing to review, will be sufficient.

**Reference**: *Oracle SQL by Example (4th Edition)*, Chapter 2

## Explanation

### An Introduction to SQL Clauses

A SQL clause is a specific component of a SQL statement that performs a particular function within the query. Clauses are the building blocks that make up complete SQL statements and help define what data you want to retrieve, modify, or manipulate.

> By learning to compose SQL clauses, you will learn to construct both basic and complex queries.

Each clause has a specific purpose and follows a defined syntax. For example, in this basic `SELECT` statement:

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

There are three clauses:

1. `SELECT` clause - Specifies which columns to retrieve
2. `FROM` clause - Identifies the table(s) to query
3. `WHERE` clause - Filters records based on conditions

Clauses must appear in a specific order within a SQL statement. For instance, a `WHERE` clause must come after a `FROM` clause, but before a `GROUP BY` clause in a `SELECT` statement.

### Basic SELECT Statements

The `SELECT` keyword indicates to SQL that you are interested in querying data. The basic syntax is:

```sql
SELECT column1, column2, ...
FROM table_name;
```

Let's break this down:

- `SELECT` specifies which columns you want to retrieve
- `FROM` identifies which table the data comes from

For example, to get the course number, description, and prerequisite information from the `course` table, you would write:

```sql
SELECT course_no,
       description,
       prerequisite
  FROM course;
```

This query retrieves the course number, description, and any prerequisite courses for all courses in the system.

Executing this query will return the data below (only the first 5 rows are shown):

|COURSE_NO|DESCRIPTION                 |PREREQUISITE|
|---------|----------------------------|------------|
|10       |Technology Concepts         |            |
|20       |Intro to Information Systems|            |
|25       |Intro to Programming        |140         |
|80       |Programming Techniques      |204         |
|100      |Hands-On Windows            |20          |

You can also use `SELECT *` to retrieve all columns.

```sql
SELECT *
FROM course;
```

This query returns all columns from the course table.

> This is less efficient than specifying each column, so it should not be used if you are building applications. However, it is fine to use this for ad-hoc querying.

### Filtering, Sorting, and Aliasing

#### Filtering with WHERE

The `WHERE` clause filters your query results based on specified conditions.

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

Common `WHERE` Operators include:

- **Comparison operators**: `=`, `>`, `<`, `>=`, `<=`, `<>` (not equal to)
- **Logical operators**: `AND`, `OR`, `NOT`
- **Range**: `BETWEEN value1 AND value2`
- **List membership**: `IN (value1, value2, ...)`
- **Pattern matching**: `LIKE 'pattern'` (with `%` as wildcard)
- **NULL values**: `IS NULL` or `IS NOT NULL`

Here's an example:

```sql
SELECT course_no,
       description,
       prerequisite
  FROM course
 WHERE course_no BETWEEN 100 AND 140;
```

This query filters courses to show only those with course numbers between 100 and 140, inclusive.

|COURSE_NO|DESCRIPTION                 |PREREQUISITE|
|---------|----------------------------|------------|
|100      |Hands-On Windows            |20          |
|120      |Intro to Java Programming   |80          |
|122      |Intermediate Java Programming|120         |
|124      |Advanced Java Programming   |122         |
|125      |Java Developer I            |122         |
|130      |Intro to Unix               |310         |
|132      |Basics of Unix Admin        |130         |
|134      |Advanced Unix Admin         |132         |
|135      |Unix Tips and Techniques    |134         |

#### Sorting with ORDER BY

The `ORDER BY` clause sorts your results based on specified columns.

```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC], ...;
```

You can use the following keywords to determine the sorting order:

- `ASC` - Ascending order (default if not specified)
- `DESC` - Descending order

Let's order the courses alphabetically.

```sql
SELECT course_no,
       description,
       prerequisite
  FROM course
 WHERE course_no BETWEEN 100 AND 140
 ORDER BY description ASC;
```

This query sorts the filtered courses by their description in alphabetical order.

|COURSE_NO|DESCRIPTION                 |PREREQUISITE|
|---------|----------------------------|------------|
|124      |Advanced Java Programming   |122         |
|134      |Advanced Unix Admin         |132         |
|132      |Basics of Unix Admin        |130         |
|100      |Hands-On Windows            |20          |
|122      |Intermediate Java Programming|120         |
|120      |Intro to Java Programming   |80          |
|130      |Intro to Unix               |310         |
|125      |Java Developer I            |122         |
|140      |Systems Analysis            |20          |
|135      |Unix Tips and Techniques    |134         |

#### Aliases

Aliases allow you to rename columns in your result set for clarity or convenience.

```sql
SELECT column_name [AS] alias_name
FROM table_name;
```

> The `AS` keyword is technically optional. We recommend you include it for clarity.

```sql
SELECT course_no AS "Course Number",
       description AS "Description",
       prerequisite AS "Prerequisites"
  FROM course
 WHERE course_no BETWEEN 100 AND 130
 ORDER BY description ASC;
```

This query creates more readable column headers using aliases.

|Course Number|Description                 |Prerequisites|
|-------------|----------------------------|-------------|
|124          |Advanced Java Programming   |122          |
|100          |Hands-On Windows            |20           |
|122          |Intermediate Java Programming|120          |
|120          |Intro to Java Programming   |80           |
|130          |Intro to Unix               |310          |
|125          |Java Developer I            |122          |

Table aliases provide shorthand references to tables. These can be useful for your own understanding, but they can also be necessary when you are working with multiple tables. We'll discuss working with multiple tables in later modules.

```sql
SELECT a.column_name, b.column_name
FROM table1 AS a, table2 AS b
WHERE a.common_field = b.common_field;
```

The query below will fetch the same data as the previous query, but the table has been renamed to `course_list`.

```sql
SELECT course_list.course_no AS "Course Number",
       course_list.description AS "Description",
       course_list.prerequisite AS "Prerequisites"
  FROM course course_list
 WHERE course_no BETWEEN 100 AND 130
 ORDER BY description ASC;
```

This query demonstrates table aliasing for clarity, even though it's not necessary with a single table.

### Built-In Functions

Oracle SQL provides numerous built-in functions to manipulate data, perform calculations, and format results.

> - These functions do **not** aggregate data. They operate element-wise on your inputs. We will discuss aggregating functions later. 
> - Don't worry about memorizing these functions. Use this as a reference when you need it.

Below are some of the most commonly used functions, categorized by type.

#### String Functions

String functions make it easy to manipulate textual data. Here are some common useful string functions.

| Function | Description | Example | Result |
|----------|-------------|---------|--------|
| `LOWER(str)` | Converts string to lowercase | `LOWER('HELLO')` | 'hello' |
| `UPPER(str)` | Converts string to uppercase | `UPPER('hello')` | 'HELLO' |
| `INITCAP(str)` | Capitalizes first letter of each word | `INITCAP('hello world')` | 'Hello World' |
| `SUBSTR(str, start, [length])` | Extracts substring | `SUBSTR('HELLO', 2, 3)` | 'ELL' |
| `LENGTH(str)` | Returns length of string | `LENGTH('HELLO')` | 5 |
| `CONCAT(str1, str2)` | Concatenates two strings (You can also use the `||` operator) | `CONCAT('Hello', ' World')` | 'Hello World' |
| `REPLACE(str, search, replace)` | Replaces all occurrences | `REPLACE('JACK AND JUE','J','BL')` | 'BLACK AND BLUE' |
| `TRIM([chars FROM] str)` | Removes specified characters | `TRIM(' hello ')` | 'hello' |
| `LPAD(str, length, [pad_str])` | Left-pad string | `LPAD('123', 5, '0')` | '00123' |
| `RPAD(str, length, [pad_str])` | Right-pad string | `RPAD('ABC', 5, 'XY')` | 'ABCXY' |

#### Numeric Functions

Numeric functions make it easy to manipulate numerical data. Here are some common useful numerical functions.

| Function | Description | Example | Result |
|----------|-------------|---------|--------|
| `ROUND(n, [decimal])` | Rounds number to specified decimal places | `ROUND(125.315, 2)` | 125.32 |
| `TRUNC(n, [decimal])` | Truncates number to specified decimal places | `TRUNC(125.315, 2)` | 125.31 |
| `CEIL(n)` | Returns smallest integer greater than or equal to n | `CEIL(125.3)` | 126 |
| `FLOOR(n)` | Returns largest integer less than or equal to n | `FLOOR(125.3)` | 125 |
| `MOD(m, n)` | Returns remainder when m is divided by n | `MOD(11, 4)` | 3 |
| `ABS(n)` | Returns absolute value | `ABS(-15)` | 15 |
| `POWER(n, m)` | Returns n raised to the power of m | `POWER(3, 2)` | 9 |
| `SQRT(n)` | Returns square root | `SQRT(25)` | 5 |

#### Conversion Functions

Conversion functions make it easy to transform data into different types. Here are some common useful conversion functions.

| Function | Description | Example | Result |
|----------|-------------|---------|--------|
| `TO_CHAR(value, [format])` | Converts number or date to string | `TO_CHAR(1234.56, '$9,999.99')` | '$1,234.56' |
| `TO_NUMBER(str, [format])` | Converts string to number | `TO_NUMBER('$1,234.56', '$9,999.99')` | 1234.56 |
| `CAST(expr AS type)` | Converts expression to specified datatype | `CAST('123' AS NUMBER)` | 123 |
| `NVL(expr1, expr2)` | Returns expr2 if expr1 is NULL | `NVL(NULL, 0)` | 0 |
| `NVL2(expr1, expr2, expr3)` | Returns expr2 if expr1 is NOT NULL, else expr3 | `NVL2(NULL, 'A', 'B')` | 'B' |
| `COALESCE(expr1, expr2, ...)` | Returns first non-NULL expression | `COALESCE(NULL, NULL, 'A', 'B')` | 'A' |

### Missing Values

In Oracle SQL, an empty value in the table is called a `NULL` value. It means that there is no information for that record.

Some programming languages treat certain values like `NULL` values. For example, a value of 0 is treated like it's false in languages like Python and JavaScript. The table below shows how 0's and empty strings differ from `NULL` values.

| **Aspect**       | **Null Values**                          | **0 (Zero)**                          | **Empty Strings**                     |
|------------------|------------------------------------------|---------------------------------------|---------------------------------------|
| **Meaning**      | Absence of a value or unknown data       | Numeric value of zero                 | Treated as null                       |
| **Behavior**     | Not equal to anything, including other nulls | Treated as a number, used in arithmetic operations | Stored as null                        |
| **Usage**        | Used when the actual value is not known or not applicable | Used to represent a numeric value that is explicitly zero | Used when no data is provided         |


## Exercises

### Simple problems

1. Write a SELECT statement that lists the first and last names of all students.

2. Write a SELECT statement that lists all cities, states, and zip codes. Order them alphabetically by state.

3. Write a SELECT statement that lists each city and zip code in New York or Connecticut. Sort the results in ascending order by zip code.

### Complex problems

1. Write a SELECT statement that lists the first and last names of instructors with the letter i (either uppercase or lowercase) in their last name, living in zip code 10025.

2. Write a SELECT statement that returns a student's ID, and their name in the format of "Last, First" for each student. Alias the name column to `Full Name`. Sort the results by their *first* name.

## Q&A

Some food for thought:

- What happens if you specify a `WHERE` clause, but no rows meet the requirements?
- Can you order by a feature which isn't in the output?
- What do you do if you need to use multiple functions to get the output you want?

## Additional Resources

For more details, see the resources below.

### Further Reading
- Quickstart Guide to SQL [@maddala22]
- Oracle SQL by Example [@rischert09, Ch. 2-4]

## Answers

### Simple problems

1. Write a SELECT statement that lists the first and last names of all students.

```sql
SELECT first_name,
       last_name
  FROM student;
```

2. Write a SELECT statement that lists all cities, states, and zip codes. Order them alphabetically by state.

```sql
SELECT city,
       state,
       zip
  FROM zipcode
 ORDER BY state;
```

3. Write a SELECT statement that lists each city and zip code in New York or Connecticut. Sort the results in ascending order by zip code.

```sql
SELECT city,
       zip
  FROM zipcode
 WHERE state = 'NY'
    OR state = 'CT'
 ORDER BY zip;
```

### Complex problems

1. Write a SELECT statement that lists the first and last names of instructors with the letter i (either uppercase or lowercase) in their last name, living in zip code 10025.

```sql
SELECT first_name,
       last_name
  FROM instructor
 WHERE last_name LIKE '%i%'
    OR last_name LIKE '%I%'
   AND zip = '10025';
```

2. Write a SELECT statement that returns a student's ID, and their name in the format of "Last, First" for each student. Alias the name column to `Full Name`. Sort the results by their *first* name.

```sql
SELECT student_id,
       CONCAT(CONCAT(last_name, ', '), first_name) AS "Full Name"
  FROM student
 ORDER BY first_name;
```

Alternative solution using concatenation operator:

```sql
SELECT student_id,
       last_name || ', ' || first_name AS "Full Name"
  FROM student
 ORDER BY first_name;
```