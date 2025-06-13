# SQL Extra Practice Exercises - STUDENT Schema

This document provides additional practice exercises organized by module topic, based on the STUDENT schema from Oracle SQL by Example. Each section includes exercise descriptions with table and column information, followed by answer solutions.

## Single Table Review

Practice working with data from a single table using `SELECT`, filtering, and sorting operations.

**Tables Used:** STUDENT, COURSE, SECTION, INSTRUCTOR

### Exercises

**1. Basic SELECT and Filtering**
Using the STUDENT table (columns: STUDENT_ID, FIRST_NAME, LAST_NAME, STREET_ADDRESS, ZIP, PHONE, EMPLOYER, REGISTRATION_DATE), write a query to display all students who registered after January 1, 2023. Show student ID, first name, last name, and registration date.

**2. Sorting and Aliases**
Using the COURSE table (columns: COURSE_NO, DESCRIPTION, COST, PREREQUISITE), create a query that shows all courses with their descriptions and costs, ordered by cost from highest to lowest. Use meaningful column aliases.

**3. Pattern Matching**
Using the STUDENT table, find all students whose first name starts with 'J' and display their full name (concatenated), phone number, and employer. Handle cases where phone might be NULL.

**4. Date Filtering**
Using the SECTION table (columns: SECTION_ID, COURSE_NO, SECTION_NO, START_DATE_TIME, LOCATION, INSTRUCTOR_ID), show all sections that start in the current year. Display section ID, location, and formatted start date.

### Answers

**1.**
```sql
SELECT student_id, first_name, last_name, registration_date
FROM student
WHERE registration_date > DATE '2023-01-01'
ORDER BY registration_date;
```

**2.**
```sql
SELECT course_no AS "Course Number",
       description AS "Course Title",
       cost AS "Tuition Cost"
FROM course
ORDER BY cost DESC NULLS LAST;
```

**3.**
```sql
SELECT first_name || ' ' || last_name AS "Full Name",
       NVL(phone, 'No Phone Listed') AS "Phone Number",
       employer
FROM student
WHERE first_name LIKE 'J%';
```

**4.**
```sql
SELECT section_id,
       location,
       TO_CHAR(start_date_time, 'MM/DD/YYYY HH24:MI') AS "Start Time"
FROM section
WHERE EXTRACT(YEAR FROM start_date_time) = EXTRACT(YEAR FROM SYSDATE);
```

## Single Row Functions

Demonstrates how to use built-in functions to manipulate individual rows of data including character, number, date, and conversion functions.

**Tables Used:** STUDENT, INSTRUCTOR, ZIPCODE

### Exercises

**1. Character Functions**
Using the STUDENT table, create a query that displays student names in proper case (first letter capitalized, rest lowercase), email addresses (create from first name + last name + '@student.edu'), and the length of their street address. Only show students with street addresses longer than 20 characters.

**2. Number Functions**
Using the COURSE table (columns: COURSE_NO, DESCRIPTION, COST, PREREQUISITE), create a query that displays course information with costs rounded to the nearest hundred dollars, calculates a 15% discount amount rounded to 2 decimal places, and shows the absolute difference between the cost and $1000. Only show courses where the cost is not NULL.

**3. Date Functions**
Using the STUDENT table, calculate how many days each student has been registered and determine what day of the week they registered. Show student ID, name, registration date, days registered, and day of week.

**4. Conditional Logic**
Using the STUDENT table, categorize students by their ZIP code into regions. Use CASE to assign 'Metro' for ZIP codes starting with '1', 'Southeast' for ZIP codes starting with '2' or '3', and 'Other' for all else. Include student name and phone.

### Answers

**1.**
```sql
SELECT INITCAP(first_name) || ' ' || INITCAP(last_name) AS "Student Name",
       LOWER(first_name) || '.' || LOWER(last_name) || '@student.edu' AS "Email",
       LENGTH(street_address) AS "Address Length"
FROM student
WHERE LENGTH(street_address) > 20;
```

**2.**
```sql
SELECT course_no,
       description,
       ROUND(cost, -2) AS "Rounded Cost",
       ROUND(cost * 0.15, 2) AS "15% Discount",
       ABS(cost - 1000) AS "Difference from $1000"
FROM course
WHERE cost IS NOT NULL;
```

