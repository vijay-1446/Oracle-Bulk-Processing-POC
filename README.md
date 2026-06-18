# Oracle-Bulk-Processing-POC
Oracle-ETL-POC/
│
├── README.md
│
├── 01_Data_Generation/
│   └── generate_employee_data.py
│
├── 02_SQLLoader/
│   ├── employees.ctl
│   ├── sqlldr_commands.txt
│   └── create_stage_a.sql
│
├── 03_Data_Transformation/
│   ├── create_stage_b.sql
│   └── prc_stagea_to_stageb.sql
│
├── 04_Data_Export/
│   ├── create_directory.sql
│   └── export_stageb_csv.sql
│
├── Sample_Output/
│   ├── employees_sample.csv
│   └── employees_stageb_sample.csv
│
└── Documentation/
    └── POC_Architecture.png
