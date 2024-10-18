# University Data ETL and Data Modeling using SQL

This project demonstrates how to create a **Star Schema** for University Data and implement an **ETL (Extract, Transform, Load)** process using SQL. The dataset includes students, courses, instructors, departments, and enrollment information, which are modeled into **dimension** and **fact** tables for easy querying and analysis.

## Table of Contents
- [Data Model](#data-model)
- [ETL Pipeline](#etl-pipeline)
  - [Extract](#extract)
  - [Transform](#transform)
  - [Load](#load)
- [How to Run](#how-to-run)

---

## Data Model

The data model is structured using a **Star Schema** with a central fact table (`Fact_Enrollment`) surrounded by dimension tables for students, courses, instructors, departments, and dates.

### Schema Design:

- **Fact_Enrollment**
    - Stores information about student enrollments in courses.
    - **Columns**: `Enrollment_ID`, `Student_ID`, `Course_ID`, `Instructor_ID`, `Department_ID`, `Date_ID`, `Grade`

- **Dim_Student**
    - Stores student details.
    - **Columns**: `Student_ID`, `Student_Name`, `Major`, `Enrollment_Year`

- **Dim_Course**
    - Stores course details.
    - **Columns**: `Course_ID`, `Course_Name`, `Credits`

- **Dim_Instructor**
    - Stores instructor details.
    - **Columns**: `Instructor_ID`, `Instructor_Name`, `Department_ID`

- **Dim_Department**
    - Stores department details.
    - **Columns**: `Department_ID`, `Department_Name`

- **Dim_Date**
    - Stores enrollment date details.
    - **Columns**: `Date_ID`, `Date`, `Term`, `Year`

[Schema - Snapshot](https://github.com/DhruvilPanchal205/Data-Models-and-ETL-using-SQL/blob/main/University_data%20Schema.png)

### SQL to Create Data Model:

```sql
-- Create Dimension Tables
CREATE TABLE Dim_Student (
    Student_ID INT PRIMARY KEY,
    Student_Name VARCHAR(255),
    Major VARCHAR(255),
    Enrollment_Year INT
);

CREATE TABLE Dim_Course (
    Course_ID INT PRIMARY KEY,
    Course_Name VARCHAR(255),
    Credits INT
);

CREATE TABLE Dim_Instructor (
    Instructor_ID INT PRIMARY KEY,
    Instructor_Name VARCHAR(255),
    Department_ID INT,
    FOREIGN KEY (Department_ID) REFERENCES Dim_Department(Department_ID)
);

CREATE TABLE Dim_Department (
    Department_ID INT PRIMARY KEY,
    Department_Name VARCHAR(255)
);

CREATE TABLE Dim_Date (
    Date_ID INT PRIMARY KEY,
    Date DATE,
    Term VARCHAR(50),
    Year INT
);

-- Create Fact Table
CREATE TABLE Fact_Enrollment (
    Enrollment_ID INT PRIMARY KEY,
    Student_ID INT,
    Course_ID INT,
    Instructor_ID INT,
    Department_ID INT,
    Date_ID INT,
    Grade VARCHAR(2),
    FOREIGN KEY (Student_ID) REFERENCES Dim_Student(Student_ID),
    FOREIGN KEY (Course_ID) REFERENCES Dim_Course(Course_ID),
    FOREIGN KEY (Instructor_ID) REFERENCES Dim_Instructor(Instructor_ID),
    FOREIGN KEY (Department_ID) REFERENCES Dim_Department(Department_ID),
    FOREIGN KEY (Date_ID) REFERENCES Dim_Date(Date_ID)
);
```

## ETL Pipeline

The ETL process is divided into three phases: **Extract**, **Transform**, and **Load**. In this project, data is extracted into staging tables, transformed to match the dimension and fact tables, and then loaded into the final schema.

### Extract

Extract data from external sources (e.g., CSV files, APIs, or databases) into staging tables.

```sql
-- Example: Import student data from CSV into staging table
LOAD DATA INFILE 'path/to/student.csv'
INTO TABLE stg_student
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
(Student_ID, Student_Name, Major, Enrollment_Year);
```

### Transform

Transform the raw data into a format that matches the structure of the dimension tables.

```sql
-- Transform and load data into Dim_Student table
INSERT INTO Dim_Student (Student_ID, Student_Name, Major, Enrollment_Year)
SELECT DISTINCT
    Student_ID,
    Student_Name,
    Major,
    Enrollment_Year
FROM stg_student;

-- Transform and load data into Dim_Course table
INSERT INTO Dim_Course (Course_ID, Course_Name, Credits)
SELECT DISTINCT
    Course_ID,
    Course_Name,
    Credits
FROM stg_course;

-- Transform and load data into Dim_Instructor table
INSERT INTO Dim_Instructor (Instructor_ID, Instructor_Name, Department_ID)
SELECT DISTINCT
    Instructor_ID,
    Instructor_Name,
    Department_ID
FROM stg_instructor;

-- Transform and load data into Dim_Department table
INSERT INTO Dim_Department (Department_ID, Department_Name)
SELECT DISTINCT
    Department_ID,
    Department_Name
FROM stg_department;

-- Transform and load data into Dim_Date table
INSERT INTO Dim_Date (Date_ID, Date, Term, Year)
SELECT DISTINCT
    ROW_NUMBER() OVER(ORDER BY Date) AS Date_ID,
    Date,
    CASE
        WHEN MONTH(Date) IN (1, 2, 3) THEN 'Spring'
        WHEN MONTH(Date) IN (9, 10, 11) THEN 'Fall'
        ELSE 'Other'
    END AS Term,
    YEAR(Date) AS Year
FROM stg_enrollment;
```

### Load

Load the transformed data into the Fact_Enrollment table, ensuring that it links with the dimension tables.

```sql
-- Load transformed data into Fact_Enrollment table
INSERT INTO Fact_Enrollment (Enrollment_ID, Student_ID, Course_ID, Instructor_ID, Department_ID, Date_ID, Grade)
SELECT
    e.Enrollment_ID,
    s.Student_ID,
    c.Course_ID,
    i.Instructor_ID,
    i.Department_ID,
    d.Date_ID,
    e.Grade
FROM stg_enrollment e
JOIN Dim_Student s ON e.Student_ID = s.Student_ID
JOIN Dim_Course c ON e.Course_ID = c.Course_ID
JOIN Dim_Instructor i ON e.Instructor_ID = i.Instructor_ID
JOIN Dim_Date d ON e.Date = d.Date;
```

## How to Run

### Prerequisites
- MySQL Server or any SQL-compatible RDBMS.
- Datasets in CSV format (e.g., student.csv, course.csv, instructor.csv, enrollment.csv).
- SQL scripts or tools to run the provided SQL code.

###Steps to Execute
- Clone this repository.
- Run the CREATE TABLE scripts provided in the Data Model section to create the database schema.
- Load the raw data into staging tables (stg_student, stg_course, stg_instructor, stg_enrollment) using LOAD DATA INFILE.
- Run the Transform scripts to process and load data into the dimension tables.
- Finally, load the data into the Fact_Enrollment table using the Load script.

##Contact
For any questions or suggestions, feel free to reach out!

Email: dhruvilpanchal205@gmail.com
LinkedIn: [LinkedIn Profile](https://www.linkedin.com/in/dhruvilpanchal/)
