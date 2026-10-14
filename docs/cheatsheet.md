# Born2beRoot 최소 치트시트

Rocky 최소 설치 때 `jungblee`를 관리자 계정(wheel)으로 생성해 sudo를 사용할 수 있는 상태에서, 해당 일반 셸로 재실습하는 순서다. **필요한 줄만 개별 실행**하며 자동 실행 스크립트로 사용하지 않는다. 설치·디스크 계획과 개념·근거는 [targets](targets.md), 현황은 [README](../README.md), 증거·이력은 [학습 기록](learning-log.md)에 둔다. 다른 로그인명을 쓰면 명령의 `jungblee`를 바꾼다.

## 1. 디스크·VG 확인

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo vgs
```

## 2. 사용자·그룹

```bash
id
getent group user42 sudo
id jungblee
```

없는 그룹만 만들고, 계정에 빠진 그룹이 있을 때 추가한다. 이미 가입되어 있으면 변경을 건너뛴다.

```bash
sudo groupadd user42
sudo groupadd sudo
sudo usermod -aG user42,sudo jungblee
id jungblee
```

그룹을 변경했다면 `exit`로 로그아웃하고 다시 로그인한 뒤 `id`로 세션 반영을 확인한다.

## 3. 비밀번호 기간 정책

새 계정 기본값은 아래 **두 방식 중 하나만** 사용한다. 먼저 libeconf 지원 빌드인지 의존성을 조회한다.

```bash
rpm -q --requires shadow-utils | grep libeconf
```

**libeconf 지원 시:** 별도 파일을 만들거나 편집한다.

```bash
sudo mkdir -p /etc/login.defs.d
sudoedit /etc/login.defs.d/90-local.defs
```

파일에 다음 세 값을 적는다. 기존 파일이면 해당 항목을 중복 없이 수정한다.

```text
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7
```

**미지원 시 대안:** 별도 파일 방식 대신 `sudoedit /etc/login.defs`로 위 세 항목을 수정한다. 지원 여부가 불분명하면 패키지 정보를 먼저 확인한다. 선택한 방식에서 해당 항목만 수정한다.

별도 파일은 기본 파일 뒤에 읽히며, drop-in은 파일명 사전순으로 처리된다. `90-local.defs`보다 뒤의 파일이 같은 값을 덮을 수 있다. [지원 조건·우선순위 근거](targets.md#3-비밀번호pam).

기존 계정에는 어느 방식이든 `chage`로 기간 값을 적용한다.

```bash
sudo chage -M 30 -m 2 -W 7 root
sudo chage -M 30 -m 2 -W 7 jungblee
sudo chage -l root
chage -l jungblee
```

최소 2일·최대 30일·경고 7일을 대조한다. `chage`는 비밀번호 자체를 바꾸지 않으며 `Account expires`는 비밀번호 만료와 별개다. 최소 2일은 일반 사용자의 재변경을 제한할 수 있다.

참고: [chage(1)](https://man7.org/linux/man-pages/man1/chage.1.html), [login.defs(5)](https://man7.org/linux/man-pages/man5/login.defs.5.html).

## 4. PAM 진입점과 관리 상태 읽기

먼저 비밀번호 변경의 진입점을 읽는다.

```bash
cat /etc/pam.d/passwd
```

그 파일의 `password` 항목이 `system-auth`를 참조하면 이어서 읽는다.

```bash
cat /etc/pam.d/system-auth
```

편집 전에 authselect의 현재 프로필과 관리 구성의 일관성을 확인한다.

```bash
sudo authselect current
sudo authselect check
```

`password` 스택에 `pam_pwquality.so`가 연결되어 있고 authselect 검사가 유효하면 기존 품질 설정을 읽는다. 연결이 없거나 검사 오류가 있으면 [PAM 설명](targets.md#3-비밀번호pam)으로 돌아가 관리 구성을 먼저 확인한다.

```bash
cat /etc/security/pwquality.conf
ls /etc/security/pwquality.conf.d
```

libpwquality는 drop-in → 본체 순으로 읽으며 PAM 인수가 우선한다. 본체에 같은 활성 설정이 있는지 함께 확인한 뒤 규칙을 작성한다. [연결과 우선순위 설명](targets.md#3-비밀번호pam).

## 5. 별도 품질 설정 파일 작성

본체와 기존 분리 파일에 충돌하는 활성 설정이 없는지 확인한다. 기존 파일이 있으면 해당 내용부터 읽고 중복·우선순위를 정리한다. 디렉터리가 없다면 먼저 생성한다.

```bash
sudo mkdir -p /etc/security/pwquality.conf.d
```

```bash
sudoedit /etc/security/pwquality.conf.d/90-local.conf
```

다음 내용을 적는다. `enforce_for_root`는 값 없이 적는다. 옵션의 의미는 [품질 규칙 설명](targets.md#품질-규칙과-선택-이유)에 둔다.

```text
minlen = 10
ucredit = -1
lcredit = -1
dcredit = -1
ocredit = 0
maxrepeat = 3
usercheck = 1
difok = 7
enforcing = 1
enforce_for_root
```

사전 검사 등 나머지 기본 설정은 유지한다.

## 6. 비밀번호 변경과 날짜 확인

`jungblee`의 현재 셸에서 **sudo 없이** 실행한다. 이전 비밀번호 입력을 거치는 이 경로로 `difok` 비교가 적용되도록 한다. `sudo passwd jungblee`로 대신하면 이전 암호 비교가 생략되므로 같은 검사가 아니다.

```bash
passwd
```

jungblee 변경에 성공한 뒤 root 비밀번호를 변경한다. root는 이전 암호 비교만 예외이며 다른 품질 규칙은 적용한다. 추가로 만든 비밀번호 사용 계정이 있다면 해당 계정도 갱신·기간 적용 대상에 포함한다.

```bash
sudo passwd root
```

두 변경에 성공하면 기간 정보를 확인한다.

```bash
chage -l jungblee
sudo chage -l root
```

성공 기준은 **마지막 변경일이 실제 날짜, 만료일이 그로부터 30일 뒤, 최소 2일·최대 30일·경고 7일 유지**다. 정상 변경 성공과 각 위반 조건의 거절 테스트는 별도 검증이다.

오류가 나면 다음 변경으로 넘어가지 않고 비밀번호를 제외한 오류 메시지만 확인한다. 성공 후 재변경은 최소 2일 제한의 영향을 받을 수 있으므로 실제 오류를 보고 판단한다. `Password inactive`·`Account expires`의 `never`는 이번에 추가 설정할 항목이 아니다. 비밀번호는 기록하지 않는다.

## 7. sudo 현재 정책 읽기

```bash
sudo -l
sudo cat /etc/sudoers
sudo ls /etc/sudoers.d
```

현재 사용자에게 허용된 명령과 Defaults를 본다. 본체에서 활성 `%wheel` 규칙과 include 지시를 찾아 별도 파일 목록과 연결한다. 그룹 가입 자체와 권한 허용은 다르다. [읽을 부분과 세 묶음의 목표](targets.md#현재-정책부터-읽기)를 참고한다.

## 8. sudo 실행 조건과 입출력 로그 설정

앞 조회에서 wheel의 허용 규칙, secure_path, `/etc/sudoers.d` include를 확인한다. 필요한 권한·경로가 이미 있으면 중복 추가하지 않는다. 구성이 다르면 [정책 설명](targets.md#현재-정책부터-읽기)을 바탕으로 먼저 확인한다. 점 없는 파일명으로 편집하며 `-P`로 기본 sudoers 모드도 맞춘다.

```bash
sudo visudo -P -f /etc/sudoers.d/90-b2br
```

다음 다섯 줄을 적는다.

```text
Defaults passwd_tries=3
Defaults badpass_message="Authentication failed. Please try again."
Defaults requiretty
Defaults log_input, log_output
Defaults iolog_dir="/var/log/sudo"
```

저장 뒤 **현재 터미널을 유지**하고 전체 문법·파일 속성과 적용 목록을 확인한다.

```bash
sudo visudo -c
sudo -l
```

본체와 별도 파일 모두 `parsed OK`이고 오류가 없어야 한다. 적용 목록에서 위 설정을 대조한다. 오류가 있으면 다음 단계로 넘어가지 않는다. 로그 디렉터리는 sudo가 필요할 때 생성한다. 입출력 로그는 공개 Git에 올리지 않는다. [설정 근거](targets.md#별도-sudo-파일-작성-안내).

## 9. 로그 생성과 출력 재생

```bash
sudo cat
```

비밀이 아닌 `b2br-log-test`를 입력하고 Enter를 누른다. cat이 같은 문장을 출력하면 빈 줄에서 Ctrl+D로 종료한다.

```bash
sudo sudoreplay -d /var/log/sudo -l
```

목록에서 cat 명령의 TSID를 찾고 아래 `ID`를 **그 실제 값으로 바꿔** 재생한다.

```bash
sudo sudoreplay -d /var/log/sudo ID
```

출력 재생 다음에는 같은 `ID`의 표준입력·터미널 입력을 함께 재생한다.

```bash
sudo sudoreplay -d /var/log/sudo -f stdin,ttyin ID
```

입력 재생에서 `b2br-log-test`가 보이는지 확인한다. `-f stdin,ttyin`은 두 입력 종류를 함께 선택한다.

`-d`는 우리가 설정한 로그 디렉터리, `-l`은 세션 목록이다. 재생에서 문장이 보이는지 확인한다. 기본 재생은 출력, `-f stdin,ttyin` 재생은 입력 확인이다. 문장이 보여도 두 종류 각각에 모두 기록됐다고 단정하지 않는다. [설명과 공식 읽기 범위](targets.md#로그-생성과-출력-재생-확인).

## 10. TTY 실행 조건 검증

`requiretty`가 적용되는 일반 사용자 셸에서 실행한다. 앞에 sudo를 추가하거나 root 셸에서 실행하지 않는다.

```bash
setsid -w sudo -n true </dev/null
echo $?
```

두 번째 줄은 첫 명령 직후 실행한다. `sudo: sorry, you must have a tty to run sudo`처럼 **TTY가 필요하다는 명시적 거절**이 확인 기준이다. 0이 아닌 종료 상태는 보조 증거이며 명시적 거절을 이미 확인했다면 재검사할 필요는 없다. `a password is required`만 나오거나 다른 오류로 실패하면 통과로 판단하지 않고 적용 정책·명령·오류를 확인한다. [개념과 공식 읽기 범위](targets.md#tty-실행-조건-확인).

## 11. sudo 인증 경로와 실패 검사

먼저 인증 경로만 조회한다.

```bash
cat /etc/pam.d/sudo
```

`auth`·`account`의 `include`·`substack` 대상 파일을 읽는다. 예를 들어 `system-auth`를 참조하면 다음을 실행하고 추가 참조도 따라간다.

```bash
cat /etc/pam.d/system-auth
```

`pam_faillock.so` 등 잠금 처리가 있거나 경로가 불명확하면 고의 실패 검사는 보류한다. 모듈 인수·설정 파일·현재 실패 기록을 확인하고 잠금 위험을 판단한다. 정책 비활성화나 기록 초기화는 하지 않는다. [판단 근거와 읽을 범위](targets.md#인증-경로와-실패-횟수-확인).

**인증 경로와 잠금 위험을 확인한 뒤에만**, 비밀번호 인증이 필요한 일반 사용자 터미널에서 한 번 검사한다.

```bash
sudo -k
sudo -v
```

프롬프트마다 의도적으로 틀린 값을 입력한다. 재시도 중 `Authentication failed. Please try again.`이 표시되고, 세 번째 실패 뒤 네 번째 비밀번호 요청 없이 종료되는지 확인한다. 마지막 실패 요약 문구와 사용자 지정 문구는 다르므로 문구가 정확히 세 번 출력될 것을 요구하지 않는다. 프롬프트가 없거나 조기 종료·잠금 메시지가 나오면 반복하지 말고 원인을 확인한다. 실제 비밀번호나 입력값은 기록하지 않는다.

## 12. SSH·firewalld·SELinux 시작 조회

일반 사용자 셸에서 각 줄을 실행한다.

```bash
systemctl is-active sshd firewalld
systemctl is-enabled sshd firewalld
getenforce
ip -br addr
```

| 조회 | 결과 읽기 |
|---|---|
| `is-active` | sshd, firewalld 순서로 현재 상태를 읽는다. `active`는 실행 중, `inactive`는 비활성, `failed`는 실패 상태다. 두 서비스를 각각 확인한다. |
| `is-enabled` | 같은 순서로 부팅 시 활성화 설정을 읽는다. `enabled`와 현재 실행 중인지는 별개다. `disabled`·`masked` 등 다른 값은 그대로 확인한 뒤 원인을 살핀다. |
| `getenforce` | `Enforcing`은 정책 강제, `Permissive`는 거절을 강제하지 않고 기록, `Disabled`는 SELinux 비활성이다. 현재 모드 조회이며 재부팅 후 상태까지 증명하지 않는다. |
| `ip -br addr` | 인터페이스 이름·상태·할당 주소와 접두사 길이를 읽는다. `lo`는 루프백이다. 다른 인터페이스의 주소가 보여도 호스트에서 바로 접속 가능한지는 VM 네트워크 방식·포트 전달과 함께 확인한다. |

서비스가 없다는 오류나 예상과 다른 상태도 그대로 확인한다. 이 묶음은 시작 상태 조회이며 최종 기준은 **4242 포트의 일반 사용자 SSH 접속 성공과 root SSH 접속 거절**이다. [세 역할과 공식 읽을 범위](targets.md#5-sshfirewalldselinux)를 바탕으로 리스닝·방화벽 규칙·SELinux 정책·실제 접속을 이어서 확인한다.

## 13. SSH 설정 파일 조회

```bash
sudo cat /etc/ssh/sshd_config
sudo ls /etc/ssh/sshd_config.d
```

본체에서 활성 `Include`의 위치·대상, `Port`, `PermitRootLogin`, `Match`를 찾는다. 목록에 나온 파일은 Include 패턴과 대조한 뒤 해당 경로를 `sudo cat`으로 읽는다. Include가 다른 경로를 가리키면 그 경로를 따른다. 파일명이나 주석만으로 적용값을 확정하지 않는다. 여러 `Port`는 함께 적용될 수 있으므로 4242 추가가 기존 포트 제거를 뜻하지 않는다. [읽기 순서와 공식 근거](targets.md#5-sshfirewalldselinux).

## 14. 별도 SSH 설정 작성과 검사

본체의 Include와 기존 포함 파일을 먼저 확인한 뒤 작성한다. 기존 암호 정책·PAM 설정은 유지한다.

```bash
sudoedit /etc/ssh/sshd_config.d/00-b2br.conf
```

다음 두 줄을 저장한다.

```text
Port 4242
PermitRootLogin no
```

먼저 설정 검사를 실행하고, 오류가 없으면 유효 설정을 확인한다.

```bash
sudo sshd -t
sudo sshd -T | grep -e '^port ' -e '^permitrootlogin '
```

`-t`는 오류 없이 종료해야 하며, 이어지는 출력의 기대값은 다음과 같다.

```text
port 4242
permitrootlogin no
```

추가 port 줄이나 다른 값·오류가 있으면 Include 경로와 중복 설정을 다시 확인한다. **SELinux의 4242 허용 준비 전에는 sshd를 재시작하지 않는다.** 검사 통과 후 방화벽·접속 경로 준비와 서비스 반영, 실제 리스닝·접속 검증으로 이어간다. [선택 이유와 공식 검사 옵션](targets.md#5-sshfirewalldselinux).

## 15. SELinux에 SSH 포트 등록

관리 도구를 설치하고 TCP 4242를 SSH 포트 유형에 등록한 뒤 확인한다. 설치가 성공한 다음 줄로 진행한다.

```bash
sudo dnf install policycoreutils-python-utils
sudo semanage port -a -t ssh_port_t -p tcp 4242
sudo semanage port -l | grep ssh_port_t
```

`ssh_port_t`의 `tcp` 목록에 4242가 포함되는 것이 기대 결과다. 기존 22가 이 목록에 남아 있어도 SSH 리스닝이나 방화벽 개방을 뜻하지 않는다.

**이미 정의되었다는 오류가 나오면** 임의로 `-m`으로 바꾸지 않는다. `sudo semanage port -l`에서 TCP 4242의 기존 유형·범위를 확인한다. 이미 `ssh_port_t`에 포함되면 재등록 없이 확인하고, 다른 유형이면 용도를 먼저 확인한다.

SELinux 등록은 방화벽 개방과 별개다. 다음은 firewalld 현황 확인이며 아직 sshd를 재시작하지 않는다. [개념과 공식 읽기 범위](targets.md#5-sshfirewalldselinux).

## 16. firewalld 활성 zone과 규칙 조회

```bash
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all
```

첫 출력에서 대상 인터페이스가 연결된 zone을 찾는다. 두 번째 출력은 **기본 zone**의 현재 실행 구성이므로 같은 zone인지 대조한다. 다르면 아래 `ZONE`을 실제 활성 zone 이름으로 바꿔 조회한다.

```bash
sudo firewall-cmd --zone=ZONE --list-all
```

zone 이름과 `interfaces`·`sources`·`services`·`ports`·추가 규칙을 읽는다. `ports`가 비어 있어도 서비스로 허용된 포트가 있을 수 있다. 조회 결과를 확인한 뒤 변경할 규칙을 정한다. [zone 개념과 공식 읽기 범위](targets.md#5-sshfirewalldselinux).

## 17. firewalld 규칙 변경과 대조

앞 조회에서 대상 zone을 확인한다. 아래는 `public`이고 `ssh`·`cockpit`·`dhcpv6-client`가 허용된 IPv4 실습 구성의 절차다. zone이 다르면 실제 이름으로 바꾸고 제거 대상도 조회 결과와 대조한다. **DHCPv6를 사용하는 환경에서는 `dhcpv6-client` 제거를 그대로 적용하지 않는다.** VM 콘솔에서 진행하거나 기존 접속을 유지하며 작업한다.

```bash
sudo firewall-cmd --permanent --zone=public --add-port=4242/tcp
sudo firewall-cmd --permanent --zone=public --remove-service=ssh
sudo firewall-cmd --permanent --zone=public --remove-service=cockpit
sudo firewall-cmd --permanent --zone=public --remove-service=dhcpv6-client
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --list-all
sudo firewall-cmd --permanent --zone=public --list-all
```

변경·reload의 성공을 확인하고, 두 조회 모두 `services`가 비어 있으며 `ports`는 `4242/tcp`인지 대조한다. 추가 허용 규칙도 살핀다. runtime의 대상 인터페이스 연결을 확인하되 permanent의 인터페이스 칸이 비었다고 임의로 추가하지 않는다. [개념과 읽을 범위](targets.md#5-sshfirewalldselinux).

## 18. sshd 재시작과 리스닝 확인

SSH 설정 검사·SELinux 포트 등록·방화벽 준비를 마친 뒤 실행한다.

```bash
sudo systemctl restart sshd
sudo systemctl is-active sshd
sudo ss -lntp
```

기대 결과는 `active`, sshd 프로세스의 4242 리스닝, 22 리스닝 없음이다. IPv4·IPv6로 두 줄이 표시되는 것은 정상일 수 있다. 오류나 다른 포트가 보이면 설정과 서비스 상태를 확인한다. 다음으로 VirtualBox 네트워크 설정과 호스트의 SSH 접속 경로를 확인하고 일반 사용자 접속·root 거절을 검증한다.

## 19. VirtualBox 포트 전달과 호스트 접속

어댑터 연결 방식이 **일반 NAT인지 먼저 확인**한다. 일반 NAT의 내장 DHCP를 사용하는 경우 Port Forwarding Rules에 다음 규칙을 작성하고 저장한다.

| 항목 | 값 |
|---|---|
| Name | `ssh` |
| Protocol | `TCP` |
| Host IP | `127.42.42.42` |
| Host Port | `4242` |
| Guest IP | 빈칸: 내장 DHCP 사용 조건 |
| Guest Port | `4242` |

Host IP `127.42.42.42`와 Host Port `4242`는 이 문서의 일관된 접속 예시다. 실제 저장한 Host IP·Port가 다르면 이후 모든 접속 명령도 함께 바꾼다. 호스트에서 사용 가능한 다른 포트를 선택해도 되며, 아래 SSH 명령의 `-p`에도 그 값을 쓴다. 게스트 SSH 포트는 과제 요구인 `4242`를 유지한다.

수동 고정 IP라면 Guest IP에 실제 VM 주소를 지정한다. Guest IP 생략과 달리 **Guest Port는 지정해야 한다.** 다른 네트워크 방식이면 이 규칙을 그대로 적용하지 않고 접속 경로를 확인한다.

포트 전달 창을 확인한 뒤 **바깥 VM 설정 창도 OK로 저장**한다. 이후 **호스트 터미널**에서 실행한다.

```bash
ssh -p 4242 jungblee@127.42.42.42
```

최초 접속에서 제시된 키가 ED25519이면 **VM 콘솔**에서 다음 공개키 지문을 조회해 비교한다.

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

다른 알고리즘이 제시되면 해당 호스트 공개키 파일의 지문을 비교한다. 일치할 때 호스트의 접속 질문에 `yes`를 입력하고, 비밀번호는 터미널 요청에 입력한다. 불일치하면 승인하지 않고 대상·키를 확인한다. 접속 후 다음을 실행한다.

```bash
hostname
whoami
```

이 실습의 기대값은 각각 `jungblee42`, `jungblee`다. 다른 로그인명으로 재실습하면 그 계정과 호스트명에 맞춰 대조한다. [조건과 공식 근거](targets.md#5-sshfirewalldselinux).

## 20. root SSH 접속 거절 확인

일반 사용자 SSH 세션에서 종료해 **호스트 셸로 돌아온 뒤** 같은 전달 경로로 검사한다.

```bash
exit
```

아래 `4242`는 호스트 포트 예시다. 일반 사용자 접속에 성공한 실제 Host Port가 다르면 `-p` 값을 그 포트로 바꾼다.

```bash
ssh -p 4242 -o NumberOfPasswordPrompts=1 root@127.42.42.42
```

비밀번호를 요청하면 올바른 root 비밀번호를 터미널에 입력한다. 기대 결과는 인증 거절과 root 셸에 진입하지 못하는 것이다. `NumberOfPasswordPrompts=1`은 클라이언트의 비밀번호 질문 횟수 제한이며 서버의 root 차단 정책이 아니다. 연결 실패·잘못된 비밀번호만으로 root 차단 성공을 판단하지 않고, 일반 사용자 접속 성공 및 `permitrootlogin no`와 함께 대조한다. [옵션과 검증 근거](targets.md#5-sshfirewalldselinux).

## 21. monitoring 첫 조회: OS와 CPU

호스트에서 일반 사용자로 다시 접속한다. `4242`는 예시이며 실제 선택한 Host Port로 바꾼다.

```bash
ssh -p 4242 jungblee@127.42.42.42
```

접속한 VM 셸에서 실행한다.

```bash
uname -a
lscpu
```

`uname -a`에서 커널·OS·아키텍처 표기를, `lscpu`에서 `CPU(s)`·`Socket(s)`를 찾는다. VM에 보이는 논리 CPU 수와 소켓 수를 구분하고 호스트 실제 하드웨어 수로 단정하지 않는다. 출력의 의미를 확인한 뒤 해당 지표만 코드로 옮긴다. [공식 자료와 읽을 범위](targets.md#6-monitoringsh와-cronwall).

## 22. CPU 계산용 출력 읽기

VM 셸에서 실행한다.

```bash
lscpu -p=SOCKET,CPU
```

`#`로 시작하는 설명 줄과 데이터 행을 구분한다. 첫 열의 소켓 ID는 중복을 제거한 종류 수, 둘째 열의 CPU는 데이터 행 수로 이해한다. `-p`는 기본적으로 온라인 CPU만 출력하므로 앞서 본 온라인 목록과 대조한다. 출력에서 이 관계를 확인한 뒤 스크립트 첫 단위를 작성한다. [옵션과 계산 근거](targets.md#6-monitoringsh와-cronwall).

