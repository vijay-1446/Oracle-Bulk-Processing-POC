# Oracle ETL POC: CSV → Oracle → Transformation → CSV

## Overview

This project demonstrates an end-to-end ETL workflow using Oracle Database, PL/SQL, SQL*Loader, UTL_FILE, and Python.

The solution processes 1 million employee records through multiple stages:

1. Generate employee data using Python
2. Load CSV data into Oracle Stage A table using SQL*Loader
3. Transform data from Stage A to Stage B using PL/SQL
4. Export transformed data to CSV using UTL_FILE

## Technology Stack

* Oracle Database
* PL/SQL
* SQL*Loader
* UTL_FILE
* Python
* GitHub

## Project Flow

CSV File
↓
Stage A Table (EMPLOYEES)
↓
PL/SQL Transformation
↓
Stage B Table (EMPLOYEES_STG_B)
↓
UTL_FILE Export
↓
Output CSV File

## Key Features

### Step-1 Data Generation

* Python-based employee data generator
* Generates 1,000,000 records
* Realistic employee attributes
```python
import csv
import random
from datetime import datetime, timedelta

# Number of records
NUM_RECORDS = 1_000_000

# Sample data
first_names = [
    "John", "Emma", "Michael", "Sophia", "David",
    "Olivia", "James", "Ava", "William", "Mia","Vijay","Madhan"
]

last_names = [
    "Smith", "Johnson", "Williams", "Brown", "Jones",
    "Garcia", "Miller", "Davis", "Wilson", "Taylor","Kumar"
]

departments = [
    "IT", "HR", "Finance", "Marketing",
    "Operations", "Sales", "Support"
]

designations = {
    "IT": ["Developer", "Senior Developer", "Tech Lead", "Architect"],
    "HR": ["HR Executive", "HR Manager"],
    "Finance": ["Accountant", "Financial Analyst", "Finance Manager"],
    "Marketing": ["Marketing Executive", "Marketing Manager"],
    "Operations": ["Operations Executive", "Operations Manager"],
    "Sales": ["Sales Executive", "Sales Manager"],
    "Support": ["Support Engineer", "Support Lead"]
}

locations = [
    "Chennai", "Bangalore", "Hyderabad",
    "Mumbai", "Pune", "Delhi", "Kolkata"
]

# Date range for hire date
start_date = datetime(2010, 1, 1)
end_date = datetime(2025, 12, 31)

date_diff = (end_date - start_date).days

output_file = "employees_1M.csv"

with open(output_file, mode="w", newline="", encoding="utf-8") as file:
    writer = csv.writer(file)

    # Header
    writer.writerow([
        "Employee_ID",
        "First_Name",
        "Last_Name",
        "Gender",
        "Department",
        "Designation",
        "Salary",
        "Hire_Date",
        "Email",
        "Phone_Number",
        "Location"
    ])

    for emp_num in range(1, NUM_RECORDS + 1):

        first_name = random.choice(first_names)
        last_name = random.choice(last_names)
        gender = random.choice(["Male", "Female"])

        department = random.choice(departments)
        designation = random.choice(designations[department])

        salary = random.randint(300000, 2500000)

        hire_date = start_date + timedelta(days=random.randint(0, date_diff))
        hire_date = hire_date.strftime("%Y-%m-%d")

        employee_id = f"EMP{emp_num:07d}"

        email = (
            f"{first_name.lower()}."
            f"{last_name.lower()}{emp_num}@company.com"
        )

        phone = f"+91{random.randint(6000000000, 9999999999)}"

        location = random.choice(locations)

        writer.writerow([
            employee_id,
            first_name,
            last_name,
            gender,
            department,
            designation,
            salary,
            hire_date,
            email,
            phone,
            location
        ])

print(f"Successfully generated {NUM_RECORDS:,} records in {output_file}")
```

### Step-2 Data Loading

* SQL*Loader control file
* Bulk loading support
* Error handling through BAD and LOG files
### 2.1 - Create Stage A Table in Oracle
```Sql
CREATE TABLE EMPLOYEES_STAGE_A (
    EMPLOYEE_ID   VARCHAR2(20),
    FIRST_NAME    VARCHAR2(50),
    LAST_NAME     VARCHAR2(50),
    GENDER        VARCHAR2(10),
    DEPARTMENT    VARCHAR2(50),
    DESIGNATION   VARCHAR2(50),
    SALARY        NUMBER,
    HIRE_DATE     DATE,
    EMAIL         VARCHAR2(100),
    PHONE_NUMBER  VARCHAR2(20),
    LOCATION      VARCHAR2(50)
);
```
### 2.2 -Prepare a control file for SQL*Loader
```Ctl
OPTIONS (SKIP=1)
LOAD DATA
INFILE 'C:\Users\VijayakumarM\Python\employees_1M.csv'
BADFILE 'employees.bad'
DISCARDFILE 'employees.dsc'

APPEND
INTO TABLE EMPLOYEES_STAGE_A

FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
TRAILING NULLCOLS

(
 EMPLOYEE_ID,
 FIRST_NAME,
 LAST_NAME,
 GENDER,
 DEPARTMENT,
 DESIGNATION,
 SALARY,
 HIRE_DATE DATE "YYYY-MM-DD",
 EMAIL,
 PHONE_NUMBER,
 LOCATION
)
```
### 2.3 - Load CSV Data into Oracle Using SQL*Loader
<img width="953" height="199" alt="Screenshot 2026-06-17 185419" src="https://github.com/user-attachments/assets/d954c7db-6728-4ed5-bb3b-80b3222b66bb" />

