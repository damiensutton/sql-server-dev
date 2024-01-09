Placeholder file to keep folder in source control.
For development purposes, the /sqlbackup file is mapped to the container so you can easily copy files to/from the container for ease of restore.

Here's the script to make life even easier:


USE [master]
GO

-- Step 1: Run this code block to determine the location of the data file and the log file
SELECT 
  DB_NAME([database_id]) [database_name]
, [file_id]
, [type_desc] [file_type]
, [name] [logical_name]
, [physical_name]
FROM sys.[master_files]
WHERE [database_id] IN (DB_ID('PBW'))
ORDER BY [type], DB_NAME([database_id]);

-- Step 2: Check the filelist of the .bak file (optional)

RESTORE FILELISTONLY FROM DISK = '/var/opt/sqlserver/backup/PBW-20240102.bak' WITH FILE = 1

-- Step 3: Perform the restore to the locations determined in Step 1.
ALTER DATABASE [PBW]
SET SINGLE_USER
WITH ROLLBACK AFTER 30

RESTORE DATABASE [PBW] FROM DISK='/var/opt/sqlserver/backup/PBW-20240102.bak'
WITH REPLACE,
MOVE 'PBW' TO '/var/opt/sqlserver/data/PBW.mdf',
MOVE 'PBW_Log' TO '/var/opt/sqlserver/log/PBW_log.ldf'

ALTER DATABASE [PBW] SET MULTI_USER
GO
