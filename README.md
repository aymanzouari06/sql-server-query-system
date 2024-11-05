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
# 11. Server Configuration and Properties
## List all server configuration settings
sql
EXEC sp_configure;
## Check server properties
sql
SELECT SERVERPROPERTY('MachineName') AS MachineName,
       SERVERPROPERTY('Edition') AS Edition,
       SERVERPROPERTY('ProductLevel') AS ProductLevel,
       SERVERPROPERTY('ProductVersion') AS ProductVersion,
       SERVERPROPERTY('IsClustered') AS IsClustered,
       SERVERPROPERTY('Collation') AS Collation;
# 12. Job and SQL Agent Information
## List all SQL Server Agent jobs
sql
SELECT job_id, name, enabled, description
FROM msdb.dbo.sysjobs;
## Check job history
sql
SELECT job_id, run_date, run_time, run_duration, message
FROM msdb.dbo.sysjobhistory
WHERE step_id = 0; -- 0 indicates job completion
## Job schedule details
sql
SELECT j.name AS JobName, s.name AS ScheduleName, s.enabled, s.freq_type,
       s.freq_interval, s.freq_subday_type, s.freq_subday_interval, s.next_run_date
FROM msdb.dbo.sysjobs j
JOIN msdb.dbo.sysjobschedules js ON j.job_id = js.job_id
JOIN msdb.dbo.sysschedules s ON js.schedule_id = s.schedule_id;
# 13. Data File and Log File Information
## List all data and log files for each database
sql
SELECT DB_NAME(database_id) AS DatabaseName, name AS FileName, type_desc AS FileType,
       physical_name AS FilePath, size, max_size, growth
FROM sys.master_files;
## Check database file sizes and usage
sql
USE [database_name];
EXEC sp_spaceused;
# 14. Performance Metrics
## CPU usage by sessions
sql
SELECT session_id, login_name, status, cpu_time, memory_usage
FROM sys.dm_exec_sessions
WHERE is_user_process = 1
ORDER BY cpu_time DESC;
## Memory usage by database
sql
SELECT DB_NAME(database_id) AS DatabaseName, 
       SUM(page_count) * 8 / 1024 AS MemoryUsed_MB
FROM sys.dm_os_buffer_descriptors
GROUP BY database_id
ORDER BY MemoryUsed_MB DESC;
## Disk I/O by database file
sql
SELECT DB_NAME(database_id) AS DatabaseName, file_id, 
       io_stall_read_ms AS ReadLatency, io_stall_write_ms AS WriteLatency,
       num_of_reads AS Reads, num_of_writes AS Writes
FROM sys.dm_io_virtual_file_stats(NULL, NULL);
# 15. Index Fragmentation
## Check fragmentation of indexes in a database
sql
USE [database_name];
SELECT OBJECT_NAME(i.object_id) AS TableName, i.name AS IndexName, 
       index_type_desc, avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED') AS d
JOIN sys.indexes AS i ON d.object_id = i.object_id AND d.index_id = i.index_id
WHERE avg_fragmentation_in_percent > 10
ORDER BY avg_fragmentation_in_percent DESC;
# 16. System Health and Error Logs
## View SQL Server error log
sql
EXEC sp_readerrorlog 0, 1;
## List most recent critical events from system health session
sql
SELECT top 10 event_data.value('(event/@name)[1]', 'varchar(50)') AS event_name,
              event_data.value('(event/data[@name="error_number"]/value)[1]', 'int') AS error_number,
              event_data.value('(event/data[@name="severity"]/value)[1]', 'int') AS severity,
              event_data.value('(event/data[@name="message"]/value)[1]', 'varchar(max)') AS message,
              event_data.value('(event/@timestamp)[1]', 'datetime') AS [Time]
FROM (
    SELECT CONVERT(xml, event_data) AS event_data
    FROM sys.fn_xe_file_target_read_file('system_health*.xel', NULL, NULL, NULL)
    ) AS tab
ORDER BY [Time] DESC;
# 17. Replication and Mirroring Information
## Check replication status
sql
SELECT name AS ReplicationName, status, publisher, subscriber
FROM msdb.dbo.MSreplication_monitordata;
## Mirroring status of databases
sql
SELECT DB_NAME(database_id) AS DatabaseName, mirroring_state_desc, mirroring_partner_name, mirroring_role_desc
FROM sys.database_mirroring;
# 18. User Permissions and Roles
## List all permissions for a specific user
sql
SELECT dp.name AS Principal, p.permission_name, p.state_desc, o.name AS ObjectName
FROM sys.database_permissions p
JOIN sys.database_principals dp ON p.grantee_principal_id = dp.principal_id
LEFT JOIN sys.objects o ON p.major_id = o.object_id
WHERE dp.name = 'your_username';
## Role memberships for users
sql
SELECT dp1.name AS RoleName, dp2.name AS UserName
FROM sys.database_role_members AS drm
JOIN sys.database_principals AS dp1 ON dp1.principal_id = drm.role_principal_id
JOIN sys.database_principals AS dp2 ON dp2.principal_id = drm.member_principal_id;
# 19. Full-Text Index and Catalog Information
## List full-text catalogs
sql
SELECT name AS FullTextCatalogName, path, is_default
FROM sys.fulltext_catalogs;
## List tables and columns with full-text indexes
sql
SELECT OBJECT_NAME(i.object_id) AS TableName, COL_NAME(i.object_id, c.column_id) AS ColumnName,
       c.type_desc AS DataType, i.name AS FullTextIndexName
FROM sys.fulltext_indexes i
JOIN sys.fulltext_index_columns ic ON i.object_id = ic.object_id
JOIN sys.columns c ON ic.column_id = c.column_id AND ic.object_id = c.object_id;
# 20. Auditing and Security Events
## Check audit settings and status
sql
SELECT name AS AuditName, status_desc, audit_file_path
FROM sys.server_audits;
## Check login audit information
sql
SELECT login_name, event_time, event_type, session_id, host_name, database_name
FROM sys.fn_get_audit_file ('C:\AuditFolder\*', default, default);
