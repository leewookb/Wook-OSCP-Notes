## Tools

- dig
- dnsenum
- nslookup

## Enumeration

```bash
# DNS 서버 소프트웨어의 버전을 확인, 알려진 취약점이 있는지 찾는다
dig version.bind chaos txt @$IP

# 서버로 부터 특정 도메인의 NS 레코드를 질의
dig ns $DOMAIN @$DNS_SERVER_IP

# View all available records
dig any $DOMAIN @$DNS_SERVER_IP

# Query a DNS server's version using a class CHAOS query and type TXT
dig CH TXT version.bind @$DNS_SERVER_IP
```

## Zone Transfer

```bash
dig axfr $DOMAIN @$DNS_SERVER_IP
dig axfr wook.com @$IP
dig axfr internal.wook.com @$IP
```

## Subdomain Brute Forcing

### Script

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.<domain> @<DNS server IP> | grep -v ';\\|SOA' | sed -r '/^\\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

### dnsenum

```bash
dnsenum --dnsserver <DNS server IP> --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt <domain>
```