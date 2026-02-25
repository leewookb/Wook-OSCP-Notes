### Definition

네트워크 장비 (스위치, 라우터, 프린터 등)의 상태를 모니터링하고 관리하는데 사용되는 UDP 기반 프로토콜

### SNMP 버전

- `v1` ,`v2c` 에서는 커뮤니티 문자열만 알면 내부 정보를 쉽게 열람 가능
- `community string` : SNMP 서비스에 접근하기 위한 일종의 비밀번호
    - 기본값: `public` 읽기, `private` 쓰기
- `OID (Object Identifier` : 장비의 속성을 숫자로 표현한 식별자
- `MIB (Management Information Base)` : SNMP 장비가 관리하는 정보를 데이터베이스처럼 정의한 것

| **버전** | **특징** | **보안성** |
| --- | --- | --- |
| v1 | 구조 간단, 커뮤니티 문자열 사용 | 낮음 (암호화 없음) |
| v2c | v1보다 성능 향상, 여전히 커뮤니티 문자열 | 낮음 |
| v3 | 인증 + 암호화 가능 | 높음 |

## Tools

### onesixtyone

목적

- SNMP 커뮤니티 문자열이 맞는지 빠르게 브루트 포싱하는 도구
- onesixtyone은 기본적으로 SNMP v1만 사용한다

```bash
onesixtyone -c $WORDLIST $IP
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt $IP

# 여기 아웃풋에서 [backup]이 커뮤니티 스트링.
10.129.202.20 [backup] Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64

# ex)
echo "public" > community.txt
onesixtyone -c community.txt 10.10.10.100

# ex)
echo public > community.txt
echo private >> community.txt
echo manager >> community.txt

for ip in $(seq 1 254); do echo 192.168.50.$ip; done > ips
onesixtyone -c community.txt -i ips

```

| 1.3.6.1.2.1.25.1.6.0 | System Processes |
| --- | --- |
| 1.3.6.1.2.1.25.4.2.1.2 | Running Programs |
| 1.3.6.1.2.1.25.4.2.1.4 | Processes Path |
| 1.3.6.1.2.1.25.2.3.1.4 | Storage Units |
| 1.3.6.1.2.1.25.6.3.1.2 | Software Name |
| 1.3.6.1.4.1.77.1.2.25 | User Accounts |
| 1.3.6.1.2.1.6.13.1.3 | TCP Local Ports |

### snmp-check

```bash
snmp-check $IP
snmp-check $IP -c $COMMUNITY_STRING -v2c
snmp-check $IP -c private -v2c
```

### snmpwalk

- SNMP 서비스를 통해 내부 정보를 Enumerate 하는 도구

```bash
snmpwalk -v2c -c public <IP>
snmpwalk -c public -v1 192.168.50.151 1.3.6.1.4.1.77.1.2.25
snmpwalk -v1 -c public 192.168.170.151 -t 10 -Oa 1.3.6.1.2.1.2.2.1.2

# Enum all
snmpwalk -v2c -c public $IP .1 

# Get Extended
snmpwalk -v2c -c public $IP NET-SNMP-EXTEND-MIB::nsExtendObjects
snmpwalk -v2c -c public $IP NET-SNMP-EXTEND-MIB::nsExtendOutputFull

# 위 명령어 이슈 생길시 아래 다운로드
sudo apt-get install snmp-mibs-downloader
```

| **OID** | **Description** |
| --- | --- |
| 1.3.6.1.2.1.25.1.6.0 | System Processes |
| 1.3.6.1.2.1.25.4.2.1.2 | Running Programs |
| 1.3.6.1.2.1.25.4.2.1.4 | Processes Path |
| 1.3.6.1.2.1.25.2.3.1.4 | Storage Units |
| 1.3.6.1.2.1.25.6.3.1.2 | Software Name |
| 1.3.6.1.4.1.77.1.2.25 | User Accounts |
| 1.3.6.1.2.1.6.13.1.3 | TCP Local Ports |
| 1.3.6.1.2.1.2.2.1.2 | Network Interface Name |

### braa

```bash
sudo apt install braa
braa $COMMUNITY_STRING@$ip:.1.3.6.*
```

### nmap

```bash
nmap --script "snmp* and not snmp-brute" $IP
```