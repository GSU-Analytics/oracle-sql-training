# Reshaping With Pivots

## Module Introduction

This module illustrates how to pivot and un-pivot data within SQL. Students will learn how to expand their data across several columns, and how to stack existing columns into a narrow, tidy structure.

:::{.callout-caution collapse=false}
### Pivots Can Be A Pain
Pivots are a messy inclusion in Oracle.

1. They aren't standard SQL syntax (e.g. MySQL and PostgreSQL don't support it)
2. There are some surprising "gotchas" that can pop up in `PIVOT` clauses (e.g. there are limitations on using subqueries in `PIVOT` clauses)
3. They require a lot of explicit typing—for example, you often have to specify each column name yourself

We point it out here because it can be useful, but it proceed with caution.

- If you are going to pivot a table, it should probably the final step in your query.
- You may find that it is actually easier to join several queries together, or to self-join a table's columns onto itself.
- Also, consider whether you actually need to pivot your table. It may be better to export your data and pivot the result with a tool like Excel.
:::

**Reference**: Oracle Database documentation, [Pivot and Unpivot](https://blogs.oracle.com/sql/how-to-convert-rows-to-columns-and-back-again-with-sql-aka-pivot-and-unpivot)

## Explanation

### PIVOT

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

|MON|TUE|WED|THU|FRI|SAT|SUN|
|---|---|---|---|---|---|---|
|15|17|7|5|4|17|13|

This is a simple example, but it conveys the idea. See [Oracle's Documentation](https://blogs.oracle.com/sql/how-to-convert-rows-to-columns-and-back-again-with-sql-aka-pivot-and-unpivot) for more details on pivoting.

### UNPIVOT

Un-pivoting does the opposite of a pivot: it narrows your table and stacks records on top of each other, consolidating several fields into two (a key and a value).

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

## Q&A

Use the following questions to guide class discussion or individual reflection after completing the exercises:

1. Why might it be better to use joins than pivots?
2. Why might it be better to use pivots than joins?
3. How could you use UNIONS to reproduce pivot results? Why might you not want to?

