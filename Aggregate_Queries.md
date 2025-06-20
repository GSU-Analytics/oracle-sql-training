# Aggregate Queries and GROUP BY Logic

## Module Introduction

In this module, students will develop skills in summarizing data using aggregate functions, grouping results using `GROUP BY`, filtering grouped data with `HAVING`, and restructuring results using `PIVOT` and `UNPIVOT`. These operations are foundational for reporting and dashboarding tasks.

## Explanation

### Aggregate Functions

Aggregate functions perform calculations on sets of rows and return a single result per group. It does not matter if a group consists of a single row or many rows.

* `COUNT`, `COUNT(DISTINCT)`
* `SUM`
* `AVG`
* `MIN`, `MAX`

Example:

```sql
SELECT COUNT(*) AS total_students
FROM student;
```

This query counts the total number of students in the database.

- Notice: when we calculate aggregates, we almost always (though not always) end up with fewer rows in our output than we started with.

**Reference**: Lab 6.1

### GROUP BY and HAVING

`GROUP BY` is used to arrange identical data into groups. `HAVING` is used to filter groups based on aggregate values.

Example:

```sql
SELECT zip, COUNT(*) AS student_count
FROM student
GROUP BY zip
HAVING COUNT(*) > 3;
```

This query groups students by ZIP code and shows only ZIP codes with more than 3 students.

We can also group by multiple columns. Separate the columns you'd like to group by with commas, and SQL will group your aggregations within these multi-column groups.

```sql
SELECT zip, employer, COUNT(*) AS student_count
FROM student
GROUP BY zip, employer
HAVING COUNT(*) > 1;
```

- Consider that, when calculating aggregate values, your selected columns should **always** be either one of the grouping variables or the result of some aggregate function!
  - What would happen if you tried to select a column which was not in an aggregate function, nor one of your grouping columns?

**Reference**: Lab 6.2

### Unique Values with DISTINCT

We can use the `DISTINCT` keyword to obtain the *unique* values over a column. 
```sql
SELECT DISTINCT section_id
  FROM enrollment
 ORDER BY section_id;
```

|SECTION_ID|
|----------|
|80        |
|81        |
|82        |
|83        |
|84        |
|... |

We can also use the `DISTINCT` keyword within some aggregate functions. For example, we can count all of the distinct values like this:

```sql
SELECT
  count(DISTINCT section_id) AS num_distinct_sections,
  count(section_id) AS num_section_records
FROM enrollment;
```

**Reference**: Lab 6.1

### PIVOT and UNPIVOT

`PIVOT` converts rows to columns and `UNPIVOT` converts columns back to rows. These operations are useful for creating cross-tabular reports and reshaping data for analysis.

**Basic PIVOT Example:**

First, let's see the data we want to pivot:

```sql
SELECT TO_CHAR(start_date_time, 'DY') AS day,
       COUNT(*) AS num_of_sections
FROM section
GROUP BY TO_CHAR(start_date_time, 'DY')
ORDER BY 2;
```

|DAY|NUM_OF_SECTIONS|
|---|---|
|FRI|4|
|THU|5|
|WED|7|
|SUN|13|
|MON|15|
|SAT|17|
|TUE|17|

Now let's pivot this data to show days as columns:

```sql
SELECT *
FROM (
  SELECT TO_CHAR(start_date_time, 'DY') day,
         COUNT(*) num_of_sections
  FROM section
  GROUP BY TO_CHAR(start_date_time, 'DY')
)
PIVOT (
  SUM(num_of_sections)
  FOR day IN ('MON','TUE', 'WED','THU', 'FRI','SAT','SUN')
);
```

|'MON'|'TUE'|'WED'|'THU'|'FRI'|'SAT'|'SUN'|
|-----|-----|-----|-----|-----|-----|-----|
|15|17|7|5|4|17|13|

**PIVOT with Multiple Grouping Columns:**

```sql
SELECT *
FROM (
  SELECT TO_CHAR(start_date_time, 'DY') day,
         location,
         COUNT(*) num_of_classes
  FROM section
  GROUP BY TO_CHAR(start_date_time, 'DY'), location
)
PIVOT (
  SUM(num_of_classes) 
  FOR day IN ('MON' AS MON, 'TUE' AS TUE, 'WED' AS WED, 'THU' AS THU,
              'FRI' AS FRI, 'SAT' AS SAT, 'SUN' AS SUN)
);
```

**UNPIVOT Example:**

Using a simple example with student grade types:

```sql
SELECT *
FROM (
  SELECT student_id, 
         MAX(CASE WHEN grade_type_code = 'HM' THEN numeric_grade END) AS homework,
         MAX(CASE WHEN grade_type_code = 'QZ' THEN numeric_grade END) AS quiz
  FROM grade
  WHERE student_id = 123
  GROUP BY student_id
)
UNPIVOT (
  grade FOR grade_type IN (homework AS 'HM', quiz AS 'QZ')
);
```

**Reference**: Lab 17.1

## Exercises

**1. Total Number of Courses**
Use the `COURSE` table to find the total number of courses offered. 

**2. Average Course Cost**
Find the average cost of courses in the `COURSE` table, rounding to the nearest dollar.

**3. Student Count by ZIP Code**
List each ZIP code from the `STUDENT` table and count how many students live in each ZIP. Sort by descending count.

**4. Students with More Than One Grade**
From the `GRADE` table, find student IDs that have more than one recorded grade.

**5. Average Grade by Section and Grade Type**
Use `GRADE` to find the average numeric grade for each section and grade type combination.

**6. Pivot Student Count by State**
Pivot the student count by `STATE` (from `ZIPCODE`) into separate columns for GA and AL. Use the query below as a starting point:

```sql
SELECT *
FROM (
  SELECT z.state, s.student_id
  FROM student s
  JOIN zipcode z ON s.zip = z.zip
)
--- Complete the PIVOT operation here
```

## Q&A

In practice, aggregate queries are essential for summarizing data across dimensions. The `GROUP BY` clause often follows business logic such as grouping by location, time period, or product category. The `HAVING` clause acts like a `WHERE` filter but applies to grouped data.

`PIVOT` and `UNPIVOT` are particularly useful when preparing data for visualization or comparing values across multiple dimensions.

Discussion topics:
* When to use `HAVING` vs `WHERE` in aggregate queries
* Real-world applications of PIVOT operations
* Performance considerations with large datasets
* Best practices for grouping data in reporting

## Additional Resources

* Oracle SQL by Example, Chapter 6, Labs 6.1 and 6.2
* Oracle 19c SQL Language Reference: Aggregate and Group Functions
* Sample queries using `ROLLUP`, `CUBE`, and `GROUPING SETS`

## Answers

**1.**

```sql
SELECT COUNT(*) AS total_courses
FROM course;
```

**2.**

```sql
SELECT ROUND(AVG(cost)) AS avg_cost
FROM course;
```

**3.**

```sql
SELECT zip, COUNT(*) AS student_count
FROM student
GROUP BY zip
ORDER BY student_count DESC;
```

**4.**

```sql
SELECT student_id, COUNT(*) AS grade_count
FROM grade
GROUP BY student_id
HAVING COUNT(*) > 1;
```

**5.**

```sql
SELECT section_id, grade_type_code, AVG(numeric_grade) AS avg_grade
FROM grade
GROUP BY section_id, grade_type_code;
```

**6.**

```sql
SELECT *
FROM (
  SELECT z.state, s.student_id
  FROM student s
  JOIN zipcode z ON s.zip = z.zip
)
PIVOT (
  COUNT(student_id) FOR state IN ('GA' AS GA, 'AL' AS AL)
);
```