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
