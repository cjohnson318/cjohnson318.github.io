---
layout: post
title: "SQL Crash Course"
date: 2025-06-06 00:00:00 -0700
tags: sql
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

Next, we'll populate the database with some dummy data. Type `sqlite3
database.db` to enter the database shell, then enter the following,

{% highlight sql %}
-- Students
INSERT INTO Students (first_name, last_name, date_of_birth) VALUES ('Alice', 'Smith', '2008-05-15');
INSERT INTO Students (first_name, last_name, date_of_birth) VALUES ('Bob', 'Johnson', '2007-11-20');
INSERT INTO Students (first_name, last_name, date_of_birth) VALUES ('Charlie', 'Brown', '2009-02-01');
INSERT INTO Students (first_name, last_name, date_of_birth) VALUES ('Diana', 'Prince', '2007-07-07');
INSERT INTO Students (first_name, last_name, date_of_birth) VALUES ('Eve', 'Adams', '2008-09-09');

-- Teachers
INSERT INTO Teachers (first_name, last_name, email) VALUES ('Mr.', 'Johnson', 'johnson@school.edu');
INSERT INTO Teachers (first_name, last_name, email) VALUES ('Ms.', 'Davis', 'davis@school.edu');
INSERT INTO Teachers (first_name, last_name, email) VALUES ('Dr.', 'Miller', 'miller@school.edu');

-- Courses
INSERT INTO Courses (course_name, course_code, credits) VALUES ('Introduction to Algebra', 'MATH101', 3);
INSERT INTO Courses (course_name, course_code, credits) VALUES ('Calculus I', 'MATH301', 4);
INSERT INTO Courses (course_name, course_code, credits) VALUES ('History of Art', 'ART301', 3);
INSERT INTO Courses (course_name, course_code, credits) VALUES ('English Literature', 'ENG201', 3);
INSERT INTO Courses (course_name, course_code, credits) VALUES ('Biology I', 'BIO101', 4);

-- Sections (assuming IDs based on insertion order)
-- MATH101 (id 1) by Mr. Johnson (id 1)
INSERT INTO Sections (course_id, teacher_id, semester, capacity) VALUES (1, 1, 'Fall 2024', 25); -- section_id: 1
-- MATH301 (id 2) by Ms. Davis (id 2)
INSERT INTO Sections (course_id, teacher_id, semester, capacity) VALUES (2, 2, 'Fall 2024', 20); -- section_id: 2
-- ART301 (id 3) by Dr. Miller (id 3)
INSERT INTO Sections (course_id, teacher_id, semester, capacity) VALUES (3, 3, 'Fall 2024', 30); -- section_id: 3
-- ENG201 (id 4) by Ms. Davis (id 2)
INSERT INTO Sections (course_id, teacher_id, semester, capacity) VALUES (4, 2, 'Fall 2024', 28); -- section_id: 4
-- BIO101 (id 5) by Mr. Johnson (id 1)
INSERT INTO Sections (course_id, teacher_id, semester, capacity) VALUES (5, 1, 'Fall 2024', 22); -- section_id: 5
-- MATH101 (id 1) by Mr. Johnson (id 1) - another section
INSERT INTO Sections (course_id, teacher_id, semester, capacity) VALUES (1, 1, 'Spring 2025', 27); -- section_id: 6

-- Enrollments
-- Alice (id 1)
INSERT INTO Enrollments (student_id, section_id) VALUES (1, 1); -- Alice in MATH101 (Fall 2024)
INSERT INTO Enrollments (student_id, section_id) VALUES (1, 3); -- Alice in ART301 (Fall 2024)
-- Bob (id 2)
INSERT INTO Enrollments (student_id, section_id) VALUES (2, 1); -- Bob in MATH101 (Fall 2024)
INSERT INTO Enrollments (student_id, section_id) VALUES (2, 2); -- Bob in MATH301 (Fall 2024)
-- Charlie (id 3)
INSERT INTO Enrollments (student_id, section_id) VALUES (3, 4); -- Charlie in ENG201 (Fall 2024)
-- Diana (id 4)
INSERT INTO Enrollments (student_id, section_id) VALUES (4, 2); -- Diana in MATH301 (Fall 2024)
INSERT INTO Enrollments (student_id, section_id) VALUES (4, 5); -- Diana in BIO101 (Fall 2024)
-- Eve (id 5)
INSERT INTO Enrollments (student_id, section_id) VALUES (5, 1); -- Eve in MATH101 (Fall 2024)
INSERT INTO Enrollments (student_id, section_id) VALUES (5, 4); -- Eve in ENG201 (Fall 2024)
{% endhighlight %}

Then type `.quit` and hit return to exit the database.

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
and then query them. This example will find out which students are in which
courses, and who teaches those courses. This requires joining Students, Enrollments,
Sections, Courses, and Teachers.

{% highlight sql %}
SELECT
    S.first_name AS student_first_name,
    S.last_name AS student_last_name,
    C.course_name,
    C.course_code,
    T.first_name AS teacher_first_name,
    T.last_name AS teacher_last_name,
    Sec.semester
FROM
    Students AS S
INNER JOIN
    Enrollments AS E ON S.student_id = E.student_id
INNER JOIN
    Sections AS Sec ON E.section_id = Sec.section_id
INNER JOIN
    Courses AS C ON Sec.course_id = C.course_id
INNER JOIN
    Teachers AS T ON Sec.teacher_id = T.teacher_id
ORDER BY
    S.last_name, S.first_name, Sec.semester, C.course_name;
{% endhighlight %}

This should produce,

{% highlight text %}
student_first_name  student_last_name  course_name            course_code  teacher_first_name  teacher_last_name  semester
------------------  -----------------  ---------------------  -----------  ------------------  -----------------  -----------
Alice               Smith              History of Art         ART301       Dr.                 Miller             Fall 2024
Alice               Smith              Introduction to Algebr MATH101      Mr.                 Johnson            Fall 2024
Bob                 Johnson            Calculus I             MATH301      Ms.                 Davis              Fall 2024
Bob                 Johnson            Introduction to Algebr MATH101      Mr.                 Johnson            Fall 2024
Charlie             Brown              English Literature     ENG201       Ms.                 Davis              Fall 2024
Diana               Prince             Biology I              BIO101       Mr.                 Johnson            Fall 2024
Diana               Prince             Calculus I             MATH301      Ms.                 Davis              Fall 2024
Eve                 Adams              English Literature     ENG201       Ms.                 Davis              Fall 2024
Eve                 Adams              Introduction to Algebr MATH101      Mr.                 Johnson            Fall 2024
{% endhighlight %}


## VIEWS

After you get the hange of `JOIN`-ing tables, you'll probably want to save all of
that work into a `VIEW` so that you can just say, `SELECT someView;` instead of,
`SELECT ... FROM ... JOIN ... ON ...` every time.

This example creates a view that combines information from our Students and
Enrollments tables to show which students are enrolled in which sections, along
with their enrollment date. This simplifies querying student enrollments 
without needing to perform a JOIN every time. 

{% highlight sql %}
CREATE VIEW StudentEnrollmentsView AS
SELECT
    S.student_id,
    S.first_name,
    S.last_name,
    E.section_id,
    E.enrollment_date
FROM
    Students AS S
INNER JOIN
    Enrollments AS E ON S.student_id = E.student_id
ORDER BY
    S.last_name, S.first_name, E.enrollment_date;
{% endhighlight %}

Now we can query Alice's enrollments as,

{% highlight sql %}
SELECT * FROM StudentEnrollmentsView WHERE first_name = 'Alice';
{% endhighlight %}


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

You can also do bulk deletes on larger sets of rows using the `WHERE` clause,

{% highlight sql %}
DELETE FROM Students
WHERE enrollment_date > '2024-01-01';
{% endhighlight %}


## Aggregate Functions and GROUP BY

Aggregate functions and `GROUP BY` allow you to summarize repeated data within
tables, and across tables.

To get the total count of students you could do,

{% highlight sql %}
SELECT COUNT(*) AS total_students
FROM Students;
{% endhighlight %}

To get the total number of courses taught by each teacher you could do,

{% highlight sql %}
SELECT
    t.first_name,
    t.last_name,
    COUNT(s.section_id) AS number_of_sections_taught
FROM
    Teachers AS t
JOIN
    Sections AS s ON t.teacher_id = s.teacher_id
GROUP BY
    t.teacher_id, t.first_name, t.last_name; -- Group by teacher's ID and name
{% endhighlight %}

This would produce something like,

{% highlight text %}
first_name  last_name   number_of_sections_taught
----------  ----------  -------------------------
Dr.         Miller      1
Mr.         Johnson     2
Ms.         Davis       2
{% endhighlight %}

If you want to count the number of students per section,

{% highlight sql %}
SELECT
    s.section_id,
    c.course_name,
    t.last_name AS teacher_last_name,
    COUNT(e.student_id) AS num_students_enrolled
FROM
    Sections AS s
JOIN
    Courses AS c ON s.course_id = c.course_id
JOIN
    Teachers AS t ON s.teacher_id = t.teacher_id
LEFT JOIN
    Enrollments AS e ON s.section_id = e.section_id
GROUP BY
    s.section_id, c.course_name, t.last_name
ORDER BY
    num_students_enrolled DESC;
{% endhighlight %}

This would produce something like,

{% highlight text %}
section_id  course_name            teacher_last_name  num_students_enrolled
----------  ---------------------  -----------------  ---------------------
1           Introduction to Algebr Johnson              3
2           Calculus I             Davis                2
4           English Literature     Davis                2
3           History of Art         Miller               1
5           Biology I              Johnson              1
6           Introduction to Algebr Johnson              0
{% endhighlight %}

## HAVING

The `HAVING` clause acts like a `WHERE` clause for `GROUP BY` statements.

In this example, we're gathering all of the teachers, grouping them by their
name/id, and then filtering that result by the teachers that teach more than
one class.

{% highlight sql %}
SELECT
    t.first_name,
    t.last_name,
    COUNT(s.section_id) AS number_of_sections_taught
FROM
    Teachers AS t
INNER JOIN
    Sections AS s ON t.teacher_id = s.teacher_id
GROUP BY
    t.teacher_id, t.first_name, t.last_name
HAVING
    COUNT(s.section_id) > 1;
{% endhighlight %}

This should produce the following,

{% highlight text %}
first_name  last_name   number_of_sections_taught
----------  ----------  -------------------------
Mr.         Johnson     2
Ms.         Davis       2
{% endhighlight %}


## Subqueries

In this example we want a list of all students who are enrolled in any course
section taught by 'Ms. Davis'. The outer query selects first and last names from
students in the Students table, according to the inner subquery in the `WHERE`
clause. The inner subquery looks at students enrolled in Ms. Davis's class.

{% highlight sql %}
SELECT
    first_name,
    last_name
FROM
    Students
WHERE
    student_id IN (
        SELECT
            e.student_id
        FROM
            Enrollments AS e
        INNER JOIN
            Sections AS sec ON e.section_id = sec.section_id
        INNER JOIN
            Teachers AS t ON Sec.teacher_id = t.teacher_id
        WHERE
            t.first_name = 'Ms.' AND t.last_name = 'Davis'
    );
{% endhighlight %}

This should return,

{% highlight text %}
first_name  last_name
----------  ----------
Bob         Johnson
Diana       Prince
Charlie     Brown
Eve         Adams
{% endhighlight %}


## CREATE, ALTER, DROP

We saw above how to create the Students table,

{% highlight sql %}
CREATE TABLE IF NOT EXISTS Students (
    student_id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name TEXT NOT NULL,
    last_name TEXT NOT NULL,
    date_of_birth TEXT, -- Stored as YYYY-MM-DD
    enrollment_date TEXT DEFAULT CURRENT_DATE -- When the student enrolled
);
{% endhighlight %}

We can add columns using the `ALTER` command. For example, we may want to add
a GPA column to the Students table.

{% highlight sql %}
ALTER TABLE Students ADD COLUMN gpa REAL;
{% endhighlight %}

If we decide we don't like that column name, then we can rename it,

{% highlight sql %}
ALTER TABLE Students RENAME COLUMN gpa TO grade_point_average;
{% endhighlight %}

Then we can drop that column later,

{% highlight sql %}
ALTER TABLE Students DROP COLUMN grade_point_average;
{% endhighlight %}

We might rename the entire table,

{% highlight sql %}
ALTER TABLE Students RENAME TO HaplessDebtors;
{% endhighlight %}

We can destory the entire table by saying,

{% highlight sql %}
DROP TABLE HaplessDebtors;
{% endhighlight %}

## TRIGGERS

Triggers are used to apply changes automatically based on events in the database.
In this example, we want to update a `last_modified_at` column on the Students
table.

First we need to add a `last_modified_at` column to the Student's table,


{% highlight sql %}
ALTER TABLE Students
ADD COLUMN last_modified_at TEXT DEFAULT CURRENT_TIMESTAMP;
{% endhighlight %}

Now we can add the trigger. Any time we update the Students table, this trigger
should fire, and then update that row's `last_modified_at` column.

{% highlight sql %}
CREATE TRIGGER update_student_timestamp
AFTER UPDATE ON Students
FOR EACH ROW
BEGIN
    UPDATE Students
    SET last_modified_at = CURRENT_TIMESTAMP
    WHERE student_id = OLD.student_id;
END;
{% endhighlight %}
