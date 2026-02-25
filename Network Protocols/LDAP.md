## Tools

- nmap
- ldapsearch
- windapsearch
- ldapdomaindump
- nxc

### Anonymous Bind

- `x`: simple authentication (null authentication)
- `H`: Host URL
- `b`: base DN

```bash
ldapsearch -x -H ldap://$IP -b "dc=example,dc=com"
```

### Enumeration

```bash
# nmap
nmap $IP --script=ldap*
nmap $DC_IP -n -sV --script "ldap* and not brute" -p 389

# ldapsearch
ldapsearch -x -H ldap://$IP -b "dc=example,dc=com" "(objectClass=user)" sAMAccountName description

ldapsearch -x -H ldap://$IP -b "dc=example,dc=com" "(objectClass=*)" 
ldapsearch -x -H ldap://$IP -b "dc=monitored,dc=htb" "(sAMAccountName=*)"
ldapsearch -x -H ldap://$IP -b "dc=monitored,dc=htb" "(uid=*)"
ldapsearch -x -H ldap://$IP -b "dc=monitored,dc=htb" "(cn=*)"
ldapsearch -x -H ldap://$IP -b "dc=moniotred,dc=htb" "user"

ldapsearch -H ldap://$IP -D 'ldap@support.htb' -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -b "DC=support,DC=htb"| less

# windapsearch
windapsearch.py --dc-ip $IP -u "" -U

# ldapdomaindump
ldapdomaindump ldap://$IP -u '[domain]\[user]' -p [password] -o [dir]
```