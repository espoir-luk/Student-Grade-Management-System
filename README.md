**Name:** Rukundo Espoir

**ID:** 27678

**GROUP:** D

# 🎓 Student Grade Management System
A comprehensive PL/SQL application demonstrating advanced database programming concepts including Collections, Records, and GOTO statements for academic grade management.

---

## 📋 Table of Contents

Project Overview

PL/SQL Features Demonstrated

Database Schema

Code Implementation

Output Results

## 🚀 Project Overview

The Student Grade Management System is a PL/SQL package that handles student academic records, grade calculations, and performance reporting. It serves as a practical demonstration of advanced Oracle PL/SQL programming concepts.

**Key Functionalities:**

Student GPA calculation and tracking

Academic performance reporting

Grade categorization and letter grade conversion

Class summary statistics

Comprehensive error handling

## 🛠 PL/SQL Features Demonstrated

**1. Collections**

**Associative Arrays:** grade_array for efficient GPA storage

**Nested Tables:** course_table and stats_table for dynamic data

**BULK COLLECT:** High-performance data retrieval

**2. Records**
   
Table-based Records: student_rec mirroring database structure

Programmer-defined Records: course_rec for custom data structures

**3. GOTO Statements**

**Controlled Flow:** Grade categorization (Excellent/Good/Average)

**Error Handling:** Jump to specific validation sections

**Function Returns:** Efficient exit points

## 🗃 Database Schema

**Tables Structure:**

```sql
-- 01_table_setup.sql
CREATE TABLE students (
    student_id NUMBER PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    enrollment_date DATE DEFAULT SYSDATE
);

CREATE TABLE courses (
    course_id NUMBER PRIMARY KEY,
    course_name VARCHAR2(100) NOT NULL,
    credits NUMBER DEFAULT 3
);

CREATE TABLE grades (
    grade_id NUMBER PRIMARY KEY,
    student_id NUMBER REFERENCES students(student_id),
    course_id NUMBER REFERENCES courses(course_id),
    grade_value NUMBER(4,2)
);

-- Sample data
INSERT INTO students VALUES (1, 'John', 'Doe', DATE '2023-09-01');
INSERT INTO students VALUES (2, 'Jane', 'Smith', DATE '2023-09-01');
INSERT INTO students VALUES (3, 'Mike', 'Johnson', DATE '2023-09-01');

INSERT INTO courses VALUES (101, 'Mathematics', 3);
INSERT INTO courses VALUES (102, 'Physics', 4);
INSERT INTO courses VALUES (103, 'Chemistry', 3);

INSERT INTO grades VALUES (1, 1, 101, 85.5);
INSERT INTO grades VALUES (2, 1, 102, 92.0);
INSERT INTO grades VALUES (3, 1, 103, 78.5);
INSERT INTO grades VALUES (4, 2, 101, 91.0);
INSERT INTO grades VALUES (5, 2, 102, 88.5);
INSERT INTO grades VALUES (6, 3, 101, 72.0);
INSERT INTO grades VALUES (7, 3, 103, 81.5);

COMMIT;
````

## 🔧 Code Implementation

**Package Specification:**

```sql
CREATE OR REPLACE PACKAGE student_mgmt_pkg IS
    -- Records
    TYPE student_rec IS RECORD (
        student_id students.student_id%TYPE,
        full_name VARCHAR2(100),
        gpa NUMBER(3,2)
    );
    
    -- Collections
    TYPE student_table IS TABLE OF student_rec;
    TYPE grade_array IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
    
    -- Procedures
    PROCEDURE calc_student_gpa(p_student_id IN NUMBER);
    PROCEDURE show_student_report(p_student_id IN NUMBER);
    PROCEDURE show_class_summary;
    
    -- Functions
    FUNCTION get_letter_grade(p_grade IN NUMBER) RETURN VARCHAR2;
    
