---
layout: post
title: "SQL Crash Course"
date: 2025-06-06 00:00:00 -0700
tags: python
---

Having a basic understanding of SQL is incredibly useful. You can query your
relational database, you can query tabular numerical data using DuckDB, you can
query, multiple distributed databases and file server using Apache Hive. Since
SQL has been around since the 1970s, a lot of technologies offer an SQL or
SQL-like interface.

This guide will not get into detail and differences between different SQL
dialects, like MySQL versus MS SQL Server, but it will give you a general
understanding of what tools exist in most SQL dialects, and how to use them, so
that you can ask an AI more specific questions for whatever SQL product that
you're using.

## Basic Idea

In this example we'll consider a simple database describing a school with
students, teachers, courses, sections, and enrollments. A course is a general
offering, like "MATH-101", while a section is particular instance of a course,
like "MATH-101, Fall 2012".

SQL is used to query database tables that (usually) have relationships to other
tables. Every row in a table has an ID, or Primary Key, and these relationships 
are defined using those IDs. The Students table has a unique ID (for that table)
for each student. The Enrollments table has a Sections ID column, and a Students
ID column so that we can see that a specific Student is enrolled in a specific
Section. If we want to find out the name of the student or the section, then
we would follow that ID, known as a Foreign Key, to the table it belongs to.
(You can also `JOIN` tables together on their primary/foreign keys, but more on
that later.)

## Schema

Schema refers to the structure of a database: the names of the tables, the names
of the columns in each table, and how tables relate to each other through primary
and foreign keys. This document creates the tables we'll be talking about below.
Comments in a `.SQL` file begin with a double dash, `--`.

{% highlight sql %}
-- File: schema.sql

-- Table: Students
-- Stores information about individual students
CREATE TABLE IF NOT EXISTS Students (
    student_id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name TEXT NOT NULL,
    last_name TEXT NOT NULL,
    date_of_birth TEXT, -- Stored as YYYY-MM-DD
    enrollment_date TEXT DEFAULT CURRENT_DATE -- When the student enrolled
);

-- Table: Teachers
-- Stores information about teachers
CREATE TABLE IF NOT EXISTS Teachers (
    teacher_id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name TEXT NOT NULL,
    last_name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    hire_date TEXT DEFAULT CURRENT_DATE
);

-- Table: Courses
-- Stores information about the courses offered at the school
CREATE TABLE IF NOT EXISTS Courses (
    course_id INTEGER PRIMARY KEY AUTOINCREMENT,
    course_name TEXT NOT NULL UNIQUE,
    course_code TEXT UNIQUE NOT NULL, -- e.g., "MATH101", "ENG202"
    credits INTEGER NOT NULL
);

-- Table: Sections
-- Represents specific instances of a course offered in a particular term
-- A course can have multiple sections, possibly taught by different teachers
CREATE TABLE IF NOT EXISTS Sections (
    section_id INTEGER PRIMARY KEY AUTOINCREMENT,
    course_id INTEGER NOT NULL,
    teacher_id INTEGER NOT NULL,
    semester TEXT NOT NULL, -- e.g., "Fall 2024", "Spring 2025"
    capacity INTEGER NOT NULL,
    FOREIGN KEY (course_id) REFERENCES Courses(course_id),
    FOREIGN KEY (teacher_id) REFERENCES Teachers(teacher_id)
);

-- Table: Enrollments
-- Links students to the sections they are enrolled in
CREATE TABLE IF NOT EXISTS Enrollments (
    enrollment_id INTEGER PRIMARY KEY AUTOINCREMENT,
    student_id INTEGER NOT NULL,
    section_id INTEGER NOT NULL,
    enrollment_date TEXT DEFAULT CURRENT_DATE,
    -- Ensure a student can only enroll in a specific section once
    UNIQUE (student_id, section_id),
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (section_id) REFERENCES Sections(section_id)
);
{% endhighlight %}

For sqlite3, we can load this document into an empty database and create these
tables by doing,

{% highlight console %}
sqlite database.db < schema.sql
{% endhighlight %}

This will create the file `database.db` if it does not exist, otherwise it will
add the above tables to `database.db` if they do not exist in that database.

However, 9 times out of 10 you'll be *querying* databases, not building them,
so let's talk about that.

## SELECT

The first, simplest statement you should learn is the simple `SELECT` statement.
This will select all the rows from the Students table.

{% highlight sql %}
SELECT * FROM Students;
{% endhighlight %}

If you want to query by name, and limit the results to 10, then do this,

{% highlight sql %}
SELECT * FROM Students WHERE Students.first_name = 'John' LIMIT 10;
{% endhighlight %}

The `WHERE` clause lets you filter the result, and the `LIMIT` clause lets you
limit the results to a maximum of, in this case, 10.

You can also order abbreviate the table naem, and order the results by date,
or some other column.

{% highlight sql %}
SELECT * FROM Students s ORDER BY s.date_of_birth ASC;
{% endhighlight %}

Instead of typing out the table name `Students` again, we used the abbreviation
`s`. You can pick any abbreviation you want. The `ASC` orders the dates in
ascending order, so oldest first. You can also use `DESC`, which orders by
youngest first.

That's kind of the 80/20 for the `SELECT` statement.

## JOIN

After you get the hang of `SELECT`, you'll probably want to `JOIN` tables together
and then query them.

## VIEWS

After you get the hange of `JOIN`-ing tables, you'll probably want to save all of
that work into a `VIEW` so that you can just say, `SELECT someView;` instead of,
`SELECT ... FROM ... JOIN ... ON ...` every time.

## INSERT, UPDATE and DELETE

These statements are used to `INSERT` a new row into the database, or `UPDATE`
an existing row.

Insert a row into the Students table with,

{% highlight sql %}
INSERT INTO Students (first_name, last_name, date_of_birth) VALUES ('Alice', 'Able', '1984-01-01'); 
{% endhighlight %}

If you need to update Alice's birthday, then use the `UPDATE` statement.

{% highlight sql %}
UPDATE Students SET (first_name, last_name, date_of_birth) VALUES ('Alice', 'Able', '1984-01-01'); 
{% endhighlight %}

If you know the ID of a row, then you can delete it using a `WHERE` clause like this,

{% highlight sql %}
DELETE FROM Students WHERE student_id = '1';
{% endhighlight %}

## Aggregate Functions and GROUP BY

### COUNT() SUM() AVG() MIN() MAX()

### HAVING

## Subqueries

## CREATE, ALTER, DELETE Table

## TRIGGERS
