# Subqueries

## Module Introduction

This module introduces Oracle SQL subqueries, a foundational topic in advanced SQL reporting. Students will learn how to embed subqueries within main queries to achieve powerful, expressive logic in their SQL. We will begin with scalar and nested subqueries, then explore correlated subqueries using `EXISTS`, `ANY`, and `ALL`, and conclude with how subqueries can be embedded in `SELECT` lists and `CASE` expressions.

**Reference**: Chapter 8, Labs 8.1–8.4

## Explanation

### Scalar and Nested Subqueries

A **scalar subquery** returns a single value and can be used wherever a single value is expected.

Example:

```sql
SELECT first_name, last_name,
       (SELECT MAX(cost) FROM course) AS max_course_cost
FROM student
WHERE ROWNUM <= 3;
```

This query adds the maximum course cost as a column to each student record, limiting to 3 rows for display.

A **nested subquery** is placed inside another subquery or the main query. It can return a set of values used by the outer query.

Example:

```sql
SELECT first_name, last_name
FROM student
WHERE student_id IN (
    SELECT student_id
    FROM enrollment
    WHERE section_id IN (
        SELECT section_id
        FROM section
        WHERE location = 'Main Campus'
    )
);
```

This query finds students enrolled in sections at the Main Campus using nested subqueries.

**Reference**: Lab 8.1

### Correlated Subqueries, EXISTS, ANY, ALL

A **correlated subquery** references a column from the outer query and is re-evaluated for each row.

Example:

```sql
SELECT first_name, last_name
FROM student s
WHERE EXISTS (
    SELECT 1
    FROM enrollment e
    WHERE e.student_id = s.student_id
      AND e.final_grade > 90
);
```

This query finds students who have at least one enrollment with a final grade greater than 90.

Use `ANY` or `ALL` to compare values against a list of results from a subquery.

Example with `ANY`:

```sql
SELECT course_no, cost
FROM course
WHERE cost > ANY (
    SELECT cost
    FROM course
    WHERE course_no < 100
);
```

This query retrieves courses with a cost greater than the cost of any course with a course number less than 100.

Example with `ALL`:

```sql
SELECT course_no, cost
FROM course
WHERE cost > ALL (
    SELECT cost
    FROM course
    WHERE description LIKE '%Intro%'
);
```

This query retrieves courses with a cost greater than the cost of all introductory courses.

**Reference**: Lab 8.2, 8.3

### Subqueries in SELECT and CASE

Scalar subqueries can be placed inside a `SELECT` clause to compute derived columns.

Example:

```sql
SELECT student_id,
       (SELECT COUNT(*) FROM enrollment e WHERE e.student_id = s.student_id) AS total_courses
FROM student s;
```

This query counts the number of courses each student is enrolled in.

Subqueries in `CASE` can be used for conditional logic.

Example:

```sql
SELECT student_id,
       CASE
         WHEN (SELECT COUNT(*) FROM enrollment e WHERE e.student_id = s.student_id) > 2 THEN 'Active'
         ELSE 'Low Enrollment'
       END AS enrollment_status
FROM student s;
```

This query assigns an "Active" status to students enrolled in more than 2 courses, otherwise "Low Enrollment".

### A Word of Caution

Subqueries can be very powerful, but try to not overuse them.

- It can be tempting to try to accomplish everything with subqueries, but doing so can come at heavy performance and readability costs.
- One or two subqueries are fine. However, having three or more *nested* subqueries is usually a sign that something might be wrong. Take a step back and see if you can write your query some other way.
- We will talk about alternative ways to accomplish the same tasks without overusing subqueries in a later module.

**Reference**: Lab 8.4

## Exercises

**1.** Find all students from the `student` table whose `zip` code is the same as that of instructor ID 101 in the `instructor` table.

**2.** List all courses from the `course` table that have a `cost` greater than the average course cost.

**3.** Display each student's `first_name` and `last_name` from the `student` table and the number of courses they are enrolled in using the `enrollment` table.

**4.** List `course_no` and `cost` from the `course` table for all courses that are more expensive than *every* course that has a non-null `prerequisite` value.

**5.** Show students from the `student` table who have received a grade in the `grade` table in a section (from the `section` table) taught by an instructor (from the `instructor` table) who lives in the same `zip` code.

**6.** Display each `student_id` from the `student` table and show "Active" if enrolled in 2 or more sections (from the `enrollment` table), else "Inactive", using a subquery inside a `CASE` expression.

## Q&A

Some food for thought:

* How does a correlated subquery differ in performance from a standard subquery?
* When would you choose `EXISTS` over `IN`?
* What are the benefits and drawbacks of using subqueries vs. joins?
* What are the performance implications of using subqueries in the SELECT clause?
* How do you decide between using ANY vs ALL in subquery comparisons?
* When might a subquery be more readable than a complex join?

## Additional Resources

* Oracle SQL by Example, Chapter 8, Labs 8.1–8.4
* Oracle SQL Language Reference: Subqueries
* Oracle LiveSQL Subquery Examples

## Answers

**1.**

```sql
SELECT *
FROM student
WHERE zip = (
    SELECT zip
    FROM instructor
    WHERE instructor_id = 101
);
```

**2.**

```sql
SELECT *
FROM course
WHERE cost > (
    SELECT AVG(cost)
    FROM course
);
```

**3.**

```sql
SELECT s.first_name, s.last_name,
       (SELECT COUNT(*)
        FROM enrollment e
        WHERE e.student_id = s.student_id) AS course_count
FROM student s;
```

**4.**

```sql
SELECT course_no, cost
FROM course
WHERE cost > ALL (
    SELECT cost
    FROM course
    WHERE prerequisite IS NOT NULL
);
```

**5.**

```sql
SELECT DISTINCT s.first_name, s.last_name
FROM student s
JOIN grade g ON s.student_id = g.student_id
JOIN section sec ON g.section_id = sec.section_id
WHERE EXISTS (
    SELECT 1
    FROM instructor i
    WHERE i.instructor_id = sec.instructor_id
      AND i.zip = s.zip
);
```

**6.**

```sql
SELECT s.student_id,
       CASE
         WHEN (SELECT COUNT(*) FROM enrollment e WHERE e.student_id = s.student_id) >= 2 THEN 'Active'
         ELSE 'Inactive'
       END AS status
FROM student s;
```