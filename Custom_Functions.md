# Custom Functions

## Module Introduction

This module introduces custom PL/SQL functions in Oracle SQL. You'll learn how to write, understand, and use reusable logic encapsulated in user-defined functions. These functions simplify SQL statements and are especially useful in reporting tasks that involve business rules.

## Explanation

### Custom Function Conceptual Overview

Custom functions in Oracle are reusable blocks of PL/SQL logic that return a single value. They're used in `SELECT`, `WHERE`, and `ORDER BY` clauses just like built-in functions.

**Benefits:**

* Reuse across queries and reports
* Clean, readable SQL
* Easier updates to business logic

**Example Use Case**: Calculate the next business day based on a given date and skip holidays stored in the `HOLIDAY` table.

**Reference**: Chapter 17

### Custom Function Metadata

Functions have:

* **Input Parameters**: Data sent to the function.
* **Return Value**: A single result.
* **Local Variables**: Temporary values used in the function.
* **Body**: The logic executed when the function is called.

**Example**: Define a custom function `next_business_day`:

```sql
CREATE OR REPLACE FUNCTION next_business_day(i_date DATE)
RETURN DATE IS
  v_date DATE;
BEGIN
  v_date := i_date;
  SELECT NVL(MAX(holiday_end_date)+1, v_date)
  INTO v_date
  FROM holiday
  WHERE v_date BETWEEN holiday_start_date AND holiday_end_date;
  RETURN v_date;
END;
```

This function returns the next business day, skipping holidays found in the holiday table.

### Usage

Custom functions are called like any other function:

```sql
SELECT student_id, next_business_day(ENROLL_DATE) AS next_day
FROM enrollment;
```

This query applies the custom function to each enrollment date to find the next business day.

**Output**:

```
STUDENT_ID   NEXT_DAY
-----------  ---------
        102  09-AUG-10
```

You can also use them in `WHERE`, `ORDER BY`, and even `JOIN` conditions.

## Exercises

1. **Call a Custom Function**
   Use the function `next_business_day` to find the next business day after each enrollment in the `ENROLLMENT` table. Show `STUDENT_ID`, `SECTION_ID`, `ENROLL_DATE`, and the next business day.

2. **Filter with a Custom Function**
   Select rows from `ENROLLMENT` where the next business day is more than two days after `ENROLL_DATE`.

3. **Custom Function with `TO_CHAR`**
   Return the weekday of the `next_business_day` for each `ENROLL_DATE` using `TO_CHAR`.

4. **Join with a Custom Function**
   Join `ENROLLMENT` and `SECTION` on `SECTION_ID`, and use `next_business_day(ENROLL_DATE)` to show when each student is expected to start.

5. **Use in Subquery**
   Select students whose `ENROLL_DATE` is not on a business day by comparing it to `next_business_day(ENROLL_DATE)`.

6. **Aggregate with Custom Function**
   Count how many enrollments fall on holidays by checking where `ENROLL_DATE != next_business_day(ENROLL_DATE)`.

## Q&A

* Why are custom functions useful for reporting?
* What kind of logic should be placed in a custom function vs a view?
* How does using a custom function affect query performance?
* What are some limitations of using custom functions in `WHERE` clauses?
* When should you create a custom function versus using existing Oracle functions?
* How do you handle error conditions within custom functions?
* What are the security considerations when creating custom functions?

## Additional Resources

* *Oracle SQL by Example*, Chapter 17 – "Creating Your Own Custom Function"
* Oracle SQL Language Reference: [PL/SQL Function Syntax](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/)
* View metadata: `USER_OBJECTS`, `USER_SOURCE`

## Answers

**1.**

```sql
SELECT student_id, section_id, enroll_date, next_business_day(enroll_date) AS next_day
FROM enrollment;
```

**2.**

```sql
SELECT student_id, section_id, enroll_date
FROM enrollment
WHERE next_business_day(enroll_date) > enroll_date + 2;
```

**3.**

```sql
SELECT student_id, TO_CHAR(next_business_day(enroll_date), 'Day') AS business_day
FROM enrollment;
```

**4.**

```sql
SELECT e.student_id, s.section_id, s.start_date_time, next_business_day(e.enroll_date) AS start_expected
FROM enrollment e
JOIN section s ON e.section_id = s.section_id;
```

**5.**

```sql
SELECT student_id
FROM enrollment
WHERE enroll_date != next_business_day(enroll_date);
```

**6.**

```sql
SELECT COUNT(*) AS holiday_enrollments
FROM enrollment
WHERE enroll_date != next_business_day(enroll_date);
```