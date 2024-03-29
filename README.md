# SQL Check

### Spazio Disco in GB per ogni DB:
```
SELECT sys.databases.name,
CONVERT(integer,SUM(size)*8/1024) AS [Total GB disk space]
FROM sys.databases
JOIN sys.master_files
ON sys.databases.database_id=sys.master_files.database_id
GROUP BY sys.databases.name
ORDER BY [Total GB disk space] DESC
```

### RAM in uso per ogni DB:
```
SELECT DB_NAME(database_id),
COUNT (1) * 8 / 1024 AS MBUsed
FROM sys.dm_os_buffer_descriptors
GROUP BY database_id
ORDER BY COUNT (*) * 8 / 1024 DESC
GO
```
