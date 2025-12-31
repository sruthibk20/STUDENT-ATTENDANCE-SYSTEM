# STUDENT-ATTENDANCE-SYSTEM
SQL-based Student Attendance Management System

CREATE DATABASE student_attendances;
USE student_attendances;
CREATE TABLE student (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(50),
    department VARCHAR(30),
    year INT
);
INSERT INTO student VALUES
(101,'Anu','CSE',2),
(102,'Bala','CSE',2),
(103,'Charan','CSE',2),
(104,'Divya','CSE',2),
(105,'Esha','CSE',2);
CREATE TABLE subject (
    subject_id INT PRIMARY KEY,
    subject_name VARCHAR(50),
    department VARCHAR(30),
    semester INT
);
INSERT INTO subject VALUES
(201,'DBMS','CSE',4),
(202,'OS','CSE',4),
(203,'CN','CSE',4);
CREATE TABLE attendance (
    attendance_id INT AUTO_INCREMENT PRIMARY KEY,
    date DATE,
    status ENUM('Present','Absent','Leave'),
    student_id INT,
    subject_id INT,
    FOREIGN KEY (student_id) REFERENCES student(student_id),
    FOREIGN KEY (subject_id) REFERENCES subject(subject_id)
);
INSERT INTO attendance (date,status,student_id,subject_id) VALUES

-- Student 101 (DBMS & OS)
('2025-01-04','Present',101,201),
('2025-01-05','Present',101,201),
('2025-01-06','Absent',101,201),

('2025-01-03','Present',101,202),
('2025-01-04','Absent',101,202),
('2025-01-05','Present',101,202),

-- Student 102 (DBMS & OS)
('2025-01-03','Present',102,201),
('2025-01-04','Absent',102,201),
('2025-01-05','Absent',102,201),

('2025-01-01','Present',102,202),
('2025-01-02','Present',102,202),
('2025-01-03','Absent',102,202),

-- Student 103 (CN & DBMS)
('2025-01-03','Present',103,203),
('2025-01-04','Present',103,203),
('2025-01-05','Absent',103,203),

('2025-01-01','Present',103,201),
('2025-01-02','Leave',103,201),
('2025-01-03','Present',103,201),

-- Student 104 (DBMS)
('2025-01-01','Present',104,201),
('2025-01-02','Present',104,201),
('2025-01-03','Present',104,201),
('2025-01-04','Absent',104,201),

-- Student 105 (OS)
('2025-01-01','Absent',105,202),
('2025-01-02','Present',105,202),
('2025-01-03','Present',105,202),
('2025-01-04','Leave',105,202);


CREATE TABLE report (
    report_id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT,
    subject_id INT,
    attendance_percentage DECIMAL(5,2),
    generated_day DATE,
    FOREIGN KEY (student_id) REFERENCES student(student_id),
    FOREIGN KEY (subject_id) REFERENCES subject(subject_id)
);
INSERT INTO report (student_id, subject_id, attendance_percentage, generated_day)
SELECT
    student_id,
    subject_id,
    (SUM(CASE WHEN status='Present' THEN 1 ELSE 0 END) * 100.0 / COUNT(*)),
    '2025-01-31'
FROM attendance
WHERE MONTH(date)=1 AND YEAR(date)=2025
GROUP BY student_id, subject_id;
CREATE TABLE analytics (
    analytic_id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT,
    overall_percentage DECIMAL(5,2),
    attendance_status VARCHAR(20),
    analysis_date DATE,
    FOREIGN KEY (student_id) REFERENCES student(student_id)
);
INSERT INTO analytics (student_id, overall_percentage, attendance_status, analysis_date)
SELECT
    student_id,
    AVG(attendance_percentage),
    CASE
        WHEN AVG(attendance_percentage) >= 75 THEN 'Good'
        ELSE 'Shortage'
    END,
    '2025-01-31'
FROM report
GROUP BY student_id;
CREATE TABLE leave_permission (
    permission_id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT,
    leave_type VARCHAR(30),
    is_applicable VARCHAR(5),
    FOREIGN KEY (student_id) REFERENCES student(student_id)
);
INSERT INTO leave_permission (student_id, leave_type, is_applicable)
SELECT student_id, 'Medical', 'NO'
FROM student;
UPDATE leave_permission lp
JOIN analytics a ON lp.student_id = a.student_id
SET lp.is_applicable =
    CASE
        WHEN a.overall_percentage >= 75 THEN 'YES'
        ELSE 'NO'
    END;
SELECT * FROM attendance;
SELECT * FROM report;
SELECT * FROM analytics;
SELECT * FROM leave_permission;

select a.student_id,a.overall_percentage,b.leave_type,b.is_applicable from
(select student_id ,overall_percentage from analytics) a
left join
(select student_id,leave_type,is_applicable from leave_permission)b
on a.student_id=b.student_id;
