- VNC (Virtual Network Computing)는 원격지에 있는 데스크탑 화면을 네트워크를 통해 보고 제어할 수 있게 해주는 시스템이다.
- RDP와 비슷하지만, 주로 리눅스, 유닉스 기반에서 사용된다.
- 서버가 화면을 전송하고, 클라이언트가 입력을 전달한다.

## Enumerate

```bash
sudo apt install tigervnc-viewer

vncviewer $IP:[port]

hydra -s [port] -P [wordlist] -t 4 $IP vnc
```

```bash
vncviewer [-passwd passwd.txt] <IP>::5901

Default **password is stored** in: ~/.vnc/passwd

# if it looks encrypted can use the following tool to crack it
git clone https://github.com/jeroennijhof/vncpwd
make
vncpwd <vnc password file>
```