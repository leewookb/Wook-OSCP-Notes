### Tip

- Bruteforce with `Hydra` using the username found from **SMTP**

### **Commands**

| **Command** | **Description** |
| --- | --- |
| `USER username` | Identifies the user. |
| `PASS password` | Authentication of the user using its password. |
| `STAT` | Requests the number of saved emails from the server. |
| `LIST` | Requests from the server the number and size of all emails. |
| `RETR id` | Requests the server to deliver the requested email by ID. |
| `DELE id` | Requests the server to delete the requested email by ID. |
| `CAPA` | Requests the server to display the server capabilities. |
| `RSET` | Requests the server to reset the transmitted information. |
| `QUIT` | Closes the connection with the POP3 server. |

### Connect

```bash
curl --ssl-reqd 'pop3://<IP>' --user user:password
curl -k 'pop3s://<IP>' --user user:password -v
curl -k 'pop3s://<IP>/1' --user user:password --request PETR # 첫 번째 메일 요청
```

### OpenSSL - TLS Encrypted Interaction

```bash
openssl s_client -connect <IP>:pop3s
```