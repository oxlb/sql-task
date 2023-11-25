
### SQL Schema Definition

```sql

CREATE TABLE "student" (
    sid INTEGER PRIMARY KEY,
    name TEXT
);

CREATE TABLE "course" (
    cid INTEGER PRIMARY KEY,
    name TEXT
);

CREATE TABLE "enrollment" (
    sid INTEGER REFERENCES "student"("sid"),
    cid INTEGER REFERENCES "course"("cid"),
    score INTEGER
);
```

### SQL Queries to Perform

#### Query 1:

-   **Objective**: Retrieve the maximum score of each course, the name of the course, and the names of students who have the maximum score in that course.
-   **Expected Output**: A table with columns `course_name`, `student_name`, and `score`.

#### Query 2:

-   **Objective**: Retrieve the maximum score of each student, the name of the student, and the names of courses in which the student has the maximum score.
-   **Expected Output**: A table with columns `student_name`, `course_name`, and `score`.

### Example Data

```sql

BEGIN;
TRUNCATE student, course, enrollment CASCADE;

INSERT INTO "student" VALUES
    (1, 'student A'),
    (2, 'student B');

INSERT INTO "course" VALUES
    (1, 'course 1'),
    (2, 'course 2');

INSERT INTO "enrollment" ("sid", "cid", "score") VALUES
    (1, 1, 10),
    (1, 1, 9),
    (1, 2, 9),
    (2, 1, 10);

COMMIT;
```

### Expected Results

#### Query 1 Expected Results
____
| course_name | student_name | score |
|-------------|--------------|-------|
| course 1    | student A    | 10    |
| course 1    | student B    | 10    |
| course 2    | student A    | 9     |

#### Query 2 Expected Results
____

| student_name | course_name | score |
|--------------|-------------|-------|
| student A    | course 1    | 10    |
| student B    | course 1    | 10    |


#### Solution

### Query 1

This query retrieves the maximum score of each course, the name of the course, and the names of students who have achieved this maximum score in that course.

```sql

SELECT c.name AS course_name, s.name AS student_name, e.score
FROM enrollment e
JOIN course c ON e.cid = c.cid
JOIN student s ON e.sid = s.sid
WHERE (c.cid, e.score) IN (
    SELECT cid, MAX(score)
    FROM enrollment
    GROUP BY cid
)
ORDER BY c.name, e.score DESC;
```
### Result
____
| course_name | student_name | score |
|-------------|--------------|-------|
| course 1    | student A    | 10    |
| course 1    | student B    | 10    |
| course 2    | student A    | 9     |

In this query, we first join the `enrollment`, `course`, and `student` tables. Then, we use a subquery to find the maximum score for each course (`cid`). The main query then matches these maximum scores with the corresponding course and student names.

### Query 2

This query retrieves the maximum score of each student, the name of the student, and the names of courses in which the student has achieved this maximum score.

```sql

SELECT s.name AS student_name, c.name AS course_name, e.score
FROM enrollment e
JOIN student s ON e.sid = s.sid
JOIN course c ON e.cid = c.cid
WHERE (e.sid, e.score) IN (
    SELECT sid, MAX(score)
    FROM enrollment
    GROUP BY sid
)
ORDER BY s.name, e.score DESC;
```


### Result
____

| student_name | course_name | score |
|--------------|-------------|-------|
| student A    | course 1    | 10    |
| student B    | course 1    | 10    |


Similar to the first query, we join the `enrollment`, `student`, and `course` tables. The subquery finds the maximum score for each student (`sid`). The main query matches these maximum scores with the corresponding student and course names.

____
### Another Case

Certainly! To create a more comprehensive and edge-case inclusive dataset for the `student`, `course`, and `enrollment` tables, let's consider various scenarios, including null values in non-primary key columns. Note that primary key columns (`sid` in `student`, `cid` in `course`) cannot be null as per the database schema.

### Extended Insert Data for `student` Table

