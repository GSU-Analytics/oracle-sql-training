# Date and Time Logic

## Module Introduction

This module explores date and time functions in Oracle SQL. Learners will practice extracting, formatting, converting, and comparing date values using the `STUDENT` schema. Mastery of these functions is essential for time-based reporting and analytics.

## Explanation

### SYSDATE, TRUNC, EXTRACT

* `SYSDATE` returns the current date and time from the database server.
* `TRUNC(date)` removes the time portion of a date, useful for daily comparisons.
* `EXTRACT` retrieves a specific component (e.g., YEAR, MONTH) from a date.

**Example 1: Current date and time**

```sql
SELECT SYSDATE FROM dual;
```

This query returns the current date and time from the database server.

**Example 2: Strip time from a section start date**

```sql
SELECT TRUNC(start_date_time) FROM section;
```

This query removes the time component from section start dates, leaving only the date portion.

**Example 3: Extract year from section start date**

```sql
SELECT EXTRACT(YEAR FROM start_date_time) FROM section;
```

This query extracts only the year component from section start dates.

**Reference**: Lab 5.1–5.2

### TO_DATE, TO_CHAR, Time Zones

* `TO_DATE` converts a string to a date using a format mask.
* `TO_CHAR(date, format)` converts a date to a formatted string.
* Oracle supports multiple timestamp types including those with time zones.

**Example 4: Parse date string**

```sql
SELECT TO_DATE('01-JAN-2015', 'DD-MON-YYYY') FROM dual;
```

This query converts a string literal into a proper date value using a format mask.

**Example 5: Format a student registration date**

```sql
SELECT TO_CHAR(registration_date, 'MM/DD/YYYY') FROM student;
```

This query formats registration dates as MM/DD/YYYY strings for display.

**Example 6: Display session and database time zones**

```sql
SELECT SESSIONTIMEZONE, DBTIMEZONE FROM dual;
```

This query shows the current session and database timezone settings.

**Reference**: Lab 5.2–5.3

## Exercises

1. **Current Date Test**
   Write a query to return the current date and time using `SYSDATE`. Use the `dual` table.

2. **Date Without Time**
   From the `SECTION` table, return `section_id`, `start_date_time`, and the truncated version of `start_date_time` with no time.

3. **Extract Year and Month**
   From the `SECTION` table, display `section_id`, and extract the `YEAR` and `MONTH` from `start_date_time`.

4. **Filter Students by Registration Year**
   List all students from the `STUDENT` table who registered in 2012. Use `EXTRACT`.

5. **Formatted Dates**
   Display each `STUDENT_ID` and the registration date formatted as `Month DD, YYYY`.

6. **Compare Dates**
   Show all sections from the `SECTION` table that started before January 1, 2015. Use `TO_DATE` for comparison.

## Q&A

* When should you use `TRUNC` vs. `TO_CHAR` for removing the time portion of a date?
* What are the differences between `SYSDATE`, `CURRENT_DATE`, and `SYSTIMESTAMP`?
* Why is explicit date conversion with `TO_DATE` preferred over implicit conversions?
* In what types of queries would `EXTRACT` be preferred over formatting functions?
* How does timezone affect date values in distributed systems?
* What are common pitfalls when working with date ranges in WHERE clauses?
* How do you handle date arithmetic for business days vs calendar days?

## Additional Resources

* Oracle SQL by Example, Chapter 5: Labs 5.1–5.3
* Oracle Documentation: Date and Time Functions
* Oracle Database Globalization Support Guide

## Answers

**1.**

```sql
SELECT SYSDATE FROM dual;
```

**2.**

```sql
SELECT section_id, start_date_time, TRUNC(start_date_time) AS start_date
FROM section;
```

**3.**

```sql
SELECT section_id,
       EXTRACT(YEAR FROM start_date_time) AS year,
       EXTRACT(MONTH FROM start_date_time) AS month
FROM section;
```

**4.**

```sql
SELECT student_id, registration_date
FROM student
WHERE EXTRACT(YEAR FROM registration_date) = 2012;
```

**5.**

```sql
SELECT student_id,
       TO_CHAR(registration_date, 'Month DD, YYYY') AS formatted_date
FROM student;
```

**6.**

```sql
SELECT section_id, start_date_time
FROM section
WHERE start_date_time < TO_DATE('01-JAN-2015', 'DD-MON-YYYY');
```