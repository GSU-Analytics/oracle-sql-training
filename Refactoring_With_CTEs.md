# Refactoring Queries with Common Table Expressions

## Module Introduction

This module explores how to use Common Table Expressions (CTEs) in Oracle SQL to write clearer, testable, and reusable queries. CTEs are defined using the `WITH` clause and allow you to isolate logical parts of your query, improving readability and simplifying testing.

## Explanation

### Using CTEs to Improve Readability

CTEs separate complex logic into named blocks that read like building steps. This is helpful when working with long queries, nested subqueries, or aggregations.

**Example:** Find the locations of sections with more than 10 students enrolled.

```sql
WITH high_enrollments AS (
  SELECT section_id
  FROM enrollment
  GROUP BY section_id
  HAVING COUNT(*) > 10
)
SELECT s.section_id, s.location
FROM section s
JOIN high_enrollments h ON s.section_id = h.section_id;
```

This query first identifies sections with high enrollments, then retrieves their locations, making it easier to understand each step.

**Reference:** Lab 17.1

### CTEs for Testing

Use CTEs to isolate and verify subquery logic. This allows you to debug step-by-step rather than deciphering deeply nested SQL.

**Example:** Calculate average grade per student, then select only those with an average above 85.

```sql
WITH valid_grades AS (
  SELECT student_id, ROUND(AVG(numeric_grade), 2) avg_grade
  FROM grade
  GROUP BY student_id
)
SELECT *
FROM valid_grades
WHERE avg_grade > 85;
```

This CTE first computes the average grade for each student, then filters those with an average above 85, making it clear what each part of the query does.

**Reference:** Lab 17.1

### CTEs and Reusability

CTEs allow modular design where intermediate results can be reused. They also support chaining: one CTE can build on another.

**Example:** Identify students enrolled in at least three sections, then list their names.

```sql
WITH enroll_count AS (
  SELECT student_id, COUNT(*) AS total_courses
  FROM enrollment
  GROUP BY student_id
),
active_students AS (
  SELECT student_id
  FROM enroll_count
  WHERE total_courses >= 3
)
SELECT s.first_name, s.last_name
FROM student s
JOIN active_students a ON s.student_id = a.student_id;
```

This example first counts enrollments per student, then filters those with three or more enrollments, and finally retrieves their names.

**Reference:** Lab 17.1

### Thinking about CTEs as Variables

If you have any experience with more general programming languages like R or Python, you may notice some similarities between CTEs and the idea of a *variable*. It is worth taking a moment to compare the two.

#### Similarities
1. Scope and Lifetime
   - Both CTEs and variables have a defined scope and lifetime. CTEs are limited to the query they are defined in, while variables can have different scopes (local, global) depending on the programming language.
2. Reusing Information Checkpoints
   - Both are used to simplify complex operations. CTEs break down complex SQL queries, and variables store data for manipulation throughout a program.
3. Readability and Maintenance
   - Both can improve readability and maintainability. CTEs make SQL queries easier to understand by breaking them into named components, while variables use meaningful names to make code more readable.

#### Differences
1. Performance
   - CTEs: Can lead to performance issues if not optimized, especially with large datasets or complex recursive queries.
   - Variables: Generally have minimal performance impact, but their usage in loops or recursive functions can affect program performance.
2. Error Handling
   - CTEs: Limited to the SQL query execution context. If a CTE fails, the entire query fails.
   - Variables: Traditional programming languages offer robust error handling mechanisms (e.g., try-catch blocks) for more granular control over errors.
3. Flexibility
   - CTEs: Limited to SQL queries and primarily used for data manipulation and retrieval.
   - Variables: Highly flexible and can be used in various contexts within a program, including arithmetic operations, string manipulation, and more.

CTEs are not quite as flexible as a variable in a more general language, but you may find it useful to think of them as "table variables." Use this understanding to make it easier for yourself to write complicated queries!

## Exercises

1. **Basic CTE Syntax**
   Create a CTE that selects all students who live in ZIP code '30303'.

2. **Aggregate Inside a CTE**
   Use a CTE to find the average course cost and select only courses that cost more than this average.

3. **Testing a Join with CTE**
   Use a CTE to join `enrollment` and `grade` on `student_id` and `section_id`, and select the average grade per student.

4. **Chain Two CTEs**
   First, use a CTE to identify students with more than one grade recorded. Then, in a second CTE, get those who have an average grade above 90.

5. **Reuse Logic with CTE**
   Use a CTE to calculate enrollments per student. Then use this CTE twice: once to filter those with 3+ enrollments, once to get those with fewer.

6. **Refactor a Nested Query**
   Refactor this nested query into CTEs:

```sql
SELECT student_id, first_name, last_name
FROM student
WHERE student_id IN (
  SELECT student_id
  FROM enrollment
  GROUP BY student_id
  HAVING COUNT(*) > 2
);
```

## Q&A

* When should you use a CTE instead of a subquery?
* What are the tradeoffs of readability vs performance in CTEs?
* How do chained CTEs improve logic clarity?
* How can you test each step of a complex query using CTEs?
* How does using CTEs relate to writing modular code in other languages?
* What are the performance implications of using multiple CTEs in a single query?
* When might a CTE be overkill for a simple query?

## Answers

**1.**

```sql
WITH zip_students AS (
  SELECT * FROM student WHERE zip = '30303'
)
SELECT * FROM zip_students;
```

**2.**

```sql
WITH avg_cost AS (
  SELECT AVG(cost) AS avg_c FROM course
)
SELECT *
FROM course
WHERE cost > (SELECT avg_c FROM avg_cost);
```

**3.**

```sql
WITH joined AS (
  SELECT e.student_id, g.numeric_grade
  FROM enrollment e
  JOIN grade g ON e.student_id = g.student_id AND e.section_id = g.section_id
)
SELECT student_id, AVG(numeric_grade) AS avg_grade
FROM joined
GROUP BY student_id;
```

**4.**

```sql
WITH multiple_grades AS (
  SELECT student_id
  FROM grade
  GROUP BY student_id
  HAVING COUNT(*) > 1
),
excellent_students AS (
  SELECT student_id, AVG(numeric_grade) avg_grade
  FROM grade
  WHERE student_id IN (SELECT student_id FROM multiple_grades)
  GROUP BY student_id
  HAVING AVG(numeric_grade) > 90
)
SELECT * FROM excellent_students;
```

**5.**

```sql
WITH enrollment_counts AS (
  SELECT student_id, COUNT(*) AS count
  FROM enrollment
  GROUP BY student_id
)
SELECT * FROM enrollment_counts WHERE count >= 3;
```

For students with fewer enrollments:

```sql
WITH enrollment_counts AS (
  SELECT student_id, COUNT(*) AS count
  FROM enrollment
  GROUP BY student_id
)
SELECT * FROM enrollment_counts WHERE count < 3;
```

**6.**

```sql
WITH enrolled AS (
  SELECT student_id
  FROM enrollment
  GROUP BY student_id
  HAVING COUNT(*) > 2
)
SELECT s.student_id, s.first_name, s.last_name
FROM student s
JOIN enrolled e ON s.student_id = e.student_id;
```