**3.**
```sql
SELECT student_id,
       first_name || ' ' || last_name AS "Name",
       registration_date,
       TRUNC(SYSDATE - registration_date) AS "Days Registered",
       TO_CHAR(registration_date, 'Day') AS "Day of Week"
FROM student;
```

**4.**
```sql
SELECT first_name || ' ' || last_name AS "Student Name",
       phone,
       CASE 
         WHEN zip LIKE '1%' THEN 'Metro'
         WHEN zip LIKE '2%' OR zip LIKE '3%' THEN 'Southeast'
         ELSE 'Other'
       END AS "Region"
FROM student;
```

## Aggregate Queries

Summarizing data using aggregate functions, grouping results, and using HAVING clauses.

**Tables Used:** ENROLLMENT, GRADE, SECTION, COURSE

### Exercises

**1. Basic Aggregates**
Using the ENROLLMENT table (columns: STUDENT_ID, SECTION_ID, ENROLL_DATE, FINAL_GRADE), calculate the total number of enrollments, average final grade (excluding NULLs), highest and lowest final grades, and count of students with final grades.

**2. GROUP BY with Multiple Columns**
Using the GRADE table (columns: STUDENT_ID, SECTION_ID, GRADE_TYPE_CODE, GRADE_CODE_OCCURRENCE, NUMERIC_GRADE), show the average grade for each grade type per section. Display section ID, grade type, and average grade rounded to 1 decimal place.

**3. HAVING Clause**
Using the ENROLLMENT table joined with SECTION table, find sections that have more than 5 enrolled students. Show course number, section ID, location, and student count.

**4. Complex Aggregation**
Using the GRADE table, find the top 3 sections with the highest average numeric grades. Include section ID, average grade, and total number of grades recorded. Order by average grade descending.

### Answers

**1.**
```sql
SELECT COUNT(*) AS "Total Enrollments",
       ROUND(AVG(final_grade), 2) AS "Average Final Grade",
       MAX(final_grade) AS "Highest Grade",
       MIN(final_grade) AS "Lowest Grade",
       COUNT(final_grade) AS "Students with Grades"
FROM enrollment;
```

**2.**
```sql
SELECT section_id,
       grade_type_code,
       ROUND(AVG(numeric_grade), 1) AS "Average Grade"
FROM grade
GROUP BY section_id, grade_type_code
ORDER BY section_id, grade_type_code;
```

**3.**
```sql
SELECT s.course_no,
       s.section_id,
       s.location,
       COUNT(e.student_id) AS "Student Count"
FROM section s
JOIN enrollment e ON s.section_id = e.section_id
GROUP BY s.course_no, s.section_id, s.location
HAVING COUNT(e.student_id) > 5
ORDER BY COUNT(e.student_id) DESC;
```

**4.**
```sql
SELECT section_id,
       ROUND(AVG(numeric_grade), 2) AS "Average Grade",
       COUNT(*) AS "Total Grades"
FROM grade
GROUP BY section_id
ORDER BY AVG(numeric_grade) DESC
FETCH FIRST 3 ROWS ONLY;
```

## Basic Joins

Introduction to joining multiple tables using inner joins and outer joins to combine related data.

**Tables Used:** STUDENT, ZIPCODE, ENROLLMENT, SECTION, COURSE, INSTRUCTOR

### Exercises

**1. Simple Two-Table Join**
Join the STUDENT and ZIPCODE tables (ZIPCODE columns: ZIP, CITY, STATE) to show student names, their complete address (street, city, state), and phone numbers for all students in New York state.

**2. Three-Table Join**
Join STUDENT, ENROLLMENT, and SECTION tables to display student names, the course numbers they're enrolled in, section numbers, and locations. Sort by student last name.

**3. Join with Filtering**
Join SECTION, COURSE, and INSTRUCTOR tables (INSTRUCTOR columns: INSTRUCTOR_ID, FIRST_NAME, LAST_NAME) to show course descriptions, section locations, and instructor names for all courses that cost more than $1000.

**4. Outer Join Practice**
Use a LEFT OUTER JOIN between INSTRUCTOR and SECTION tables to show all instructors and the sections they teach (if any). Include instructor name and section ID. Show instructors even if they don't teach any sections.

### Answers

