# Single-Row Built-In Functions

## Module Introduction

In this module, students will explore single-row functions in Oracle SQL, focusing on character, number, conversion, and null-handling functions, as well as conditional logic and introductory regular expressions. These functions are foundational for data transformation and conditional logic in SELECT queries.

**Reference**: *Oracle SQL by Example (4th Edition)*, Chapter 4, pp. 133–187

## Explanation

### Character Functions

Character functions allow manipulation and evaluation of string values.

Examples:

```sql
SELECT student_id, 
       UPPER(first_name) AS upper_name, 
       LENGTH(last_name) AS name_length
FROM student;
```

|STUDENT_ID|UPPER_NAME|NAME_LENGTH|
|----------|----------|-----------|
|360       |CALVIN    |6          |
|361       |RAWAN     |9          |
|362       |YU        |7          |
|363       |BRIDGET   |5          |
|364       |HOWARD    |6          |
|365       |KATHLEEN  |10         |
|...       |...  |...         |

This query demonstrates converting names to uppercase and calculating the length of last names.

Common functions: `UPPER`, `LOWER`, `INITCAP`, `SUBSTR`, `INSTR`, `TRIM`, `LPAD`, `RPAD`, `LENGTH`

Refer to Table 4.2, **Which Character Function Should You Use?** in Lab 4.1 for a full list and description of character functions.

### Number Functions

Number functions perform arithmetic or rounding operations.

Example:

```sql
SELECT course_no, 
       ROUND(cost, -2) AS rounded_cost
FROM course;
```

|COURSE_NO|ROUNDED_COST|
|---------|------------|
|10       |1200        |
|20       |1200        |
|25       |1200        |
|80       |1600        |
|...       |...        |

This query rounds course costs to the nearest hundred dollars.

Common functions: `ROUND`, `TRUNC`, `MOD`, `FLOOR`, `CEIL`, `ABS`, `SIGN`, `POWER`, `SQRT`, `EXP`, `LOG`

Refer to Table 4.3, **Which Number Function Should You Use?** in Lab 4.2 for a full list and description of number functions.

### Conversion Functions

Used to convert data types between strings, numbers, and dates.

Example:

```sql
SELECT TO_CHAR(registration_date, 'YYYY-MM-DD') AS reg_date_str
FROM student;
```

|REG_DATE_STR|
|------------|
|2007-02-21  |
|2007-02-21  |
|2007-02-21  |
|2007-02-21  |
|...|

This query converts registration dates to a standardized string format.

Common functions: `TO_CHAR`, `TO_DATE`, `CAST`, `TO_NUMBER`

### Null Handling

These functions help manage NULL values effectively.

Example:

```sql
SELECT NVL(phone,'No phone') AS contact_number
  FROM student
 ORDER BY student_id;
```

|CONTACT_NUMBER|
|--------------|
|... |
|201-555-5555  |
|718-555-5555  |
|617-555-5555  |
|718-555-5555  |
|718-555-5555  |
|No phone      |
|... |

This query replaces NULL phone numbers with a default message.

Common functions: `NVL`, `NVL2`, `COALESCE`, `NULLIF`, `LNNVL`

### Conditional Logic

SQL offers conditional functions like `CASE` and `DECODE` for inline logic.

Example:

```sql
SELECT student_id,
       zip,
       CASE WHEN zip <= '10025' THEN 'Downtown'
            WHEN zip >= '11216' THEN 'Northside'
            ELSE 'Other'
       END AS region
FROM student;
```

|STUDENT_ID|ZIP  |REGION   |
|----------|-----|---------|
|396       |01247|Downtown |
|...       |...|... |
|223       |10025|Downtown |
|145       |10048|Other    |
|...       |...|... |
|266       |11215|Other    |
|321       |11216|Northside|
|...       |...|... |
|238       |48104|Northside|


This query assigns region names based on ZIP codes using CASE logic.

Alternative using `DECODE`:

```sql
SELECT DECODE(zip, '30303', 'Downtown', '30342', 'Northside', 'Other') AS region
FROM student;
```

This query achieves the same result using Oracle's DECODE function.

Refer to Table 4.4, **Which Functions and CASE Expressions Should You Use?** in Lab 4.3 for a full list and description of conditional functions.

### String Manipulation Functions

This category includes concatenation, replacement, and regex-based transformations.

**Concatenation**:

```sql
SELECT student_id, 
       first_name || ' ' || last_name AS full_name
FROM student;
```

|STUDENT_ID|FULL_NAME             |
|----------|----------------------|
|360       |Calvin Kiraly         |
|361       |Rawan Rosenberg       |
|362       |Yu Sentell            |
|...       |... |

