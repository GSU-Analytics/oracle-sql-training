# Advanced Aggregations

## Module Introduction

`GROUP BY` gets you a single level of summary — one row per group. But reporting often calls for *subtotals and grand totals alongside the detail*, the way a spreadsheet pivot table shows a total at the bottom of each group and a grand total at the very end.

You could get there by writing several separate queries and `UNION ALL`-ing them together, but Oracle gives you three extensions to `GROUP BY` that do this in a single query: `ROLLUP`, `CUBE`, and the `GROUPING` function. We'll also talk about using `PIVOT` to reshape your output.

**Reference**: Oracle SQL by Example, Chapter 6 · Oracle SQL Language Reference, *Aggregate and Group Functions*

## Explanation

### Queries with Subtotals

#### ROLLUP: Subtotals Up a Hierarchy

`ROLLUP` produces subtotal rows as it "rolls up" through the columns you list, in order, finishing with a grand total row. Think of it as: totals per innermost group, then totals per next group up, then one overall total.

```sql
SELECT section_id, grade_type_code, AVG(numeric_grade) AS avg_grade
FROM grade
GROUP BY ROLLUP(section_id, grade_type_code)
ORDER BY section_id, grade_type_code;
```

This returns one row per `(section_id, grade_type_code)` combination, plus a subtotal row per `section_id` (with `grade_type_code` as `NULL`), plus a single grand-total row (with both columns `NULL`). The column order inside `ROLLUP()` matters — it defines the hierarchy being subtotaled, from left to right.

#### CUBE: Every Combination of Subtotals

`ROLLUP` only rolls up one direction, following the order you gave it. `CUBE` goes further: it produces subtotals for *every possible combination* of the grouping columns, not just a strict hierarchy.

```sql
SELECT section_id, grade_type_code, AVG(numeric_grade) AS avg_grade
FROM grade
GROUP BY CUBE(section_id, grade_type_code)
ORDER BY section_id, grade_type_code;
```

With two columns, `CUBE` gives you: detail rows, subtotals by `section_id` alone, subtotals by `grade_type_code` alone, *and* the grand total — four levels of summary instead of `ROLLUP`'s three. As you add more columns, `CUBE` grows quickly (2ⁿ combinations for n columns), so it's best reserved for a handful of grouping columns at a time.

#### Telling Subtotal Rows Apart with GROUPING

Both `ROLLUP` and `CUBE` mark subtotal rows by putting `NULL` in the columns that were "rolled up." That's a problem the moment one of your columns can *legitimately* contain `NULL` on its own — how do you tell a real `NULL` apart from a subtotal row?

The `GROUPING()` function solves this. For a given column, it returns `1` on subtotal/grand-total rows (where that column was aggregated away) and `0` on normal detail rows.

```sql
SELECT section_id,
       grade_type_code,
       AVG(numeric_grade) AS avg_grade,
       GROUPING(section_id) AS is_section_subtotal,
       GROUPING(grade_type_code) AS is_type_subtotal
FROM grade
GROUP BY ROLLUP(section_id, grade_type_code);
```

This is especially handy paired with `DECODE` or `CASE` to relabel subtotal rows with something more readable than `NULL`, e.g. `DECODE(GROUPING(grade_type_code), 1, 'All Types', grade_type_code)`.


Both `ROLLUP` and `CUBE` mark subtotal rows by putting `NULL` in the columns that were "rolled up." That's a problem the moment one of your columns can *legitimately* contain `NULL` on its own — how do you tell a real `NULL` apart from a subtotal row?

The `GROUPING()` function solves this. For a given column, it returns `1` on subtotal/grand-total rows (where that column was aggregated away) and `0` on normal detail rows.

```sql
SELECT c.description,
       s.section_id,
       COUNT(e.student_id) AS num_enrolled,
       GROUPING(c.description) AS is_course_subtotal,
       GROUPING(s.section_id) AS is_section_subtotal
FROM course c
JOIN section s ON c.course_no = s.course_no
JOIN enrollment e ON s.section_id = e.section_id
GROUP BY ROLLUP(c.description, s.section_id);
```

This is especially handy paired with `DECODE` or `CASE` to relabel subtotal rows with something more readable than `NULL`, e.g. `DECODE(GROUPING(s.section_id), 1, 'All Sections', s.section_id)`.

#### Choosing Between Them

- Need a running set of subtotals building toward one grand total, in a natural hierarchy (e.g., course → section)? Use **`ROLLUP`**.
- Need subtotals sliced every possible way across a small number of columns (e.g., by course *and* separately by term)? Use **`CUBE`**.
- Need to identify which rows are subtotals — especially to relabel them or filter them out? Use **`GROUPING`**.

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



## Exercises

Using `ENROLLMENT`, `SECTION`, and `COURSE`:

1. Write a query that shows the number of enrollments per `section_id` within each `course_no`, with a subtotal per course and a grand total. Which clause do you need?
2. Modify your query to replace the `NULL` placeholders in subtotal rows with the label `'Course Total'` and `'Grand Total'`, using `GROUPING`.
3. Suppose a manager wants enrollment counts broken out by `course_no` alone, by `section_id` alone, and by both together — all in one result set. Which clause fits best, and why wouldn't `ROLLUP` be sufficient here?
4. Pivot the student count by `STATE` (from `ZIPCODE`) into separate columns for GA and AL.

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