**1.**
```sql
SELECT s.first_name,
       s.last_name,
       s.street_address,
       z.city,
       z.state,
       s.phone
FROM student s
JOIN zipcode z ON s.zip = z.zip
WHERE z.state = 'NY'
ORDER BY s.last_name;
```

**2.**
```sql
SELECT st.first_name,
       st.last_name,
       sec.course_no,
       sec.section_no,
       sec.location
FROM student st
JOIN enrollment e ON st.student_id = e.student_id
JOIN section sec ON e.section_id = sec.section_id
ORDER BY st.last_name, st.first_name;
```

**3.**
```sql
SELECT c.description AS "Course",
       s.location AS "Location",
       i.first_name || ' ' || i.last_name AS "Instructor"
FROM course c
JOIN section s ON c.course_no = s.course_no
JOIN instructor i ON s.instructor_id = i.instructor_id
WHERE c.cost > 1000
ORDER BY c.description;
```

**4.**
```sql
SELECT i.first_name || ' ' || i.last_name AS "Instructor Name",
       s.section_id
FROM instructor i
LEFT OUTER JOIN section s ON i.instructor_id = s.instructor_id
ORDER BY i.last_name;
```

## Advanced Joins

Techniques for complex joins, including self-joins, cross joins, and set operations.

**Tables Used:** COURSE, STUDENT, INSTRUCTOR, SECTION

### Exercises

**1. Self-Join**
Using the COURSE table, perform a self-join to show each course along with its prerequisite course description. Display course number, course description, prerequisite course number, and prerequisite description. Only show courses that have prerequisites.

**2. Cartesian Product (Cross Join)**
Create a Cartesian product between a subset of STUDENT (first 3 students by ID) and a subset of COURSE (first 3 courses by course number) to show all possible student-course combinations. This simulates course shopping.

**3. Multiple Join Conditions**
Join STUDENT and INSTRUCTOR tables on ZIP code to find students and instructors who live in the same area. Show student name, instructor name, and the shared ZIP code.

**4. Complex Join with Aggregation**
Join ENROLLMENT, SECTION, and COURSE tables to find the total enrollment count for each course description. Show course description and total students enrolled across all sections of that course.

### Answers

**1.**
```sql
SELECT c1.course_no AS "Course Number",
       c1.description AS "Course Description",
       c1.prerequisite AS "Prerequisite Number",
       c2.description AS "Prerequisite Description"
FROM course c1
JOIN course c2 ON c1.prerequisite = c2.course_no
ORDER BY c1.course_no;
```

**2.**
```sql
SELECT s.student_id,
       s.first_name || ' ' || s.last_name AS "Student Name",
       c.course_no,
       c.description AS "Course"
FROM (SELECT * FROM student WHERE ROWNUM <= 3) s
CROSS JOIN (SELECT * FROM course WHERE ROWNUM <= 3) c
ORDER BY s.student_id, c.course_no;
```

**3.**
```sql
SELECT s.first_name || ' ' || s.last_name AS "Student",
       i.first_name || ' ' || i.last_name AS "Instructor",
       s.zip AS "Shared ZIP"
FROM student s
JOIN instructor i ON s.zip = i.zip
ORDER BY s.zip;
```

**4.**
```sql
SELECT c.description AS "Course Description",
       COUNT(e.student_id) AS "Total Enrolled"
FROM course c
JOIN section s ON c.course_no = s.course_no
JOIN enrollment e ON s.section_id = e.section_id
GROUP BY c.description
ORDER BY COUNT(e.student_id) DESC;
```

## Subqueries

Using subqueries to filter, aggregate, and manipulate data within queries including scalar, correlated, and inline views.

**Tables Used:** STUDENT, ENROLLMENT, SECTION, COURSE, GRADE

### Exercises

**1. Simple Subquery with IN**
Find all students who are enrolled in sections taught in location 'L211'. Use the STUDENT and ENROLLMENT tables with a subquery on SECTION table. Display student ID, name, and phone.

**2. Correlated Subquery**
Using the GRADE table, find students who have grades above the average grade for their specific section. Show student ID, section ID, their grade, and the section average.

**3. Scalar Subquery in SELECT**
Display each course with its description and the count of how many sections are offered for that course. Use a scalar subquery in the SELECT clause with COURSE and SECTION tables.