### Step-3 Data Transformation

* BULK COLLECT with LIMIT
* FORALL
* SAVE EXCEPTIONS
* Data cleansing
* Business rule implementation
### 3.1 - Create Stage B Table
```sql
CREATE TABLE HR.EMPLOYEES_STG_B (
    EMPLOYEE_ID      VARCHAR2(20),
    FULL_NAME        VARCHAR2(120),
    GENDER           VARCHAR2(10),
    DEPARTMENT       VARCHAR2(50),
    DESIGNATION      VARCHAR2(50),
    MONTHLY_SALARY   NUMBER,
    ANNUAL_SALARY    NUMBER,
    HIRE_DATE        DATE,
    EMAIL            VARCHAR2(100),
    LOCATION         VARCHAR2(50),
    LOAD_DATE        DATE
);
```
### 3.2 -Develop PL/SQL Transformation Procedure for Implement BULK COLLECT & FORALL for Bulk Processing
```sql
CREATE OR REPLACE PROCEDURE HR.PRC_EMP_STAGEA_TO_STAGEB
IS

    CURSOR C_EMP IS
        SELECT *
        FROM hr.employees_stage_a;

    TYPE T_EMP_TAB IS TABLE OF C_EMP%ROWTYPE;
    L_EMP_TAB T_EMP_TAB;

    TYPE T_EMP_B_TAB IS TABLE OF HR.EMPLOYEES_STG_B%ROWTYPE;
    L_EMP_B_TAB T_EMP_B_TAB := T_EMP_B_TAB();

    L_LIMIT NUMBER := 1000;

BEGIN

    OPEN C_EMP;

    LOOP

        FETCH C_EMP BULK COLLECT
        INTO L_EMP_TAB
        LIMIT L_LIMIT;

        EXIT WHEN L_EMP_TAB.COUNT = 0;

        L_EMP_B_TAB.DELETE;

        FOR I IN 1 .. L_EMP_TAB.COUNT
        LOOP

            IF L_EMP_TAB(I).SALARY > 0
               AND L_EMP_TAB(I).EMAIL IS NOT NULL
            THEN

                L_EMP_B_TAB.EXTEND;

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).EMPLOYEE_ID :=
                    L_EMP_TAB(I).EMPLOYEE_ID;

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).FULL_NAME :=
                    UPPER(TRIM(L_EMP_TAB(I).FIRST_NAME)
                    || ' '
                    || TRIM(L_EMP_TAB(I).LAST_NAME));

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).GENDER :=
                    L_EMP_TAB(I).GENDER;

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).DEPARTMENT :=
                    L_EMP_TAB(I).DEPARTMENT;

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).DESIGNATION :=
                    L_EMP_TAB(I).DESIGNATION;

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).MONTHLY_SALARY :=
                    L_EMP_TAB(I).SALARY;

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).ANNUAL_SALARY :=
                    L_EMP_TAB(I).SALARY * 12;

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).HIRE_DATE :=
                    L_EMP_TAB(I).HIRE_DATE;

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).EMAIL :=
                    LOWER(L_EMP_TAB(I).EMAIL);

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).LOCATION :=
                    INITCAP(L_EMP_TAB(I).LOCATION);

                L_EMP_B_TAB(L_EMP_B_TAB.COUNT).LOAD_DATE :=
                    SYSDATE;

            END IF;

        END LOOP;

        BEGIN

            FORALL I IN 1 .. L_EMP_B_TAB.COUNT
                SAVE EXCEPTIONS

                INSERT INTO HR.EMPLOYEES_STG_B
                VALUES L_EMP_B_TAB(I);

            COMMIT;

        EXCEPTION
            WHEN OTHERS THEN

                IF SQLCODE = -24381 THEN
                    FOR J IN 1 .. SQL%BULK_EXCEPTIONS.COUNT
                    LOOP
                        DBMS_OUTPUT.PUT_LINE(
                            'Error Index=' ||
                            SQL%BULK_EXCEPTIONS(J).ERROR_INDEX ||
                            ' Error Code=' ||
                            SQL%BULK_EXCEPTIONS(J).ERROR_CODE
                        );
                    END LOOP;
                ELSE
                    DBMS_OUTPUT.PUT_LINE(SQLERRM);
                END IF;

        END;

    END LOOP;

    CLOSE C_EMP;

EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Procedure Failed : ' || SQLERRM);

END PRC_EMP_STAGEA_TO_STAGEB;
```
### 3.3 - Load Transformed Data into Stage B
```sql
begin
HR.PRC_EMP_STAGEA_TO_STAGEB;
end;
```