```sql

INSERT INTO "student" (sid, name) VALUES
(1, 'student A'),
(2, 'student B'),
(3, NULL),               -- Edge case: student with no name
(4, 'student D');         -- Additional student

```
### Extended Insert Data for `course` Table

```sql

INSERT INTO "course" (cid, name) VALUES
(1, 'course 1'),
(2, 'course 2'),
(3, NULL),                -- Edge case: course with no name
(4, 'course 4');          -- Additional course` 
```

### Extended Insert Data for `enrollment` Table

Here, we consider cases like students enrolled in multiple courses, courses with multiple students, and different score scenarios including NULL scores.

```sql

INSERT INTO "enrollment" (sid, cid, score) VALUES
(1, 1, 10),
(1, 2, 9),
(1, 3, NULL),            -- Edge case: student A enrolled in a course with no score
(2, 1, 10),
(2, 3, 8),
(3, 1, 7),               -- Edge case: student with no name enrolled
(4, 4, 10),              -- Additional case: another student in a different course
(1, 4, 6),               -- Student A in another course
(2, 2, NULL);            -- Edge case: student B in a course with no score
```

In these insert statements, we've included edge cases like students and courses with `NULL` names, and enrollments with `NULL` scores. This will allow testing the SQL queries against a wider variety of data scenarios.

____

Based on the `new data`  objective is to select the highest scoring student(s) in each course along with the course name and their scores. However, the existing query might not work as expected because it doesn't account for cases where multiple students have the same highest score in a course. 

### Improved Query 1:

1.  **Find the Maximum Score for Each Course**: First, we need a subquery or a common table expression (CTE) to determine the maximum score in each course.
2.  **Join with Enrollment, Course, and Student Tables**: Then, join this result with the `enrollment`, `course`, and `student` tables to get the details of the students who achieved these maximum scores.

```sql

WITH MaxScores AS (
    SELECT cid, MAX(score) AS max_score
    FROM enrollment
    GROUP BY cid
)
SELECT c.name AS course_name, s.name AS student_name, e.score
FROM enrollment e
JOIN MaxScores ms ON e.cid = ms.cid AND e.score = ms.max_score
JOIN course c ON e.cid = c.cid
JOIN student s ON e.sid = s.sid
ORDER BY c.name, e.score DESC;
```
### Explanation:

-   **CTE MaxScores**: This common table expression finds the maximum score (`max_score`) for each course (`cid`).
-   **Main Query**:
    -   The main query joins the `enrollment` table with `MaxScores` on both the course ID and the score. This ensures we only get records where the score matches the maximum score for that course.
    -   It then joins with the `course` and `student` tables to fetch the course names and student names.
    -   The results are ordered by course name and score in descending order.

----


improve this query based on the new data and ensure it accurately fetches the highest score of each student along with the course names where they achieved these scores, a few adjustments are needed. The current query does not account for cases where a student may have the same highest score in multiple courses. 

### Improved Query 2:

1.  **Find the Maximum Score for Each Student**: First, use a subquery or a common table expression (CTE) to determine the maximum score for each student.
2.  **Join with Enrollment, Student, and Course Tables**: Then, join this result with the `enrollment`, `student`, and `course` tables to get the course names and scores where students achieved their maximum scores.

```sql

WITH MaxScores AS (
    SELECT sid, MAX(score) AS max_score
    FROM enrollment
    GROUP BY sid
)
SELECT s.name AS student_name, c.name AS course_name, e.score
FROM enrollment e
JOIN MaxScores ms ON e.sid = ms.sid AND e.score = ms.max_score
JOIN student s ON e.sid = s.sid
JOIN course c ON e.cid = c.cid
ORDER BY s.name, e.score DESC;
```

### Explanation:

-   **CTE MaxScores**: This common table expression finds the maximum score (`max_score`) for each student (`sid`).
-   **Main Query**:
    -   The main query joins the `enrollment` table with `MaxScores` on both the student ID and the score. This ensures we only get records where the score matches the maximum score for that student.
    -   It then joins with the `student` and `course` tables to fetch the student names and course names.
    -   The results are ordered by student name and score in descending order.