This query combines first and last names with a space separator.

**REPLACE**:

```sql
SELECT student_id, 
       REPLACE(phone, '-', '') AS phone_cleaned
FROM student;
```

|STUDENT_ID|PHONE_CLEANED|
|----------|-------------|
|360       |2035555555   |
|361       |7185555555   |
|362       |7185555555   |
|...       |...   |

This query removes dashes from phone numbers.

**REGEXP_REPLACE**:

```sql
SELECT student_id, 
       REGEXP_REPLACE(phone, '[^0-9]', '') AS phone_digits_only
FROM student;
```

|STUDENT_ID|PHONE_DIGITS_ONLY|
|----------|-----------------|
|360       |2035555555       |
|361       |7185555555       |
|...       |...       |

This query extracts only numeric digits from phone numbers using regular expressions.

Other regex functions: `REGEXP_LIKE`, `REGEXP_INSTR`, `REGEXP_SUBSTR`

Refer to Lab 16.1, **Regular Expressions** for a detailed guide to using regular expressions.

### Regular Expressions

Regular expressions provide powerful pattern matching capabilities for complex string operations.

```sql
SELECT zip
FROM zipcode
WHERE REGEXP_LIKE(zip, '[^0-9]');
```

This query finds ZIP codes that contain non-numeric characters.

> Regular expressions can get very complicated, but they are essentially a form a pattern matching in text. See [Stanford's CS103 Notes](https://web.stanford.edu/class/cs103/guide_to_regexes) for a high-level overview.

## Exercises

Use the following prompts to reinforce your understanding of single-row functions. These exercises reference the STUDENT, COURSE, and ZIPCODE tables.

1. **Search for student last names beginning with 'Mo'**
   Use the STUDENT table.

2. **Find student names containing a period and order results by last name length**
   Use the STUDENT table.

3. **Format the CITY, STATE, and ZIP columns into a single readable address line separated by commas**
   Use the ZIPCODE table.

4. **Display the course costs increased by 75% and round to the nearest dollar**
   Use the COST column in the COURSE table.

5. **Custom output for prerequisite courses based on conditional logic**
   Write a query that selects the course number, description, and prerequisite from the COURSE table for the following course numbers: 20, 120, 122, and 132. Alias the prerequisite column as "Original". Then, use a CASE statement to create a new column with customized output: if the prerequisite is course number 120, substitute it with '200'; if the prerequisite is 130, substitute with 'N/A'; if the prerequisite is null, display 'None'. For all other prerequisites, display the original value as a character. Sort the results by course number in descending order.

6. **Find zip codes that contain non-digit characters**
   Use a regular expression function to identify all rows in the ZIPCODE table where the ZIP column contains any characters that are not numeric digits.

## Q&A

Open floor for questions and best practices discussion. Consider these topics:

* When to use character functions vs. regular expressions for pattern matching
* Best practices for handling NULL values in reporting queries
* Performance considerations when using functions in WHERE clauses
* Common pitfalls when converting between data types

## Additional Resources

* [Oracle Documentation on SQL Functions](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/SQL-Functions.html)
* Practice more with the STUDENT schema using Labs 4.1–4.4 in the textbook

## Answers

1. Search for student last names beginning with 'Mo'

```sql
SELECT first_name, last_name
FROM student
WHERE SUBSTR(last_name, 1, 2) = 'Mo';
```

2. Find student names containing a period and order results by last name length

```sql
SELECT first_name, last_name
FROM student
WHERE INSTR(first_name, '.') > 0
ORDER BY LENGTH(last_name);
```

3. Format the city, state, and zip columns into a single readable address line separated by commas

```sql
SELECT city || ', ' || state || ', ' || zip AS formatted_address
FROM zipcode;
```

4. Display the course costs increased by 75% and round to the nearest dollar

```sql
SELECT cost, 
       cost * 1.75 AS increased,
       ROUND(cost * 1.75) AS rounded_increased
FROM course;
```

5. Custom output for prerequisite courses based on conditional logic

```sql
SELECT course_no, 
       description, 
       prerequisite AS "Original",
       CASE WHEN prerequisite = 120 THEN '200'
            WHEN prerequisite = 130 THEN 'N/A'
            WHEN prerequisite IS NULL THEN 'None'
            ELSE TO_CHAR(prerequisite)
       END AS "NEW"
FROM course
WHERE course_no IN (20, 120, 122, 132)
ORDER BY course_no DESC;
```

6. Find zip codes that contain non-digit characters

```sql
SELECT zip
FROM zipcode
WHERE REGEXP_LIKE(zip, '[^0-9]');
```