**4. Exists Subquery**
Find all courses that have never had any sections created. Use EXISTS with COURSE and SECTION tables. Display course number and description.

### Answers

**1.**
```sql
SELECT s.student_id,
       s.first_name || ' ' || s.last_name AS "Student Name",
       s.phone
FROM student s
WHERE s.student_id IN (
    SELECT e.student_id
    FROM enrollment e
    JOIN section sec ON e.section_id = sec.section_id
    WHERE sec.location = 'L211'
);
```

**2.**
```sql
SELECT g1.student_id,
       g1.section_id,
       g1.numeric_grade AS "Student Grade",
       ROUND((SELECT AVG(g2.numeric_grade)
              FROM grade g2
              WHERE g2.section_id = g1.section_id), 2) AS "Section Average"
FROM grade g1
WHERE g1.numeric_grade > (
    SELECT AVG(g2.numeric_grade)
    FROM grade g2
    WHERE g2.section_id = g1.section_id
)
ORDER BY g1.section_id, g1.numeric_grade DESC;
```

**3.**
```sql
SELECT c.course_no,
       c.description,
       (SELECT COUNT(*)
        FROM section s
        WHERE s.course_no = c.course_no) AS "Section Count"
FROM course c
ORDER BY c.course_no;
```

**4.**
```sql
SELECT c.course_no,
       c.description
FROM course c
WHERE NOT EXISTS (
    SELECT 1
    FROM section s
    WHERE s.course_no = c.course_no
)
ORDER BY c.course_no;
```

## Date and Time Logic

Working with date and time functions including SYSDATE, date formatting, extraction, and timezone handling.

**Tables Used:** STUDENT, SECTION, ENROLLMENT

### Exercises

**1. Date Arithmetic**
Using the STUDENT table, calculate each student's registration anniversary date for this year and determine if it has already passed. Show student name, original registration date, this year's anniversary, and a status indicator.

**2. Date Formatting and Extraction**
Using the SECTION table, display section information with the start date formatted in multiple ways: full date with day name, just the month and year, and just the time portion. Extract the day of the week number.

**3. Date Comparisons**
Find all enrollments from the ENROLLMENT table that occurred within the last 30 days of each section's start date (using SECTION table). This identifies last-minute enrollments.

**4. Business Date Logic**
Create a query using ENROLLMENT table that shows the enrollment date and calculates the next business day (assuming Saturday and Sunday are non-business days). Display the difference in days between enrollment and start of next business week.

### Answers

**1.**
```sql
SELECT first_name || ' ' || last_name AS "Student Name",
       registration_date AS "Original Registration",
       ADD_MONTHS(registration_date, 
                  (EXTRACT(YEAR FROM SYSDATE) - EXTRACT(YEAR FROM registration_date)) * 12) AS "Anniversary This Year",
       CASE 
         WHEN ADD_MONTHS(registration_date, 
                        (EXTRACT(YEAR FROM SYSDATE) - EXTRACT(YEAR FROM registration_date)) * 12) <= SYSDATE 
         THEN 'Anniversary Passed'
         ELSE 'Anniversary Coming'
       END AS "Status"
FROM student
ORDER BY registration_date;
```

**2.**
```sql
SELECT section_id,
       location,
       TO_CHAR(start_date_time, 'Day, Month DD, YYYY') AS "Full Date",
       TO_CHAR(start_date_time, 'Mon YYYY') AS "Month Year",
       TO_CHAR(start_date_time, 'HH24:MI') AS "Time Only",
       TO_CHAR(start_date_time, 'D') AS "Day Number"
FROM section
ORDER BY start_date_time;
```

**3.**
```sql
SELECT e.student_id,
       e.section_id,
       e.enroll_date,
       s.start_date_time,
       TRUNC(s.start_date_time) - TRUNC(e.enroll_date) AS "Days Before Start"
FROM enrollment e
JOIN section s ON e.section_id = s.section_id
WHERE e.enroll_date >= s.start_date_time - 30
  AND e.enroll_date <= s.start_date_time
ORDER BY s.start_date_time, e.enroll_date;
```

