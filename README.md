# Sql Server: Cursor that automate shrink all logs
With this sql cursor statement, is possible to Shrink all Database Logs that increased your size, applicable at large and complex MSSQL instances with many databases (excluding master, model, msdb and tempdb)
.
```sql
SET NOCOUNT ON;

DECLARE  @name                 VARCHAR(MAX)
		,@DBName               SYSNAME
        ,@LogicalLogFile       SYSNAME
        ,@SQL_CMD_ALTER_0	   VARCHAR(MAX);

DECLARE clear_logs_cursor CURSOR FOR
SELECT name FROM master.dbo.sysdatabases
WHERE name NOT IN ('master','model','msdb','tempdb');

OPEN clear_logs_cursor;
FETCH NEXT FROM clear_logs_cursor INTO @name;

WHILE @@FETCH_STATUS = 0
   BEGIN
	  PRINT 'Inicia a limpeza de LOG na base de dados ' + @name;
	  
		SET @DBName = @name;
 
		-- Log file
		SELECT  @LogicalLogFile = name
		FROM    sys.master_files
		WHERE   database_id = db_id(@DBName)
				AND type = 1;
--SELECT @name AS [@name],  @LogicalLogFile   AS [@LogicalLogFile]
		
	  SET @SQL_CMD_ALTER_0 = 'USE "' + @name + '";
							 ALTER DATABASE "' + @name + '" SET RECOVERY SIMPLE;
							 DBCC SHRINKFILE ("' + @LogicalLogFile + '", 1);
							 ALTER DATABASE "' + @name + '" SET RECOVERY FULL;';
	  
	  EXEC(@SQL_CMD_ALTER_0);
		

      FETCH NEXT FROM clear_logs_cursor INTO @name;
      
      PRINT 'Sucesso! A pr�xima base de dados a realizar limpeza de LOG ser� ' + @name;
      
   END;
PRINT 'Finaliza a limpeza de LOGs';

CLOSE clear_logs_cursor;
DEALLOCATE clear_logs_cursor;
GO
```
