https://book.hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html?highlight=redis

# Enumeration

## Nmap

```bash
nmap $IP -sV --script=redis-info -p 6379
```

## Authentication

```bash
redis-cli -h $IP

# if the server returns the following, it means there's an authentication set.
$IP:6379> info
NOAUTH Authentication required.

# Authenticate with password
$IP:6379> AUTH $PWD
```

## Folders and Files to look for

```bash
redis.conf

# Typical location
/etc/redis/redis.conf
/usr/local/etc/redis.conf

# Password paramter
requirepass
```

```bash
/opt/redis-files
```