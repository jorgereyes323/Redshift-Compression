# Database Column Compression Implementation

This document contains SQL scripts for implementing column compression on database tables to improve query performance through reduced disk I/O operations.

## Overview

Column compression significantly impacts query performance by reducing the space required on disk, which decreases the number of disk I/O operations needed during query execution.

## Process Steps

### Step 1: Record Count Analysis
Get baseline record counts for all tables before modification:
- `cstg_case`
- `cstg_mastercase` 
- `cstg_debttransaction`
- `cstg_debtcasestatushistory`
- `cstg_agency`
- `cstg_program`
- `cstg_invoice`
- `cstg_debtorindividualmaster`
- `cstg_address`
- `cstg_debtorcontactmaster`
- `cstg_lookupvalue`

### Step 2: Table Backup
Rename existing tables to create backup copies with `_COPY` suffix.

### Step 3: Table Recreation
Create new tables with optimized:
- **Distribution Style**: KEY distribution
- **Sort Keys**: Optimized for query patterns
- **Column Compression**: Various encoding types (zstd, az64, lzo)

## Compression Encodings Used

- **zstd**: Primary compression for most columns
- **az64**: Used for administrative cost fields
- **lzo**: Used for timestamp fields (create/update/delete)

## Table Configurations

### cstg_case
- **Distribution Key**: `mastercaseid`
- **Sort Keys**: `caseid`, `mastercaseid`
- **Primary Key**: `caseid`

### cstg_mastercase  
- **Distribution Key**: `mastercaseid`
- **Sort Key**: `mastercaseid`
- **Primary Key**: `mastercaseid`

## Permissions

All tables include standard permission grants for:
- `dwdmsusers` (SELECT)
- `dwfrbusers` (SELECT) 
- `dwappusers` (SELECT)
- `dwloadusers` (SELECT, INSERT, UPDATE, DELETE)

## Usage

Execute the SQL statements in the provided order to implement column compression on your database tables. Ensure you have appropriate backups before running the scripts.