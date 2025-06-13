# The LISTAGG Function {.unnumbered}

## Module Introduction

This mini-module introduces the LISTAGG function, an aggregate function that concatenates values from multiple rows into a single comma-separated string. This function is particularly useful for creating readable summaries when you need to combine related text values from grouped data. Students will practice using LISTAGG with the STUDENT schema tables to create meaningful concatenated results.

## Explanation

### What is LISTAGG?

LISTAGG is an aggregate function that concatenates values from multiple rows into a single string, with a specified delimiter separating each value. Unlike other aggregate functions that return numeric summaries, LISTAGG returns a text result containing all the grouped values.

**Basic Syntax:**
```sql
LISTAGG(column, 'delimiter') WITHIN GROUP (ORDER BY column)
```

**Example 1: Basic LISTAGG usage**

```sql
SELECT instructor_id,
       LISTAGG(section_id, ', ') WITHIN GROUP (ORDER BY section_id) AS sections_taught
FROM section
GROUP BY instructor_id;
```

This query groups sections by instructor and creates a comma-separated list of all section IDs each instructor teaches.

**Example 2: LISTAGG with meaningful data**

```sql
SELECT c.description AS course_name,
       LISTAGG(s.section_id, ', ') WITHIN GROUP (ORDER BY s.section_id) AS available_sections
FROM course c
JOIN section s ON c.course_no = s.course_no
GROUP BY c.description;
```

This query shows course names with a list of all available section IDs for each course.

### LISTAGG with Different Delimiters

You can use any delimiter, not just commas. This allows for flexible formatting of the concatenated results.

**Example 3: Using different delimiters**

```sql
SELECT location,
       LISTAGG(section_id, ' | ') WITHIN GROUP (ORDER BY section_id) AS sections
FROM section
GROUP BY location;
```

This query uses the pipe symbol (|) as a delimiter to separate section IDs by location.

**Example 4: LISTAGG with student enrollment information**

```sql
SELECT s.first_name || ' ' || s.last_name AS student_name,
       LISTAGG(c.description, '; ') WITHIN GROUP (ORDER BY c.description) AS enrolled_courses
FROM student s
JOIN enrollment e ON s.student_id = e.student_id
JOIN section sec ON e.section_id = sec.section_id  
JOIN course c ON sec.course_no = c.course_no
GROUP BY s.student_id, s.first_name, s.last_name;
```

This query shows each student with a semicolon-separated list of all courses they're enrolled in.

## Exercises

1. Write a query using LISTAGG to show each location with a comma-separated list of all instructor IDs who teach in that location.

2. Create a query that displays each course number with a pipe-separated (|) list of all students enrolled in any section of that course. Include both first and last names and use ON OVERFLOW TRUNCATE to handle potential length issues.

3. Use LISTAGG to show each instructor ID with a semicolon-separated list of all course descriptions they teach.

4. Write a query that groups students by their last name initial (first letter) and shows a comma-separated list of their full names.

5. Create a query using LISTAGG to show each grade type with a list of all students who received that grade type, formatted as "Last, First". Important: Use ON OVERFLOW TRUNCATE clause to handle cases where there are many students with the same grade type.

6. Use LISTAGG to display each zip code with a comma-separated list of all cities in that zip code area from the zipcode table.

## Q&A

Some conversation starters for the Q&A session:

* When would LISTAGG be more appropriate than using traditional concatenation with the `||` operator?
* How does LISTAGG handle NULL values in the data being concatenated?
* What are the performance considerations when using LISTAGG with large datasets?
* How does the WITHIN GROUP (ORDER BY) clause affect the final result of LISTAGG?
* What happens if the concatenated result exceeds Oracle's string length limits?
* Can you use LISTAGG with DISTINCT to remove duplicate values before concatenation?
* How would you handle cases where the delimiter character appears within the data being concatenated?
* What are some real-world business scenarios where LISTAGG would be particularly useful?

## Additional Resources

* Oracle SQL by Example, Chapter 6: Lab 6.1-6.2 (Aggregate Functions and GROUP BY)
* Oracle SQL by Example, Chapter 17: Lab 17.1 (Advanced SQL Concepts and Analytical Functions)
* Oracle SQL by Example, Table 6.1 (Commonly Used Aggregate Functions)
* Oracle SQL by Example, Chapter 4: Lab 4.1 (Character Functions and Concatenation)
* Oracle Documentation: SQL Language Reference - Aggregate Functions

## Answers

**1.**

```sql
SELECT location,
       LISTAGG(instructor_id, ', ') WITHIN GROUP (ORDER BY instructor_id) AS instructors
FROM section
GROUP BY location;
```

**2.**

```sql
SELECT c.course_no,
       LISTAGG(s.first_name || ' ' || s.last_name, ' | ' 
               ON OVERFLOW TRUNCATE '...' WITH COUNT) 
       WITHIN GROUP (ORDER BY s.last_name, s.first_name) AS enrolled_students
FROM course c
JOIN section sec ON c.course_no = sec.course_no
JOIN enrollment e ON sec.section_id = e.section_id
JOIN student s ON e.student_id = s.student_id
GROUP BY c.course_no;
```

**3.**

```sql
SELECT i.instructor_id,
       LISTAGG(c.description, '; ') WITHIN GROUP (ORDER BY c.description) AS courses_taught
FROM instructor i
JOIN section s ON i.instructor_id = s.instructor_id
JOIN course c ON s.course_no = c.course_no
GROUP BY i.instructor_id;
```

**4.**

```sql
SELECT SUBSTR(last_name, 1, 1) AS last_name_initial,
       LISTAGG(first_name || ' ' || last_name, ', ') 
       WITHIN GROUP (ORDER BY last_name, first_name) AS students
FROM student
GROUP BY SUBSTR(last_name, 1, 1);
```

**5.**

```sql
SELECT g.grade_type_code,
       LISTAGG(s.last_name || ', ' || s.first_name, ', ' 
               ON OVERFLOW TRUNCATE '...' WITH COUNT) 
       WITHIN GROUP (ORDER BY s.last_name, s.first_name) AS students_with_grade
FROM grade g
JOIN enrollment e ON g.section_id = e.section_id AND g.student_id = e.student_id
JOIN student s ON e.student_id = s.student_id
GROUP BY g.grade_type_code;
```

**6.**

```sql
SELECT zip,
       LISTAGG(city, ', ') WITHIN GROUP (ORDER BY city) AS cities
FROM zipcode
GROUP BY zip;
```