END student_mgmt_pkg;
/
```

**Package Body**

```sql
CREATE OR REPLACE PACKAGE BODY student_mgmt_pkg IS
    
    -- Private collection
    student_grades grade_array;
    
    -- Function to calculate letter grade with GOTO
    FUNCTION get_letter_grade(p_grade IN NUMBER) RETURN VARCHAR2 IS
        v_grade VARCHAR2(2);
    BEGIN
        IF p_grade >= 90 THEN
            v_grade := 'A';
            GOTO return_result;
        ELSIF p_grade >= 80 THEN
            v_grade := 'B';
            GOTO return_result;
        ELSIF p_grade >= 70 THEN
            v_grade := 'C';
            GOTO return_result;
        ELSIF p_grade >= 60 THEN
            v_grade := 'D';
            GOTO return_result;
        ELSE
            v_grade := 'F';
            GOTO return_result;
        END IF;
        
        <<return_result>>
        RETURN v_grade;
    END get_letter_grade;
    
    -- Procedure to calculate GPA
    PROCEDURE calc_student_gpa(p_student_id IN NUMBER) IS
        v_total NUMBER := 0;
        v_count NUMBER := 0;
        v_gpa NUMBER;
        v_name VARCHAR2(100);
    BEGIN
        -- Get student name
        SELECT first_name || ' ' || last_name INTO v_name
        FROM students WHERE student_id = p_student_id;
        
        -- Calculate average grade
        SELECT AVG(grade_value), COUNT(*)
        INTO v_gpa, v_count
        FROM grades 
        WHERE student_id = p_student_id;
        
        -- GOTO for no grades
        IF v_count = 0 THEN
            DBMS_OUTPUT.PUT_LINE('No grades found for ' || v_name);
            GOTO end_procedure;
        END IF;
        
        DBMS_OUTPUT.PUT_LINE('Student: ' || v_name);
        DBMS_OUTPUT.PUT_LINE('GPA: ' || ROUND(v_gpa, 2));
        
        -- Store in collection
        student_grades(p_student_id) := v_gpa;
        
        <<end_procedure>>
        NULL;
        
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Student not found!');
    END calc_student_gpa;
    
    -- Procedure to show student report
    PROCEDURE show_student_report(p_student_id IN NUMBER) IS
        -- Record for course details
        TYPE course_rec IS RECORD (
            course_name courses.course_name%TYPE,
            grade_value grades.grade_value%TYPE,
            letter_grade VARCHAR2(2)
        );
        
        -- Collection of records
        TYPE course_table IS TABLE OF course_rec;
        v_courses course_table;
        v_name VARCHAR2(100);
        
    BEGIN
        -- Get student name
        SELECT first_name || ' ' || last_name INTO v_name
        FROM students WHERE student_id = p_student_id;
        
        DBMS_OUTPUT.PUT_LINE('=== REPORT FOR ' || v_name || ' ===');
        
        -- Get courses and grades using bulk collection
        SELECT c.course_name, g.grade_value, get_letter_grade(g.grade_value)
        BULK COLLECT INTO v_courses
        FROM courses c, grades g
        WHERE c.course_id = g.course_id 
        AND g.student_id = p_student_id;
        
        -- Display results with GOTO
        FOR i IN 1..v_courses.COUNT LOOP
            DBMS_OUTPUT.PUT(v_courses(i).course_name || ': ' || 
                           v_courses(i).grade_value || ' (' || 
                           v_courses(i).letter_grade || ')');
            
            -- GOTO for grade comments
            IF v_courses(i).grade_value >= 90 THEN
                GOTO excellent;
            ELSIF v_courses(i).grade_value >= 80 THEN
                GOTO good;
            ELSE
                GOTO average;
            END IF;
            
            <<excellent>>
            DBMS_OUTPUT.PUT_LINE(' - Excellent!');
            GOTO next_course;
            
            <<good>>
            DBMS_OUTPUT.PUT_LINE(' - Good');
            GOTO next_course;
            
            <<average>>
            DBMS_OUTPUT.PUT_LINE(' - Average');
            
            <<next_course>>
            NULL;
        END LOOP;
        
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Student not found!');
    END show_student_report;
    
    -- Procedure to show class summary
    PROCEDURE show_class_summary IS
        -- Record for stats
        TYPE stats_rec IS RECORD (
            student_name VARCHAR2(100),
            avg_grade NUMBER,
            course_count NUMBER
        );
        
        -- Collection of records
        TYPE stats_table IS TABLE OF stats_rec;
        v_stats stats_table;
        
    BEGIN
        DBMS_OUTPUT.PUT_LINE('=== CLASS SUMMARY ===');
        
        -- Get student statistics
        SELECT s.first_name || ' ' || s.last_name,
               ROUND(AVG(g.grade_value), 2),
               COUNT(g.grade_id)
        BULK COLLECT INTO v_stats
        FROM students s, grades g
        WHERE s.student_id = g.student_id
        GROUP BY s.student_id, s.first_name, s.last_name
        ORDER BY AVG(g.grade_value) DESC;
        
        -- Display summary
        FOR i IN 1..v_stats.COUNT LOOP
            DBMS_OUTPUT.PUT_LINE(
                RPAD(v_stats(i).student_name, 15) ||
                RPAD(v_stats(i).avg_grade, 8) ||
                v_stats(i).course_count || ' courses'
            );
        END LOOP;
        
    END show_class_summary;
    
END student_mgmt_pkg;
/
```

**Test Script**

```sql
SET SERVEROUTPUT ON;

BEGIN
    DBMS_OUTPUT.PUT_LINE('🎓 STUDENT MANAGEMENT SYSTEM TEST');
    DBMS_OUTPUT.PUT_LINE('================================');
    
    -- Test 1: Calculate GPAs
    DBMS_OUTPUT.PUT_LINE(CHR(10) || '1. GPA CALCULATION:');
    student_mgmt_pkg.calc_student_gpa(1);
    student_mgmt_pkg.calc_student_gpa(2);
    student_mgmt_pkg.calc_student_gpa(3);
    
    -- Test 2: Student Reports
    DBMS_OUTPUT.PUT_LINE(CHR(10) || '2. STUDENT REPORTS:');
    student_mgmt_pkg.show_student_report(1);
    
    -- Test 3: Class Summary
    DBMS_OUTPUT.PUT_LINE(CHR(10) || '3. CLASS SUMMARY:');
    student_mgmt_pkg.show_class_summary;
    
    -- Test 4: Letter Grade Function
    DBMS_OUTPUT.PUT_LINE(CHR(10) || '4. LETTER GRADES:');
    DBMS_OUTPUT.PUT_LINE('85 = ' || student_mgmt_pkg.get_letter_grade(85));
    DBMS_OUTPUT.PUT_LINE('92 = ' || student_mgmt_pkg.get_letter_grade(92));
    DBMS_OUTPUT.PUT_LINE('67 = ' || student_mgmt_pkg.get_letter_grade(67));
    
    DBMS_OUTPUT.PUT_LINE(CHR(10) || '✅ ALL TESTS COMPLETED!');
    
END;
/
```

## Output:

![](./tests%20results.PNG) 
