# Course Introduction

## Table of Contents

* [Overview](#overview)
* [What is Oracle SQL and Why Use It](#what-is-oracle-sql-and-why-use-it)
* [Module Objectives](#module-objectives)
* [Module Structure](#module-structure)
* [Prerequisites](#prerequisites)
* [Course Textbook and Materials](#course-textbook-and-materials)
* [What You Will Learn](#what-you-will-learn)
* [How to Succeed in This Course](#how-to-succeed-in-this-course)

## Overview

This course is designed for analysts and professionals who are already familiar with basic SQL syntax and who use Oracle SQL Developer for querying databases in support of data analysis and reporting. Students will build skills in writing efficient, professional-grade SQL with a focus on multi-table queries, set operations, analytical functions, and common reporting tasks.

## What is Oracle SQL and Why Use It

Structured Query Language (SQL) is the standard language used for managing and querying relational databases. Oracle SQL is Oracle Corporation’s implementation of this standard, enhanced with proprietary features and optimizations specific to the Oracle Database system. While the core syntax of SQL is consistent across database platforms (e.g., MySQL, PostgreSQL, Microsoft SQL Server), each vendor introduces extensions or nuances—Oracle included.

Oracle SQL supports a rich set of features such as analytic functions, advanced date handling, and performance tuning tools that make it a powerful choice for querying large, complex datasets.

At Georgia State University (GSU), Oracle SQL is the language used to interact with our two primary "ground-truth" data systems:

- **WPRD (Data Warehouse)**: A centralized repository that integrates data from various university systems for reporting and analysis.
- **BREPTS (Banner Database)**: The transactional system of record used by the university for managing student, faculty, finance, and administrative data.

Whether pulling data for institutional research, operational reporting, or ad hoc analysis, Oracle SQL is the essential tool for accessing and working with these authoritative sources.

## Module Objectives

By the end of this module, students will be able to:

* Understand the structure and expectations of the course
* Identify the major types of SQL operations that will be covered
* Confirm they meet the technical setup requirements and prerequisites
* Locate the primary textbook and reference materials for the course

## Module Structure

This course follows a modular structure. Each module includes:

* **Lesson Overview**: Short introduction to the topic
* **Guided Examples**: Hands-on walkthroughs using the STUDENT schema
* **Practice Exercises**: Independent SQL challenges
* **Review Quiz**: Multiple choice or short answer checks for understanding

Each lesson builds upon previous concepts. The course progresses from intermediate to advanced topics, ending with data transformation and performance optimization patterns.

## Prerequisites

To participate in this course, students should:

* Have completed the [LinkedIn Learning Quick Start Guide to SQL](https://www.linkedin.com/learning/quick-start-guide-to-sql)
* Be familiar with running basic `SELECT`, `WHERE`, `ORDER BY`, and `JOIN` queries

> SQL Developer and database setup instructions are covered in the next module. We will not troubleshoot installation issues here.

## Course Textbook and Materials

**Primary Textbook**:
[Oracle SQL by Example, 4th Edition](https://learning.oreilly.com/library/view/oracle-sql-by/9780137047345/) by Alice Rischert

Students can access the textbook for free through O’Reilly using GSU credentials. It will serve as the primary reference throughout the course.

* Most labs and practice exercises are drawn from the book
* Additional scripts and documentation are included in the course repository

## What You Will Learn

Topics include:

* Review of single-table operations
* Aggregations and `GROUP BY` queries
* Multi-table joins and self-joins
* Subqueries, `IN`, `EXISTS`, and correlated subqueries
* Built-in functions: character, number, date/time, and null handling
* Set operators: `UNION`, `INTERSECT`, `MINUS`
* Analytical/window functions and `WITH` clause
* Best practices for query readability and performance

## How to Get the Most Out of This Course

* Practice! SQL is a skill best learned hands-on
* Use the textbook as both a reference and a source for deeper exploration
* Ask questions and share queries with classmates for feedback
* Review answers and explanations for exercises