## 23. monitoring 첫 구현 예시

`monitoring.sh`는 34절의 한 코드 블록을 기준으로 관리한다. 처음 학습할 때는 Architecture·CPU physical·vCPU 부분부터 원자료와 대조하고 이후 항목을 순서대로 읽는다.

전체 코드의 해당 부분은 [34절](#34-전체-출력을-wall-한-번으로-전달)에 모았다.

출력의 Architecture는 `uname -a`, CPU physical은 온라인 CPU에 나타난 서로 다른 소켓 수, vCPU는 온라인 CPU 데이터 행 수와 대조한다. [각 처리 단계의 의미](targets.md#monitoring-첫-구현의-흐름).

## 24. monitoring 메모리·디스크 원자료 읽기

VM 셸에서 조회한다.

```bash
free -m
df -BM --total -x tmpfs -x devtmpfs
```

`free`의 Mem 행에서 total·used·free·shared·buff/cache·available을 구분하고 Swap 행과 섞지 않는다. `-m`은 MiB 단위다. `df`에서는 파일시스템별 Size·Used·Avail·Use%·마운트 지점과 total 행을 읽는다. `-BM`은 1,048,576바이트 단위로 표시하고 `-x`는 지정한 파일시스템 유형을 제외한다. 이 조회의 total은 선택된 마운트 파일시스템의 합계이며 가상 디스크 전체 용량이나 swap 용량이 아니다.

표시값은 반올림되므로 행별 숫자의 합과 total이 정확히 일치한다고 가정하지 않는다. 먼저 집계 대상·단위·사용량과 가용량의 의미를 정하고 원문 예시와 대조한다. 계산 코드와 출력 형식은 그 다음 단위에서 작성한다. [공식 읽기 범위와 원문 표기의 한계](targets.md#monitoring-메모리디스크의-읽기-범위).

## 25. monitoring 메모리·디스크 출력

원자료의 열과 집계 대상을 확인하고 전체 코드의 Memory Usage·Disk Usage 두 줄과 대조한다.

전체 코드의 해당 부분은 [34절](#34-전체-출력을-wall-한-번으로-전달)에 모았다.

메모리는 Mem 행, 디스크는 total 행의 used/total로 사용률을 계산한다. 원문 예시의 `MB`·`Gb` 표기를 따르지만 실제로 메모리와 디스크 사용량은 MiB, 디스크 전체 표시값은 GiB다. 디스크 퍼센트는 GiB로 반올림한 표시값이 아니라 MiB 원자료로 계산한다. `df`의 Use%와 동일한 값일 필요는 없다. [계산 선택과 awk 읽기 범위](targets.md#monitoring-메모리디스크의-계산과-awk).

## 26. monitoring CPU·부팅 시각 조회와 부분 출력

먼저 VM에서 원자료를 읽는다.

```bash
LC_ALL=C vmstat 1 2
uptime -s
```

vmstat의 헤더에서 `id`가 15열인지 확인하고 두 번째 수치 행을 읽는다. `uptime -s`의 `YYYY-MM-DD HH:MM:SS` 날짜·시각을 확인한다. CPU 열 배치를 확인한 뒤 전체 코드의 CPU load·Last boot 두 줄과 대조한다.

전체 코드의 해당 부분은 [34절](#34-전체-출력을-wall-한-번으로-전달)에 모았다.

CPU 출력은 직전 1초 구간의 `100 - id`이며, I/O 대기·steal도 포함한다. 시간에 따라 변하므로 이전 조회와 숫자가 같아야 하는 것은 아니다. 부팅 시각 출력은 같은 `uptime -s` 원자료의 분 단위 값과 대조한다. 다른 도구의 시각과 자동으로 일치한다고 가정하지 않는다. [측정 구간과 해석](targets.md#monitoring-cpu와-부팅-시각의-읽기-범위).

## 27. monitoring LVM·TCP 연결 원자료 조회

VM 셸에서 조회한다.

```bash
lsblk -o NAME,TYPE
ss -Htn state established
```

lsblk의 TYPE 열에서 `lvm`이 있는지 확인한다. 장치 이름과 트리 관계를 읽되 장치 행 수를 LVM 활성 여부와 혼동하지 않는다. ss는 헤더 없는 현재 ESTABLISHED TCP 소켓 목록이다. 각 행의 로컬·상대 주소와 포트를 읽고 행 수를 확인한다. 연결 수를 로그인 사용자 수로 해석하지 않는다. 조회가 오류 없이 끝났는지도 함께 확인한다. [공식 읽기 범위](targets.md#monitoring-lvmtcp-연결의-읽기-범위)를 대조한 뒤 해당 출력 코드를 작성한다.

## 28. monitoring LVM·TCP 연결 부분 출력

VM 터미널의 원자료를 전체 코드의 LVM 조건과 TCP 집계에 대조한다.

전체 코드의 해당 부분은 [34절](#34-전체-출력을-wall-한-번으로-전달)에 모았다.

LVM 출력은 TYPE에 `lvm`이 있는지, TCP 출력은 같은 시점의 필터된 소켓 행 수와 대조한다. 연결 수는 조회 시점에 따라 바뀔 수 있다. [조건 검사와 집계의 의미](targets.md#monitoring-lvmtcp-부분-구현의-읽기-범위).

## 29. monitoring 로그인 세션·IP·MAC 원자료 조회

VM 터미널에서 조회한다.

```bash
who
ip -br address
ip -br link
```

who에서 사용자명과 터미널을 읽고, 같은 사용자가 여러 행에 있는지 확인한다. 세션 행 수와 중복을 제외한 사용자 수를 구분한다. ip 출력은 같은 인터페이스 이름으로 대조한다. `enp0s3`의 IPv4와 접두사 길이, 링크의 MAC 주소를 찾고 `lo` 및 IPv6 주소와 구분한다. 다른 환경이면 실제 인터페이스 이름을 확인한다. [공식 읽기 범위](targets.md#monitoring-로그인-세션ipmac의-읽기-범위)를 읽은 뒤 집계·출력 방식을 정한다.

## 30. monitoring 사용자 수·IPv4·MAC 부분 출력

VM 터미널에서 `who`와 네트워크 원자료를 대조한다. 다음 예시는 대상 NIC가 `enp0s3`이고 IPv4 주소가 하나인 구성이다. 이름이나 주소 개수가 다르면 먼저 대상 범위를 정한다.

전체 코드의 해당 부분은 [34절](#34-전체-출력을-wall-한-번으로-전달)에 모았다.

User log는 같은 사용자명을 한 번만 센 값이다. Network는 선택한 NIC의 IPv4에서 접두사 길이를 제외한 값과 같은 NIC의 MAC을 대조한다. [집계 선택과 공식 읽기 범위](targets.md#monitoring-사용자수ipv4mac-부분-구현).

## 31. sudo 횟수를 위한 로그 원자료 조회

VM 터미널에서 최근 sudo 로그의 형식을 확인한다.

```bash
sudo journalctl _COMM=sudo -n 10 --no-pager -o cat
```

최근 10개 로그 메시지를 읽는 조회이며 실행 횟수 계산은 아니다. 명령 실행 기록과 세션 열기·닫기 등 다른 메시지를 구분하고, 조회 자체도 sudo 실행으로 기록될 수 있음을 고려한다. 형식을 확인한 뒤 집계 범위와 기준을 정한다. [읽기 범위](targets.md#sudo-횟수-원자료의-읽기-범위).

## 32. sudo I/O 세션 목록의 형식 확인

VM 터미널에서 기존 I/O 로그 디렉터리의 목록을 조회한다.

```bash
sudo sudoreplay -d /var/log/sudo -l | tail -n 5
```

`-d`로 로그 디렉터리, `-l`로 목록 모드를 지정하고 마지막 다섯 줄의 형식을 읽는다. 목록은 파일명 또는 시퀀스 번호 순서이며 `tail`은 표본 확인용이다. TSID와 사용자·터미널·실행 명령 등 필드를 구분하고 실제 실행 기록과 대조한다. 조회 자체도 sudo I/O 세션을 만들 수 있다. 출력 확인 전에는 전체 횟수를 계산하지 않는다. [범위와 공식 근거](targets.md#sudo-io-세션-목록의-읽기-범위).

## 33. sudo I/O 세션 수 부분 출력

목록의 한 줄이 한 세션임을 확인하고 전체 코드의 sudo_count 집계와 대조한다.

전체 코드의 해당 부분은 [34절](#34-전체-출력을-wall-한-번으로-전달)에 모았다.

스크립트 내부의 sudoreplay 앞에는 sudo를 붙이지 않는다. 출력은 현재 보관된 I/O 세션 수이며 수동 `sudo bash` 실행 세션도 포함될 수 있다. 조회 오류 없이 목록을 읽었는지 함께 확인한다. [권한·집계 범위](targets.md#sudo-io-세션-집계의-실행-범위).

## 34. 전체 출력을 wall 한 번으로 전달

아래는 앞 단위에서 직접 적용한 코드를 한곳에 모은 `monitoring.sh`다. 현재 학습 구성의 재현 기준이며, 오류 처리와 남은 평가 검증의 범위는 targets·학습 기록을 따른다. 대상 NIC는 `enp0s3`·IPv4 하나이며, 지표 해석과 한계는 위 단위 및 targets에서 확인한다. 기존 파일이 있으면 내용을 대조하고 **조각을 중복 추가하지 않는다.**

```bash
#!/bin/bash
{
    architecture=$(uname -a)
    physical=$(lscpu -p=SOCKET | grep -v '^#' | sort -u | wc -l)
    virtual=$(lscpu -p=CPU | grep -v '^#' | wc -l)

    printf '#Architecture: %s\n' "$architecture"
    printf '#CPU physical : %s\n' "$physical"
    printf '#vCPU : %s\n' "$virtual"

    LC_ALL=C free -m | awk '$1 == "Mem:" {printf "#Memory Usage: %s/%sMB (%.2f%%)\n", $3, $2, $3 / $2 * 100}'
    LC_ALL=C df -m --total -x tmpfs -x devtmpfs | awk '$1 == "total" {printf "#Disk Usage: %s/%.0fGb (%.0f%%)\n", $3, $2 / 1024, $3 / $2 * 100}'

    LC_ALL=C vmstat 1 2 | awk 'END {printf "#CPU load: %.1f%%\n", 100 - $15}'
    printf '#Last boot: %s\n' "$(uptime -s | cut -c 1-16)"

    if lsblk -rn -o TYPE | grep -qx lvm; then
        printf '#LVM use: yes\n'
    else
        printf '#LVM use: no\n'
    fi
    printf '#Connections TCP : %s ESTABLISHED\n' "$(ss -Htn state established | wc -l)"

    users=$(who | awk '{print $1}' | sort -u | wc -l)
    ip_address=$(ip -4 -o address show dev enp0s3 | awk '{print $4}' | cut -d / -f 1)
    mac_address=$(cat /sys/class/net/enp0s3/address)
    printf '#User log: %s\n' "$users"
    printf '#Network: IP %s (%s)\n' "$ip_address" "$mac_address"

    sudo_count=$(sudoreplay -d /var/log/sudo -l | wc -l)
    printf '#Sudo : %s cmd\n' "$sudo_count"
} | wall
```

VM 터미널에서 먼저 구문을 검사하고, 통과하면 실행한다.

```bash
bash -n monitoring.sh
sudo bash monitoring.sh
```

VM 콘솔 tty1과 SSH 터미널을 모두 로그인 상태로 두고 같은 12개 항목이 한 방송으로 수신되는지 각각 확인한다. Architecture처럼 긴 줄은 wall이 화면 줄을 나눌 수 있어 화면 줄 수 대신 항목 내용을 대조한다. 오류가 없는지도 확인한다. 수동 sudo 실행에서는 I/O 로깅 PTY 때문에 같은 화면에 중복 전달될 수 있어 수신 경로를 구분한다. 로깅을 끄지 않고 최종 단일 수신은 root cron 실행에서 확인한다. [그룹·파이프·wall 읽기 범위](targets.md#monitoring-그룹-출력과-wall).

## 35. root 소유 실행 파일 설치와 crond 상태 조회

검토한 `monitoring.sh`가 있는 VM 디렉터리에서 실행한다.

```bash
sudo install -o root -g root -m 755 monitoring.sh /usr/local/sbin/monitoring.sh
ls -l /usr/local/sbin/monitoring.sh
systemctl status crond --no-pager
```

설치본의 소유자·그룹이 `root root`, 권한이 `-rwxr-xr-x`인지 확인한다. crond의 현재 상태와 활성화 정보를 읽는다. 설치본은 원본의 복사이므로 이후 원본을 수정하면 검토 후 다시 설치해야 한다. 이 단계는 파일 설치와 서비스 상태 조회이며 예약 등록은 별도 단계다. [공식 자료와 설계 이유](targets.md#monitoring-설치-경로소유권과-crond).

## 36. root crontab 등록과 예약 방송 확인

VM에서 root의 기존 예약을 편집한다.

```bash
sudo crontab -e
```

기존 내용을 보존하고 같은 monitoring 예약을 중복 추가하지 않는다. 아래는 root 사용자 crontab 내용이다. 기존 PATH 설정이 있다면 필요한 경로를 대조해 한 설정으로 정리한다.

```cron
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

저장한 뒤 등록 내용을 조회한다.

```bash
sudo crontab -l
```

root 사용자 열이나 sudo를 명령에 추가하지 않는다. crond 재시작은 필요 없다. VM의 시각 기준 다음 00·10·20·30·40·50분 경계에 SSH·콘솔 각각에서 방송이 한 번씩 수신되는지 확인한다. 목록 조회 성공과 예약 실행 성공을 구분하고, 부팅 실행은 별도로 검증한다. [시간 필드·환경·검증 범위](targets.md#root-crontab의-읽기-범위).

## 37. 재부팅 후 서비스·접속·부팅 예약 확인

편집 내용을 저장한 뒤 VM에서 재부팅한다.

```bash
sudo reboot
```

SSH 연결이 끊기면 VM 콘솔에서 LUKS 암호를 입력하고 부팅·로그인한다. 호스트에서 저장한 포트 전달 경로로 다시 접속한다. 현재 실습 경로의 예시는 다음과 같다.

```bash
ssh -p 4242 jungblee@127.42.42.42
```

접속한 VM에서 조회한다.

```bash
systemctl is-active sshd firewalld crond
getenforce
sudo journalctl -b -u crond --no-pager
sudo journalctl -b -t CROND --no-pager
```

서비스는 명령에 나열한 순서대로 `active`, SELinux는 `Enforcing`인지 대조한다. journal의 `-b`는 이번 부팅, `-u crond`는 서비스, `-t CROND`는 작업 로그의 식별자로 범위를 한정한다. 이번 부팅의 monitoring 명령 실행 기록과 오류 여부를 읽는다. 부팅 직후 로그인 사용자가 없었다면 wall 화면이 없을 수 있으므로 기록상 실행과 터미널 수신을 별도로 확인한다. 로그인 후 다음 주기 방송에서 각 터미널의 수신도 대조한다. [검증 범위](targets.md#재부팅-검증의-읽기-범위).

## 38. 스크립트 수정 없이 monitoring 예약 중단·복구

VM에서 root crontab을 편집한다.

```bash
sudo crontab -e
```

PATH와 다른 예약은 유지하고 monitoring의 두 실행 줄만 주석 처리한다.

```cron
#*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
#@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

`sudo crontab -l`로 두 줄이 주석인지 확인한 뒤 다음 10분 경계에 방송이 없는지 관찰한다. crond 전체를 중단하거나 스크립트를 수정하지 않는다. 검증 뒤 같은 편집 명령에서 두 줄의 첫 `#`만 제거하고 저장한다. 다시 목록을 확인하고 다음 주기 방송으로 복구 여부를 검증한다. [중단 범위와 증거](targets.md#monitoring-예약-중단과-복구).

## 39. OS·기본 부팅 대상·대표 GUI 패키지 점검

VM에서 조회한다.

```bash
cat /etc/os-release
systemctl get-default
rpm -q xorg-x11-server-Xorg gnome-shell gdm
```

배포판 이름·릴리스를 읽고 기본 부팅 대상이 무엇인지 확인한다. rpm은 지정한 세 패키지의 설치 여부를 조회한다. `not installed`는 해당 패키지 부재의 근거이며, 모든 그래픽 서버가 없다는 완전한 증명은 아니다. 기본 대상만으로 설치 패키지나 현재 실행 상태를 단정하지 않는다. [전체 설치 확인 범위](targets.md#1-설치-결과와-디스크-확인)와 대조한다.

## 40. 종료된 VDI 기준 백업과 해시 대조

호스트에서 실행한다. 대상 b2br가 `Powered Off`이고 스냅샷·차등 디스크가 없으며 Storage의 실제 연결 경로가 아래 원본과 같은지 먼저 확인한다. Git·VM 폴더 밖에 충분한 여유 공간을 마련한다. [전체 조건과 평가 후 복원](targets.md#평가-사이의-디스크-보존과-복원)을 먼저 읽는다.

새 백업 폴더를 만들고 각 명령의 성공을 확인한 뒤 다음 줄로 진행한다. 폴더가 이미 있으면 중단해 새 이름을 고르며 기존 기준 보관본을 덮어쓰지 않는다.

```bash
mkdir "$HOME/goinfre/b2br-baseline-1ac763cd"
cp -i -- "$HOME/goinfre/VirtualBox VMs/b2br/b2br.vdi" "$HOME/goinfre/b2br-baseline-1ac763cd/b2br.vdi"
cmp -- "$HOME/goinfre/VirtualBox VMs/b2br/b2br.vdi" "$HOME/goinfre/b2br-baseline-1ac763cd/b2br.vdi"
sha1sum "$HOME/goinfre/VirtualBox VMs/b2br/b2br.vdi" "$HOME/goinfre/b2br-baseline-1ac763cd/b2br.vdi"
```

`cmp`가 정상 종료하고 두 해시가 같아야 한다. 복사 취소·실패는 완료가 아니다. 기준 보관본은 실행하거나 VirtualBox에 별도로 등록하지 않는다. 이 절은 재현 안내이며 백업 실행 완료 기록은 아니다.

## 이후 단위

예약·부팅 방송 및 중단 검증 → 평가 재현·README·서명. 각 목표는 [전체 지도](targets.md#전체-학습-지도)에서 확인한다.