### Step-4 Data Export

* UTL_FILE based CSV export
* Header generation
* Exception handling
* Proper file management
### 4.1 - Create Oracle Directory for File Export And verify it.
```sql
#Create Oracle Directory from SYSDBA user;

CREATE OR REPLACE DIRECTORY CSV_DIR AS 'C:\Users\VijayakumarM\Oracle_practice\csv_files';

#Grant permissions from SYSDBA user:
GRANT READ, WRITE ON DIRECTORY CSV_DIR TO HR;

#Verify:
SELECT *
FROM ALL_DIRECTORIES
WHERE DIRECTORY_NAME = 'CSV_DIR';
```
### 4.2 - Develop UTL_FILE Export Procedure
```sql
CREATE OR REPLACE PROCEDURE HR.EXPORT_EMP_STAGEB_CSV
IS
    L_FILE      UTL_FILE.FILE_TYPE;
    L_HEADER    VARCHAR2(1000);

    CURSOR C_EMP IS
        SELECT EMPLOYEE_ID,
               FULL_NAME,
               GENDER,
               DEPARTMENT,
               DESIGNATION,
               MONTHLY_SALARY,
               ANNUAL_SALARY,
               HIRE_DATE,
               EMAIL,
               LOCATION,
               LOAD_DATE
        FROM HR.EMPLOYEES_STG_B;

    L_RECORD VARCHAR2(4000);

BEGIN

    -- Open file in write mode
    L_FILE := UTL_FILE.FOPEN(
                  'CSV_DIR',
                  'employees_stageb.csv',
                  'W',
                  32767);

    -- Write Header
    L_HEADER :=
          'EMPLOYEE_ID,FULL_NAME,GENDER,DEPARTMENT,DESIGNATION,'
       || 'MONTHLY_SALARY,ANNUAL_SALARY,HIRE_DATE,EMAIL,LOCATION,LOAD_DATE';

    UTL_FILE.PUT_LINE(L_FILE, L_HEADER);

    -- Write Data
    FOR REC IN C_EMP
    LOOP

        L_RECORD :=
               REC.EMPLOYEE_ID || ','
            || '"' || REPLACE(REC.FULL_NAME,'"','""') || '"' || ','
            || REC.GENDER || ','
            || REC.DEPARTMENT || ','
            || REC.DESIGNATION || ','
            || REC.MONTHLY_SALARY || ','
            || REC.ANNUAL_SALARY || ','
            || TO_CHAR(REC.HIRE_DATE,'YYYY-MM-DD') || ','
            || REC.EMAIL || ','
            || REC.LOCATION || ','
            || TO_CHAR(REC.LOAD_DATE,'YYYY-MM-DD HH24:MI:SS');

        UTL_FILE.PUT_LINE(L_FILE, L_RECORD);

    END LOOP;

    UTL_FILE.FCLOSE(L_FILE);

    DBMS_OUTPUT.PUT_LINE('CSV Export Completed Successfully.');

EXCEPTION

    WHEN UTL_FILE.INVALID_PATH THEN
        DBMS_OUTPUT.PUT_LINE('Invalid Directory Path');

    WHEN UTL_FILE.INVALID_MODE THEN
        DBMS_OUTPUT.PUT_LINE('Invalid File Mode');

    WHEN UTL_FILE.WRITE_ERROR THEN
        DBMS_OUTPUT.PUT_LINE('File Write Error');

    WHEN UTL_FILE.INVALID_OPERATION THEN
        DBMS_OUTPUT.PUT_LINE('Invalid File Operation');

    WHEN OTHERS THEN

        IF UTL_FILE.IS_OPEN(L_FILE) THEN
            UTL_FILE.FCLOSE(L_FILE);
        END IF;

        DBMS_OUTPUT.PUT_LINE(
            'Unexpected Error: ' || SQLERRM
        );

END HR.EXPORT_EMP_STAGEB_CSV;
```
### 4.3 -Export Stage B Data to CSV
```sql
Begin
HR.PRC_EMP_STAGEA_TO_STAGEB;
end;
```

## Performance Techniques

* BULK COLLECT
* FORALL
* LIMIT clause
* Batch commits
* SQL*Loader Direct Path Loading

## Learning Outcomes

* Oracle ETL development
* Bulk processing optimization
* File-based data integration
* Large-volume data migration techniques

## Author

Vijayakumar M

Oracle PL/SQL Developer | PostgreSQL | Python | Unix
