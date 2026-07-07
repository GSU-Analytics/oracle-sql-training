# Advanced Aggregations

## Module Introduction

In this module, we will discuss some powerful ways to calculate aggregates across groups. You will do this by using `ROLLUP`, `CUBE`, and the `GROUPING` function.

**Reference**: Oracle SQL by Example, Chapter 6 · Oracle SQL Language Reference, *Aggregate and Group Functions*

## Explanation

### Motivation: Queries with Subtotals

`GROUP BY` gets you a single level of summary — one row per group. But reporting often calls for *subtotals and grand totals alongside the detail*, the way a spreadsheet pivot table shows a total at the bottom of each group and a grand total at the very end.

You could get there by writing several separate queries and `UNION ALL`-ing them together, but Oracle gives you three extensions to `GROUP BY` that do this in a single query:

- `ROLLUP`
- `CUBE`
- The `GROUPING` function


### ROLLUP: Subtotals Up a Hierarchy

`ROLLUP` produces subtotal rows as it "rolls up" through the columns you list, in order, finishing with a grand total row.

> Think of it as, "totals per innermost group, then totals per next group up, then one overall total."

```sql
SELECT
  section_id,
  grade_type_code,
  AVG(numeric_grade) AS avg_grade
FROM grade
GROUP BY
    ROLLUP(section_id, grade_type_code)
ORDER BY
    section_id,
    grade_type_code;
```

This returns:

1. One row per `(section_id, grade_type_code)` combination
2. A subtotal row per `section_id` (with `grade_type_code` as `NULL`)
3. A single grand-total row (with both columns `NULL`)

The column order inside `ROLLUP()` matters! It defines the hierarchy being subtotaled, from left to right.

:::{.callout-tip collapse=false}
#### Only "Rolling Up" Some Totals

You can also do partial `ROLLUP`'s. In the example below, we only roll up the instructor ID and the section ID. There won't be a grand total for capacity across all courses.

```sql
SELECT
    COURSE_NO,
    INSTRUCTOR_ID,
    SECTION_ID,
    SUM(CAPACITY)
FROM
    SECTION
GROUP BY
    COURSE_NO,
    ROLLUP(
        INSTRUCTOR_ID,
        SECTION_ID
    )
ORDER BY
    COURSE_NO,
    INSTRUCTOR_ID,
    SECTION_ID
```
:::

### CUBE: Every Combination of Subtotals

`CUBE` goes further: it produces subtotals for *every possible combination* of the grouping columns, not just a strict hierarchy.

```sql
SELECT
  section_id,
  grade_type_code,
  AVG(numeric_grade) AS avg_grade
FROM grade
GROUP BY
  CUBE(section_id, grade_type_code)
ORDER BY
  section_id,
  grade_type_code;
```

With two columns, `CUBE` gives you:

1. Detail rows (subtotals for specific values across all groups)
2. Subtotals by `section_id` alone
3. Subtotals by `grade_type_code` alone
4. The grand total

You get four levels of summary instead of `ROLLUP`'s three.

> As you add more columns, `CUBE` grows quickly ($2^n$ combinations for $n$ columns), so it's best reserved for a handful of grouping columns at a time.

### Telling Subtotal Rows Apart with GROUPING

Both `ROLLUP` and `CUBE` mark subtotal rows by putting `NULL` in the columns that were "rolled up." But what if your data might actually contain `NULL` values?

> In other words, how do you tell a real `NULL` apart from a subtotal row?

The `GROUPING()` function solves this. For a given column, it returns:

- `1` on subtotal/grand-total rows (where that column was aggregated away)
- `0` on normal detail rows.

```sql
SELECT section_id,
       grade_type_code,
       AVG(numeric_grade) AS avg_grade,
       GROUPING(section_id) AS is_section_subtotal,
       GROUPING(grade_type_code) AS is_type_subtotal
FROM grade
GROUP BY
  ROLLUP(section_id, grade_type_code);
```

This is especially handy paired with `DECODE` or `CASE`. Use it to relabel subtotal rows with something more readable than `NULL`. For example:

```sql
SELECT 
  DECODE(GROUPING(grade_type_code), 1, 'All Types', grade_type_code)
  ...
``` 



## Exercises

Using `ENROLLMENT`, `SECTION`, and `COURSE`:

1. Write a query that shows the number of enrollments per `section_id` within each `course_no`, with a subtotal per course and a grand total. Which clause do you need?
2. Modify your query to replace the `NULL` placeholders in subtotal rows with the label `'Course Total'` and `'Grand Total'`, using `GROUPING`.
3. Suppose a manager wants enrollment counts broken out by `course_no` alone, by `section_id` alone, and by both together — all in one result set. Which clause fits best, and why wouldn't `ROLLUP` be sufficient here?
4. Pivot the student count by `STATE` (from `ZIPCODE`) into separate columns for GA and AL.

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

- Why might a report writer prefer `ROLLUP` over `CUBE` even when both would technically answer the question?
- If a `GROUP BY ROLLUP(a, b)` query returns a row where both `a` and `b` are `NULL`, what does that row represent?
- What happens to query cost/performance as you add more columns to a `CUBE`? Why?

## Additional Resources

### Further Reading

- Oracle SQL Language Reference, [*ROLLUP, CUBE, and GROUPING*](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SELECT.html)
- Oracle SQL by Example ([Rischert 2009](references.html#ref-rischert09)), Chapter 6

## Answers

1. `GROUP BY ROLLUP(c.course_no, s.section_id)` rolls up section-level detail into course-level subtotals and then a grand total.
2. Wrap each grouped column: `DECODE(GROUPING(s.section_id), 1, 'Course Total', TO_CHAR(s.section_id))`, and similarly for `course_no` with `'Grand Total'` where both `GROUPING` values equal 1.
3. `CUBE(c.course_no, s.section_id)` — it produces subtotals for course alone, section alone, and both together, whereas `ROLLUP` only produces one direction of subtotal (course, then course+section), not section alone.
4.

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