# 1. Database Information
## List all databases
sql
SELECT name, database_id, state_desc
FROM sys.databases;
## Database size
sql
EXEC sp_databases;
## Detailed information about a specific database
sql
USE [database_name];
EXEC sp_helpdb [database_name];
# 2. Table and Column Information
## List all tables in a database
sql
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE';
## List all columns for a table
sql
SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'your_table_name';
## Find column usage across tables
sql
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME = 'your_column_name';
# 3. Indexes and Constraints
## List indexes for a table
sql
SELECT i.name AS IndexName, i.type_desc AS IndexType, c.name AS ColumnName
FROM sys.indexes i
JOIN sys.index_columns ic ON i.index_id = ic.index_id AND i.object_id = ic.object_id
JOIN sys.columns c ON ic.object_id = c.object_id AND ic.column_id = c.column_id
WHERE i.object_id = OBJECT_ID('your_table_name');
## List constraints for a table
sql
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE TABLE_NAME = 'your_table_name';
# 4. Stored Procedures and Functions
## List all stored procedures
sql
SELECT name, create_date, modify_date
FROM sys.procedures;
## List all user-defined functions
sql
SELECT name, type_desc, create_date, modify_date
FROM sys.objects
WHERE type IN ('FN', 'IF', 'TF'); -- Scalar, Inline, and Table functions
# 5. Views
## List all views in a database
sql
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.VIEWS;
# 6. User and Role Information
## List all users
sql
SELECT name, create_date, modify_date
FROM sys.database_principals
WHERE type IN ('S', 'U'); -- SQL users and Windows users
## List all roles
sql
SELECT name, type_desc
FROM sys.database_principals
WHERE type = 'R'; -- Roles
## List all users in a specific role
sql
SELECT dp.name AS UserName, rp.name AS RoleName
FROM sys.database_role_members rm
JOIN sys.database_principals dp ON rm.member_principal_id = dp.principal_id
JOIN sys.database_principals rp ON rm.role_principal_id = rp.principal_id
WHERE rp.name = 'role_name';
# 7. Session and Connection Information
## List active sessions
sql
SELECT session_id, login_name, status, host_name, program_name
FROM sys.dm_exec_sessions
WHERE is_user_process = 1;
## List active connections
sql
SELECT session_id, client_net_address, connect_time, local_net_address
FROM sys.dm_exec_connections;
# 8. Execution Statistics
## Query execution statistics
sql
SELECT total_worker_time AS CPU_Time, total_elapsed_time AS Elapsed_Time,
       creation_time, execution_count, text AS Query_Text
FROM sys.dm_exec_query_stats
CROSS APPLY sys.dm_exec_sql_text(sql_handle);
## Index usage statistics
sql
SELECT OBJECT_NAME(s.object_id) AS TableName, i.name AS IndexName, 
       user_seeks, user_scans, user_lookups, user_updates
FROM sys.dm_db_index_usage_stats AS s
JOIN sys.indexes AS i ON s.object_id = i.object_id AND s.index_id = i.index_id;
# 9. Lock and Blocking Information
## Check for locking and blocking
sql
SELECT request_session_id AS BlockingSessionID, 
       blocking_session_id AS BlockedBy,
       wait_type, wait_time, wait_resource
FROM sys.dm_exec_requests
WHERE blocking_session_id <> 0;
# 10. Backup and Restore Information
## List recent backups
sql
SELECT database_name, backup_start_date, backup_finish_date, type,
       physical_device_name AS BackupLocation
FROM msdb.dbo.backupset bs
JOIN msdb.dbo.backupmediafamily bmf ON bs.media_set_id = bmf.media_set_id
ORDER BY backup_start_date DESC;