**4.**
```sql
SELECT student_id,
       section_id,
       enroll_date,
       TO_CHAR(enroll_date, 'Day') AS "Enrollment Day",
       CASE 
         WHEN TO_CHAR(enroll_date, 'D') IN ('1', '7') THEN -- Sunday or Saturday
           NEXT_DAY(enroll_date, 'MONDAY')
         ELSE 
           enroll_date + 1
       END AS "Next Business Day",
       CASE 
         WHEN TO_CHAR(enroll_date, 'D') IN ('1', '7') THEN
           NEXT_DAY(enroll_date, 'MONDAY') - enroll_date
         ELSE 1
       END AS "Days to Next Business Day"
FROM enrollment
ORDER BY enroll_date;
```

## Query Readability and Optimization

Best practices for writing readable and efficient SQL queries, including formatting, documentation, and performance optimization.

**Tables Used:** Various STUDENT schema tables

### Exercises

**1. Query Reformatting**
Take this poorly formatted query and rewrite it with proper indentation, aliases, and comments:
```sql
select s.student_id,s.first_name,s.last_name,c.description,sec.location from student s,enrollment e,section sec,course c where s.student_id=e.student_id and e.section_id=sec.section_id and sec.course_no=c.course_no and c.cost>1000
```

**2. Performance Optimization**
Identify and fix performance issues in this query that finds students with high grades:
```sql
SELECT *
FROM student
WHERE student_id IN (
    SELECT student_id
    FROM grade
    WHERE TO_NUMBER(SUBSTR(TO_CHAR(numeric_grade), 1, 2)) > 90
);
```

**3. Subquery to Join Conversion**
Convert this subquery-based solution to use joins for better performance:
```sql
SELECT course_no, description
FROM course
WHERE course_no IN (
    SELECT DISTINCT course_no
    FROM section
    WHERE section_id IN (
        SELECT section_id
        FROM enrollment
        WHERE student_id = 123
    )
);
```

**4. Documentation Practice**
Add comprehensive comments to this complex query explaining the business logic:
```sql
SELECT s.first_name, s.last_name, AVG(g.numeric_grade)
FROM student s
JOIN enrollment e ON s.student_id = e.student_id
JOIN grade g ON e.student_id = g.student_id AND e.section_id = g.section_id
GROUP BY s.student_id, s.first_name, s.last_name
HAVING AVG(g.numeric_grade) > 85
ORDER BY AVG(g.numeric_grade) DESC;
```

### Answers

**1.**
```sql
-- Display student enrollment information for expensive courses
-- Shows student details, course information, and section location
-- for courses costing more than $1000
SELECT s.student_id,
       s.first_name,
       s.last_name,
       c.description AS course_title,
       sec.location
FROM student s
JOIN enrollment e ON s.student_id = e.student_id
JOIN section sec ON e.section_id = sec.section_id
JOIN course c ON sec.course_no = c.course_no
WHERE c.cost > 1000
ORDER BY s.last_name, s.first_name;
```

**2.**
```sql
-- Optimized version: avoid functions on columns, use direct comparison
SELECT s.student_id,
       s.first_name,
       s.last_name
FROM student s
WHERE EXISTS (
    SELECT 1
    FROM grade g
    WHERE g.student_id = s.student_id
      AND g.numeric_grade > 90
);
```

**3.**
```sql
-- Join-based solution for better performance
-- Find courses that student 123 is enrolled in
SELECT DISTINCT c.course_no,
                c.description
FROM course c
JOIN section sec ON c.course_no = sec.course_no
JOIN enrollment e ON sec.section_id = e.section_id
WHERE e.student_id = 123;
```

**4.**
```sql
/*
 * High-Performing Students Report
 * 
 * Business Logic: Identify students with strong academic performance
 * by calculating their overall grade average across all enrolled courses.
 * Only include students with an average grade above 85%.
 * 
 * Tables Used:
 * - STUDENT: Student demographic information
 * - ENROLLMENT: Links students to their enrolled sections
 * - GRADE: Individual assignment/exam grades for enrolled students
 */

-- Main query: Calculate average grades for high-performing students
SELECT s.first_name,
       s.last_name,
       ROUND(AVG(g.numeric_grade), 2) AS overall_average
FROM student s
    -- Join to get student enrollments
    JOIN enrollment e ON s.student_id = e.student_id
    -- Join to get grades for each enrollment
    -- Note: Both student_id and section_id needed for proper grade linkage
    JOIN grade g ON e.student_id = g.student_id 
                AND e.section_id = g.section_id
-- Group by student to calculate individual averages
GROUP BY s.student_id, s.first_name, s.last_name
-- Filter for high performers (>85% average)
HAVING AVG(g.numeric_grade) > 85
-- Order by performance, best first
ORDER BY AVG(g.numeric_grade) DESC;
```

