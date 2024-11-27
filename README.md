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


### Latenza di lettura/scrittura sui dischi
```
SELECT

  [AvgReadLatency] = CASE WHEN [num_of_reads] = 0 THEN 0 ELSE ([io_stall_read_ms] / (1.0 +[num_of_reads])) END

  ,[AvgWriteLatency] = CASE WHEN [num_of_writes] = 0 THEN 0 ELSE ([io_stall_write_ms] / (1.0+[num_of_writes])) END

  ,[Latency] = CASE WHEN ([num_of_reads] = 0 AND [num_of_writes] = 0) THEN 0 ELSE ([io_stall] / ([num_of_reads] + [num_of_writes])) END

  ,[AvgBPerRead] = CASE WHEN [num_of_reads] = 0 THEN 0 ELSE ([num_of_bytes_read] / [num_of_reads]) END

  ,[AvgBPerWrite] = CASE WHEN [io_stall_write_ms] = 0 THEN 0 ELSE ([num_of_bytes_written] / [num_of_writes]) END

  ,[AvgBPerTransfer] = CASE WHEN ([num_of_reads] = 0 AND [num_of_writes] = 0) THEN 0 ELSE (([num_of_bytes_read] + [num_of_bytes_written]) / ([num_of_reads] + [num_of_writes])) END

  ,LEFT ([mf].[physical_name], 2) AS [Drive]

  ,DB_NAME ([vfs].[database_id]) AS [DB]

  ,[mf].[physical_name]

FROM

  sys.dm_io_virtual_file_stats (NULL,NULL) AS [vfs]

JOIN

  sys.master_files AS [mf] ON [vfs].[database_id] = [mf].[database_id] AND [vfs].[file_id] = [mf].[file_id]

ORDER BY

  [Latency] DESC

GO
```
