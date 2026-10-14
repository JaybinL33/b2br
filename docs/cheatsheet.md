# Born2beRoot 최소 치트시트

Rocky 최소 설치 후 `jungblee`의 일반 셸에서 처음부터 재실습하는 순서다. **필요한 줄만 개별 실행**하며 자동 실행 스크립트로 사용하지 않는다. 설치·디스크 계획과 개념·근거는 [targets](targets.md), 현황은 [README](../README.md), 증거·이력은 [학습 기록](learning-log.md)에 둔다. 다른 로그인명을 쓰면 명령의 `jungblee`를 바꾼다.

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

`jungblee`의 현재 셸에서 **sudo 없이** 실행한다. 이전 비밀번호 입력을 거치는 이 경로로 `difok` 비교가 적용되도록 한다.

```bash
passwd
```

jungblee 변경에 성공한 뒤 root 비밀번호를 변경한다.

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

두 번째 줄은 첫 명령 직후 실행한다. `sudo: sorry, you must have a tty to run sudo`처럼 **TTY가 필요하다는 명시적 거절**이 확인 기준이다. 0이 아닌 종료 상태는 보조 증거이며 명시적 거절을 이미 확인했다면 재검사할 필요는 없다. `a password is required`만 나오거나 다른 오류로 실패하면 통과로 판단하지 않고 적용 정책·명령·오류를 확인한다. `-c`는 추가하지 않는다. [개념과 공식 읽기 범위](targets.md#tty-실행-조건-확인).

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

## 이후 단위

SSH·firewalld·SELinux 설정·접속 검증 → monitoring.sh·cron·wall → 평가 재현·README·서명. 각 목표는 [전체 지도](targets.md#전체-학습-지도)에서 확인한다.
