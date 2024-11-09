# Assingment_2
**Course:** Database Management System  (CSEC-322)<br />
**Assingment  On:** Problem Solving with SQL Queries <br/>

## Discription of Problem 3.1 
------------------------------------------
Write the following queries in SQL, using the university schema. (We suggest
you actually run these queries on a database, using the sample data that we
provide on the web site of the book, db-book.com. Instructions for setting up
a database, and loading sample data, are provided on the above web site.)<br>

<b> Query for a : Find the titles of courses in the Comp. Sci. department that have 3 credits.</br>
---------------------------------------------------------------------------------------------------------</b>
```sql
SELECT course_title
FROM Courses
WHERE department = 'CSE'
  AND credits = 3;
```

<b> Query for _b: Find the IDs of all students who were taught by an instructor named Einstein; make sure there are no duplicates in the result.</br>
----------------------------------------------------------------------------------------------------------------------------------------------------------</b>
```sql
SELECT s.student_id
FROM Students s
JOIN Enrollments e ON s.student_id = e.student_id
JOIN Sections sec ON e.section_id = sec.section_id
JOIN Instructors i ON sec.instructor_id = i.instructor_id
WHERE i.name = 'Einstein'
GROUP BY s.student_id;
```

<b> Query for_c: Find the highest salary of any instructor.</br>
------------------------------------------------------------------</b>
```sql
SELECT MAX(salary) AS highest_salary
FROM Instructors;

```


<b> Query for_d: Find all instructors earning the highest salary (there may be more than one with the same salary). </br>
-------------------------------------------------------------------------------------------------------------------------------</b>
```sql
SELECT name, salary
FROM Instructors
WHERE salary = (SELECT MAX(salary) FROM Instructors);

```

<b> Query for_e: Find the enrollment of each section that was offered in Fall 2017.</br>
------------------------------------------------------------------------------------------------------------</b>
```sql
SELECT s.section_id, c.course_title, COUNT(e.student_id) AS enrollment_count
FROM Sections s
JOIN Enrollments e ON s.section_id = e.section_id
JOIN Courses c ON s.course_id = c.course_id
WHERE s.term = 'Fall 2017'
GROUP BY s.section_id, c.course_title;

```


<b> Query for_f: Find the maximum enrollment, across all sections, in Fall 2017.</br>
-------------------------------------------------------------------------------------------------</b>
```sql
SELECT MAX(enrollment_count) AS max_enrollment
FROM (
    SELECT COUNT(e.student_id) AS enrollment_count
    FROM Sections s
    JOIN Enrollments e ON s.section_id = e.section_id
    WHERE s.term = 'Fall 2017'
    GROUP BY s.section_id
) AS SectionEnrollments;
```


<b> Query for_g: Find the sections that had the maximum enrollment in Fall 2017.</br>
----------------------------------------------------------------------------------------------</b>
```sql

SELECT s.section_id, c.course_title, COUNT(e.student_id) AS enrollment_count
FROM Sections s
JOIN Enrollments e ON s.section_id = e.section_id
JOIN Courses c ON s.course_id = c.course_id
WHERE s.term = 'Fall 2017'
GROUP BY s.section_id, c.course_title
HAVING COUNT(e.student_id) = (
    SELECT MAX(enrollment_count)
    FROM (
        SELECT COUNT(e.student_id) AS enrollment_count
        FROM Sections s
        JOIN Enrollments e ON s.section_id = e.section_id
        WHERE s.term = 'Fall 2017'
        GROUP BY s.section_id
    ) AS SectionEnrollments
);
```

## Discription of Problem 3.2 
----------------------------
Suppose you are given a relation grade points(grade, points) that provides a conversion from letter grades in the takes relation to numeric scores; for example,
an “A” grade could be specified to correspond to 4 points, an “A−” to 3.7 points,
a “B+” to 3.3 points, a “B” to 3 points, and so on. The grade points earned by a
student for a course offering (section) is defined as the number of credits for the
course multiplied by the numeric points for the grade that the student received.
Given the preceding relation, and our university schema, write each of the
following queries in SQL. You may assume for simplicity that no takes tuple has
the null value for grade.

<b> Query for _a: Find the total grade points earned by the student with ID '12345', across all courses taken by the student.</br>
----------------------------------------------------------------------------------------------------------------------------------------</b>
```sql
SELECT 
    SUM(course.credits * grade_points.points) AS total_grade_points
FROM 
    takes
JOIN 
    grade_points ON takes.grade = grade_points.grade
JOIN 
    course ON takes.course_id = course.course_id
WHERE 
    takes.student_id = '12345';

```
<b> Query for _b: Find the grade point average (GPA) for the above student, that is, the total grade points divided by the total credits for the associated courses.</br>
-----------------------------------------------------------------------------------------------------------------------------------------------------------</b>
```sql
SELECT 
    takes.student_id,
    SUM(course.credits * grade_points.points) / SUM(course.credits) AS GPA
FROM 
    takes
JOIN 
    grade_points ON takes.grade = grade_points.grade
JOIN 
    course ON takes.course_id = course.course_id
GROUP BY 
    takes.student_id;


```

<b> Query for _c: Find the ID and the grade-point average of each student.</br>
-------------------------------------------------------------------------------</b>
```sql
SELECT 
    takes.student_id,
    SUM(course.credits * grade_points.points) / SUM(course.credits) AS GPA
FROM 
    takes
JOIN 
    grade_points ON takes.grade = grade_points.grade
JOIN 
    course ON takes.course_id = course.course_id
GROUP BY 
    takes.student_id;

```
<b> Query for _d: Now reconsider your answers to the earlier parts of this exercise under the assumption that some grades might be null. Explain whether your
 solutions still work and, if not, provide versions that handle nulls properly. </br>
 -------------------------------------------------------------------------------------------------------------------------------------------------------------<b>
```sql
SELECT 
    takes.student_id,
    SUM(CASE WHEN grade_points.points IS NOT NULL THEN course.credits * grade_points.points ELSE 0 END) / 
    SUM(CASE WHEN grade_points.points IS NOT NULL THEN course.credits ELSE 0 END) AS GPA
FROM 
    takes
JOIN 
    grade_points ON takes.grade = grade_points.grade
JOIN 
    course ON takes.course_id = course.course_id
GROUP BY 
    takes.student_id;

```
<b>Query for problem 3.22: Rewrite the where clause where unique (select title from course) without using the unique construct. </br>
----------------------------------------------------------------------------------------------------------------------------------------------------</b>
```sql
SELECT * 
FROM some_table
WHERE (SELECT COUNT(*) 
       FROM course 
       GROUP BY title 
       HAVING COUNT(*) = 1) = 1;

```
<b> Query for problem 3.25 : Using the university schema, write an SQL query to find the names of those
 departments whose budget is higher than that of Philosophy. List them in alphabetic order. </br>
 ------------------------------------------------------------------------------------------------------------------------------------------------------------</b>
```sql
SELECT department_name
FROM department
WHERE budget > (SELECT budget FROM department WHERE department_name = 'Philosophy')
ORDER BY department_name;
```
<b> Query for problem 3.26 : Using the university schema, use SQL to do the following: For each student who
 has retaken a course at least twice (i.e., the student has taken the course at least
 three times), show the course ID and the student’s ID.Please display your results in order
 of course ID and do not display duplicate rows.</br>
 ---------------------------------------------------------------------------------------------------------------------------------------------------------</b>
```sql
SELECT takes.student_id, takes.course_id
FROM takes
GROUP BY takes.student_id, takes.course_id
HAVING COUNT(*) >= 3
ORDER BY takes.course_id;
```
