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

Vijay

Oracle PL/SQL Developer | PostgreSQL | Python | Unix
