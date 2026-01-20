# Database Column Compression Implementation

This document contains SQL scripts for implementing column compression on database tables to improve query performance through reduced disk I/O operations.

## Overview

Column compression significantly impacts query performance by reducing the space required on disk, which decreases the number of disk I/O operations needed during query execution.

## Process Steps

## Amazon Redshift Encoding Types

### Primary Encodings Used
- **AZ64**: Proprietary encoding for numeric, date, and time data types with high compression ratio and fast decompression
- **ZSTD**: General-purpose compression algorithm recommended for most data types
- **LZO**: High compression ratio for CHAR and VARCHAR columns with long strings or free-form text
- **BYTEDICT**: Effective for columns with low cardinality (few unique values)
- **RAW**: No compression, used for sort key columns and primary keys to maintain performance

### Additional Available Encodings
- **RLE**: Run-Length Encoding
- **DELTA**: Delta encoding
- **MOSTLY8/16/32/80**: For columns with mostly small values
- **TEXT255**: For text columns

## 12-Step Implementation Process

### Step 1: Set Database Schema
```sql
SET SEARCH_PATH TO DW;
```
Establish working schema context for all operations

### Step 2: Capture Current Table Metrics
```sql
SELECT "table",size, tbl_rows FROM SVV_TABLE_INFO WHERE "table" LIKE '%target_pattern%'
```
Document baseline table sizes and row counts for comparison

### Step 3: Record Count Validation
```sql
SELECT COUNT(*) FROM target_table;
```
Verify exact row counts for each table before modification

### Step 4: Create Table Backups
```sql
ALTER TABLE original_table RENAME TO original_table_COPY;
```
Safely preserve existing tables with _COPY suffix

### Step 5: Design Compression Strategy
Analyze column data types and cardinality to select optimal encoding

### Step 6: Create Primary Table with Optimized Encoding
Implement first table with new compression, distribution, and sort keys

### Step 7: Create Secondary Tables
Apply compression optimization to remaining high-priority tables

### Step 8: Configure Distribution Keys
Set appropriate DISTSTYLE (KEY/EVEN) based on join patterns

### Step 9: Optimize Sort Keys
Define SORTKEY columns for query performance optimization

### Step 10: Apply Column-Level Compression
Implement specific ENCODE settings per column characteristics

### Step 11: Validate Table Structure
Verify new table definitions match requirements and constraints

### Step 12: Performance Verification
```sql
SELECT "table",size, tbl_rows FROM SVV_TABLE_INFO WHERE "table" LIKE '%target_pattern%'
```
Compare before/after metrics to confirm compression effectiveness

## Encoding Strategy by Data Type

| Data Type | Encoding | Reason |
|-----------|----------|---------|
| UUID Primary Keys | RAW | No compression for frequently accessed keys |
| CHAR/VARCHAR (general) | ZSTD | General-purpose compression |
| Fee amounts | BYTEDICT | Low cardinality reference data |
| Timestamps (create/update) | AZ64 | Optimized for date/time data |
| Foreign key references | BYTEDICT | Low cardinality lookups |
| Sort key columns | RAW | Maintain scan performance |

## Table Configuration Guidelines

### Distribution Strategy
- **KEY Distribution**: Use for tables with clear join patterns on specific columns
- **EVEN Distribution**: Use for tables without dominant join columns or smaller tables

### Sort Key Selection
- Primary keys for unique identification
- Foreign keys for join optimization
- Date/timestamp columns for time-based queries
- Frequently filtered columns

## Performance Benefits

- Reduced disk I/O operations
- Faster query execution
- Lower storage costs
- Improved scan performance on compressed columns

## Usage

Execute steps sequentially. Monitor table sizes before/after using SVV_TABLE_INFO to measure compression effectiveness.