## Refactoring Queries with CTEs

How to simplify complex queries using Common Table Expressions (WITH clause) for better readability and maintainability.

**Tables Used:** STUDENT, ENROLLMENT, SECTION, COURSE, GRADE

### Exercises

**1. Basic CTE Usage**
Refactor a complex nested query using a CTE. Find students who are enrolled in more than 2 sections and show their enrollment details with course information.

**2. Multiple CTEs**
Create a query using multiple CTEs to analyze student performance. First CTE calculates average grades per student, second CTE finds high performers (>90 average), then join with student info to show names and averages.

**3. CTE for Complex Aggregation**
Use CTEs to create a student report card showing: student info, courses enrolled, average grade per course, and overall GPA. Break down the complexity using multiple CTEs.

### Answers

**1.**
```sql
-- Use CTE to find students with multiple enrollments
WITH student_enrollment_count AS (
    SELECT s.student_id,
           s.first_name,
           s.last_name,
           COUNT(e.section_id) AS enrollment_count
    FROM student s
    JOIN enrollment e ON s.student_id = e.student_id
    GROUP BY s.student_id, s.first_name, s.last_name
    HAVING COUNT(e.section_id) > 2
)
SELECT sec.student_id,
       sec.first_name,
       sec.last_name,
       sec.enrollment_count,
       c.description AS course_title,
       sect.location
FROM student_enrollment_count sec
JOIN enrollment e ON sec.student_id = e.student_id
JOIN section sect ON e.section_id = sect.section_id
JOIN course c ON sect.course_no = c.course_no
ORDER BY sec.last_name, c.description;
```

**2.**
```sql
-- Multiple CTEs for student performance analysis
WITH student_averages AS (
    -- Calculate average grade per student
    SELECT g.student_id,
           ROUND(AVG(g.numeric_grade), 2) AS avg_grade
    FROM grade g
    GROUP BY g.student_id
),
high_performers AS (
    -- Identify students with >90 average
    SELECT student_id, avg_grade
    FROM student_averages
    WHERE avg_grade > 90
)
-- Final result with student names
SELECT s.first_name,
       s.last_name,
       hp.avg_grade AS "Overall Average"
FROM high_performers hp
JOIN student s ON hp.student_id = s.student_id
ORDER BY hp.avg_grade DESC;
```

**3.**
```sql
-- Complex student report card using multiple CTEs
WITH student_courses AS (
    -- Get student course enrollments
    SELECT s.student_id,
           s.first_name,
           s.last_name,
           c.course_no,
           c.description AS course_title,
           e.section_id
    FROM student s
    JOIN enrollment e ON s.student_id = e.student_id
    JOIN section sec ON e.section_id = sec.section_id
    JOIN course c ON sec.course_no = c.course_no
),
course_grades AS (
    -- Calculate average grade per student per course
    SELECT sc.student_id,
           sc.first_name,
           sc.last_name,
           sc.course_no,
           sc.course_title,
           ROUND(AVG(g.numeric_grade), 2) AS course_average
    FROM student_courses sc
    LEFT JOIN grade g ON sc.student_id = g.student_id 
                     AND sc.section_id = g.section_id
    GROUP BY sc.student_id, sc.first_name, sc.last_name, 
             sc.course_no, sc.course_title
),
student_gpa AS (
    -- Calculate overall GPA per student
    SELECT student_id,
           ROUND(AVG(course_average), 2) AS overall_gpa
    FROM course_grades
    WHERE course_average IS NOT NULL
    GROUP BY student_id
)
-- Final report card
SELECT cg.first_name || ' ' || cg.last_name AS "Student Name",
       cg.course_no AS "Course",
       cg.course_title AS "Course Title",
       NVL(TO_CHAR(cg.course_average), 'No Grades') AS "Course Average",
       sg.overall_gpa AS "Overall GPA"
FROM course_grades cg
LEFT JOIN student_gpa sg ON cg.student_id = sg.student_id
ORDER BY cg.last_name, cg.first_name, cg.course_no;
```