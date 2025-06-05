# Query Readability and Optimization Techniques

## Module Introduction

This module covers how to write readable, maintainable, and performant SQL code. We'll review formatting practices, documentation techniques, and strategies for optimizing query performance. You'll also learn how to rewrite subqueries using joins and avoid common performance pitfalls.

## Explanation

### Query Format Guidelines

Use consistent, readable formatting:

* Uppercase for SQL keywords (`SELECT`, `FROM`, `WHERE`)
* Indentation for logical blocks
* Aliases to clarify table sources
* Place each clause on a new line

**Example:**

```sql
SELECT s.student_id, s.first_name, z.city
FROM student s
JOIN zipcode z ON s.zip = z.zip
WHERE z.state = 'GA'
ORDER BY s.student_id;
```

This query demonstrates proper formatting with clear aliases, indentation, and logical clause separation.

> This might seem pedantic.
>
> Sometimes, it is. However, code readability matters when you have to verify whether your logic is correct (a common requirement when writing more complicated queries). You will save yourself time by making your query easier to read, because you will be able to read it more easily and make changes based on what you think should be modified.

**Reference**: Appendix B

### Documenting Queries

Add inline comments to explain logic, especially joins and filtering. Use `--` for single-line and `/* ... */` for multi-line notes.

**Example:**

```sql
-- List students with no phone number on file
SELECT student_id, last_name
FROM student
WHERE phone IS NULL;
```

This query includes a comment explaining the business purpose of the query.

**Reference**: Chapter 12

### Rewriting Subqueries as Joins for Performance

Subqueries can often be rewritten as joins to improve performance and clarity.

**Subquery Version:**

```sql
SELECT course_no, description
FROM course
WHERE course_no IN (
    SELECT course_no
    FROM section
    WHERE location = 'L211'
);
```

This query uses a subquery to find courses offered in location L211.

**Join Version:**

```sql
SELECT DISTINCT c.course_no, c.description
FROM course c
JOIN section s ON c.course_no = s.course_no
WHERE s.location = 'L211';
```

This rewritten query uses a join to achieve the same result, often with better performance.

**Reference**: Lab 8.1

### Minimizing Expensive Operations

Tips to improve performance:

* Avoid functions on indexed columns in `WHERE` clauses
* Filter early using `WHERE` before aggregation
* Prefer joins over correlated subqueries
* Minimize use of `DISTINCT`, especially with joins
* Use analytic functions (`RANK`, `ROW_NUMBER`) where possible

**Example (Inefficient):**

```sql
SELECT *
FROM student
WHERE UPPER(last_name) = 'SMITH';
```

This query applies a function to an indexed column, preventing index usage.

**Better:**

```sql
SELECT *
FROM student
WHERE last_name = 'Smith';
```

This optimized query allows the database to use indexes effectively.

**Reference**: Chapter 18

## Exercises

**1. Basic Formatting Practice**
Format the following query for readability:

```sql
select student_id, last_name, phone from student where zip = '30303'
```

**2. Commenting for Clarity**
Add documentation to this query:

```sql
SELECT s.student_id, s.last_name, e.section_id
FROM student s
JOIN enrollment e ON s.student_id = e.student_id;
```

**3. Rewriting Subqueries**
Rewrite using a join:

```sql
SELECT first_name, last_name
FROM student
WHERE zip IN (
    SELECT zip
    FROM zipcode
    WHERE state = 'GA'
);
```

**4. Optimizing Aggregation**
Identify inefficiencies:

```sql
SELECT COUNT(DISTINCT student_id)
FROM (
    SELECT student_id
    FROM enrollment
    WHERE section_id IN (
        SELECT section_id
        FROM section
        WHERE location = 'L211'
    )
);
```

**5. Analytical Rewriting**
Replace `GROUP BY` logic with an analytic function to find the top 1 student by numeric grade per section using the `GRADE` table. Include `student_id`, `section_id`, `numeric_grade`, and the computed rank.

**6. Performance Debugging**
This query runs slowly. Diagnose and improve:

```sql
SELECT *
FROM student
WHERE TO_CHAR(registration_date, 'YYYY') = '2023';
```

## Q&A

* What formatting habits help when reviewing others' code?
* When should you prioritize joins over subqueries?
* How do you decide when to use `DISTINCT`?
* What's the impact of applying functions in `WHERE` clauses?
* How can documenting your SQL improve debugging?
* What are the trade-offs between readable code and optimal performance?
* How do you identify performance bottlenecks in complex queries?

## Additional Resources

* Oracle SQL by Example, Appendix B – SQL Formatting Guidelines
* Oracle SQL by Example, Chapter 12 – Query Documentation
* Oracle SQL by Example, Chapter 18 – Performance Tuning
* Oracle Database Performance Tuning Guide

## Answers

**1.**

```sql
SELECT student_id, last_name, phone
FROM student
WHERE zip = '30303';
```

**2.**

```sql
-- Show each student and their enrollment section ID
SELECT s.student_id, s.last_name, e.section_id
FROM student s
JOIN enrollment e ON s.student_id = e.student_id;
```

**3.**

```sql
SELECT s.first_name, s.last_name
FROM student s
JOIN zipcode z ON s.zip = z.zip
WHERE z.state = 'GA';
```

**4.**

Optimized version:

```sql
SELECT COUNT(DISTINCT e.student_id)
FROM enrollment e
JOIN section s ON e.section_id = s.section_id
WHERE s.location = 'L211';
```

**5.**

```sql
SELECT student_id, section_id, numeric_grade,
       RANK() OVER (PARTITION BY section_id ORDER BY numeric_grade DESC) AS grade_rank
FROM grade;
```

**6.**

Avoid function on date column:

```sql
SELECT *
FROM student
WHERE registration_date >= TO_DATE('2023-01-01', 'YYYY-MM-DD')
  AND registration_date < TO_DATE('2024-01-01', 'YYYY-MM-DD');
```