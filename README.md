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

### Data Generation

* Python-based employee data generator
* Generates 1,000,000 records
* Realistic employee attributes
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



### Data Loading

* SQL*Loader control file
* Bulk loading support
* Error handling through BAD and LOG files

### Data Transformation

* BULK COLLECT with LIMIT
* FORALL
* SAVE EXCEPTIONS
* Data cleansing
* Business rule implementation

### Data Export

* UTL_FILE based CSV export
* Header generation
* Exception handling
* Proper file management

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
