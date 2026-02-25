# Tips

- [ ]  Login Form 만나면 single quote `'` 로 SQL Injection 확인
- [ ]  if `GET` doesn’t work, try `POST`
- [ ]  만약 `/etc/hosts` 에 도메인 맵핑을 해야한다면? subdomain 확인

```bash
echo "10.10.10.1 wook.com" | sudo tee -a /etc/hosts
```

# Tools

- https://github.com/swisskyrepo/PayloadsAllTheThings
- Gobuster, feroxbuster
- cadaver

# Enumeration

### Nmap

```bash
nmap $IP --script=http-enum -sV -p 80
```

### Browser

```bash
View Page Source
robots.txt
sitemap.xml
```

### Subdomains

```bash
theharvester -d [domain] -b [search engine]
amass enum -passive -src -d [domain]
amass enum -active -d [domain]
cat [file with domains] | httprobe
```

### Gobuster

- `b 403,404`: 무시할 상태 코드 지정
- `k`: SSL 인증서 무시
- `-exclude-length <LENGTH>`

```bash
# directory bursting
gobuster dir -u http://$IP -w [wordlist] -x [file extensions] -t [num threads]
gobuster dir -u http://$IP -x [file extensions] -w [wordlist] -U [auth username] -P [auth password] -s [invalid status codes] -t [num threads] 

# file extensions
php,asp,xml,html,md,js,sql,gz,zip

# dns
gobuster dns -d http://$IP -w [wordlist] -t 50

# vhost
gobuster vhost -d http://$IP -w [wordlist] -t 50

# API
{GOBUSTER}/v1 # pattern.txt
{GOBUSTER}/v2 # pattern.txt
gobuster dir -u http://<IP> -w wordlist -p pattern.txt

# wordlists
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt (or medium)
/usr/share/wordlists/dirb/big.txt
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt (API)

```

### feroxbuster

```jsx
feroxbuster -u http://$IP
feroxbuster -u http://$IP -m GET,POST -k
feroxbuster -u http://$IP --filter-status [status code]
```

### Manual Enumeration

```bash
- Check for usernames, potential domains
- Check for places where you can upload files
- check for places where you can provide user input
	- try SQL ' or "
	- if it works, throw xp_cmdshell and try to get a reverse shell (if it is windows)
	- if not, enumerate the table
- If login pages, try default credentials (try with uppercase as well!)
	- admin:admin
	- admin:password
	- root:root
	- root:toor
	- username:username ( if you find a username)
```

### ffuf

```bash
# directories:
ffuf -w [wordlist] -u http://[URL]/FUZZ

# files:
ffuf -w [wordlist] -u http://[URL]/FUZZ -e .aspx,.html,.php,.txt,.pdf -recursion

# Subdomains:
ffuf -w [wordlist] -u http://[URL] -H "Host: FUZZ.[domain]"

# POST Data:	
ffuf -w [wordlist] -X POST -d "[username=admin\&password=FUZZ]" -u http://[URL]

# From File:	
ffuf -request [req.txt] -request-proto http -w [wordlist]

# Creds:		
ffuf -request [req.txt] -request-proto http -mode [pitchfork/clusterbomb] -w [usernames.txt]:[HFUZZ] -w [passwords.txt]:[WFUZZ]
```

```bash
“Good” (Match)
-mc ⇒ status code
-ms ⇒ response size
-mw ⇒ number of words
-ml ⇒ number of lines
-mr ⇒ regex pattern

“Bad” (Filter)
-fc ⇒ status code
-fs ⇒ response size
-fw ⇒ number of words
-fl ⇒ number of lines
-fr ⇒ regex pattern
```

### Git

```bash
# git-dumper
pipx install git-dumper
git-dumper http://[url] [output dir]
cd [dir]
git log

# basic git commands
git status
git log
git log -P
git log --oneline
git log --stat
git log -- [filename]
git show # without paramters
git show [commit hash]
git reset --hard [commit hash]
git checkout [commit] -- [filename]
git branch
git config --list

# look for interesting files
find . -type f \( -name '*config*' -o -name '*setting*' \) | 
grep -iE '\.ya?ml$|\.ini$|\.php$|\.json$|\.conf$|\.config$|\.txt$'

find . -type f \( -name '*config*' -o -name '*setting*' \) |
grep -iE '\.ya?ml$|\.ini$|\.php$|\.json$|\.conf$|\.config$|\.txt$' |
xargs -I {} grep --color -PHair '^(?!(\s{0,}?//\s?|\s{0,}?\*\s{1,}?|\s{0,}?\#\s{1,}?)).*(root|admin|passw|database|db|sql|domain\.tld)' {}

# emails and hostnames
grep -Eair "domain.tld"

# passwords
grep -Eair "(secret|passwd|password)\ ?[=|:]\ ?['|\"]?\w{1,}['|\"]?" \
--exclude '*.css' --exclude '*.js'

# Git Revision History
git rev-list --all | xargs git -P grep --color -Eair "(secret|passwd|password)\ ?[=:]\ ?['|\"]?\w{1,}['|\"]?" | sort -u
```

### cadaver

- WebDAV 기반 클라이언트
- WebDAV는 웹 서버를 파일 서버처럼 쓸 수 있게 해주는 기술.

```bash
cadaver http://$IP
or
cadaver http://$IP/webdav

# 접속하면 계정/비밀번호를 입력하라는 메시지가 나올 수 있다.
# 인증 없이 접근 가능하다면 바로 쉘 프롬프트 (dav:/webdav/>)가 열림
  
cadaver http://$IP
put shell.php
quit

curl http://$IP/shell.php
```

### wget

```bash
# 한번에 여러 파일 다운로드 받기
# -r: recursively
# -np: no parent 부모 디렉토리로 거슬러 올라가지 않음
# -nd: no directories 로컬에 디렉토리 구조를 만들지 않고 파일만 다운로드
# -A zip: .zip파일만 다운로드
wget -r -np -nd -A zip http://$IP/zipfiles/
```

# CMS

[Wordpress](https://www.notion.so/Wordpress-2e356e0e1b6a8070a048c5d595b894db?pvs=21) 

# Exploits

[Exploitation](https://www.notion.so/Exploitation-2e256e0e1b6a80cd88e7ced1f2bd26b4?pvs=21)