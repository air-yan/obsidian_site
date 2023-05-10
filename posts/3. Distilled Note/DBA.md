# SQL Server

## 1. Stored Procedure
---
### 全局查询存储过程关键字

```sql
SELECT DISTINCT name
FROM sysobjects o
    , syscomments s
WHERE o.id = s.id
    AND TEXT LIKE '%8%'
    AND o.xtype = 'P'
```

### 循环

```sql
DECLARE @param VARCHAR(10)

DECLARE myCursor CURSOR
FOR
SELECT *
FROM (
    VALUES ('2022-01')
        , ('2022-02')
        , ('2022-03')
        , ('2022-04')
        , ('2022-05')
        , ('2022-06')
        , ('2021-12')
        , ('2021-06')
    ) tb([year_month])

OPEN myCursor

FETCH NEXT
FROM myCursor
INTO @param

WHILE @@FETCH_STATUS = 0
BEGIN
    EXEC [dwd].[proc_dwd_psi_insert] @param
        , @param

    FETCH NEXT
    FROM myCursor
    INTO @param
END

CLOSE myCursor

DEALLOCATE myCursor

```

## 2. Optimization
---
### 表空间占用

```sql
-- 查询表
SELECT 
    t.NAME AS TableName,
    s.Name AS SchemaName,
    p.rows AS RowCounts,
    SUM(a.total_pages) * 8 AS TotalSpaceKB, 
    SUM(a.used_pages) * 8 AS UsedSpaceKB, 
    (SUM(a.total_pages) - SUM(a.used_pages)) * 8 AS FreeSpaceKB
FROM 
    sys.tables t
INNER JOIN      
    sys.indexes i ON t.OBJECT_ID = i.object_id
INNER JOIN 
    sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
INNER JOIN 
    sys.allocation_units a ON p.partition_id = a.container_id
LEFT OUTER JOIN 
    sys.schemas s ON t.schema_id = s.schema_id
GROUP BY 
    t.Name, s.Name, p.Rows
ORDER BY 
    SUM(a.total_pages) DESC;

```

### 数据库文件信息查询

```sql
EXEC sp_helpfile;
```

### 改变Log大小

```sql
ALTER DATABASE [topscore_bi] 
MODIFY FILE (NAME = [topscore_bi_log], SIZE = 80GB)
```

### 日志大小查询

```sql
SELECT total_log_size_in_bytes / 1024 / 1024 AS [ 日志的大小 MB]
    , used_log_space_in_bytes / 1024 / 1024 AS [ 日志的占用大小 ]
    , used_log_space_in_percent AS [ 占日志总大小的百分比 ]
    , log_space_in_bytes_since_last_backup / 1024 / 1024 AS [ 自上次日志备份以来使用的空间量 MB ]
FROM sys.dm_db_log_space_usage;

```