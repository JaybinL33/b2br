# Born2beRoot 필수 경로·명령 빠른 참조

**목적별로 경로 → 조회 → 판단 → 평가 변경·원복을 찾는 문서다.** 신규 구축 순서는 [스피드런](speedrun.md), 설정 원문·전체 코드는 [치트시트](cheatsheet.md), 평가 자리에서 따라갈 순서는 [디펜스 진행본](defense.md#평가-자리에서-따라갈-진행-순서), 디렉터리 원리는 [디렉터리 지도](directories.md)를 따른다.

명령 자체의 뜻이나 옵션을 찾으려면 [명령어 중심 참조](commands.md)를 읽는다. 특히 [systemctl](commands.md#systemctl), [usermod](commands.md#usermod), [firewall-cmd](commands.md#firewall-cmd), [crontab](commands.md#crontab)은 조회와 변경의 차이를 먼저 확인한다.

근거는 [과제 v5.2](../en.subject.pdf) 인쇄 p.7~10·12~13·16~17과 [디펜스가 대조한 첨부 평가지](defense.md#먼저-확인할-준비-공백) p.1~4다. **필수**는 평상시 과제 조건, **시연**은 평가 때 요청받을 변경이다. 아래 값은 기존 b2br 구성 기준이며 명령은 이번에 VM에서 실행한 결과가 아니다. 실제 완료·미확인은 [학습 기록](learning-log.md)이 기준이다.

**실행 위치:** 별도 표시가 없으면 **게스트·jungblee 일반 셸**이다. `sudo`가 붙은 줄만 관리자 권한으로 실행한다. **호스트** 블록은 VM 밖 터미널에서 실행한다. **조회**와 **변경** 블록을 한꺼번에 실행하지 않는다. `defenseuser`, `reviewer42`, `ZONE`, `ID`는 설명용 이름이며 안내에 맞게 바꾼다. 경로 표의 **디렉터리**와 **파일**은 역할을 구분한 것이며 실제 링크일 수도 있다.

| 목적 | 바로 가기 |
|---|---|
| OS·부팅·서비스·SELinux | [1](#1-os부팅서비스selinux) |
| 계정·그룹·비밀번호·PAM | [2](#2-계정그룹기간품질-pam) |
| sudo 허용·설정·로그 | [3](#3-sudo-정책과-로그) |
| hostname·파티션·암호화·LVM | [4](#4-hostname과-저장장치) |
| SSH·firewalld | [5](#5-ssh와-firewalld) |
| monitoring·cron | [6](#6-monitoring과-cron) |
| 제출·서명·VDI 보존 | [7](#7-제출서명vdi-보존) |

## 1. OS·부팅·서비스·SELinux

**필수:** 허용 OS의 최신 안정 버전, 그래픽 서버 없음, SSH·방화벽 동작과 부팅 유지, Rocky의 SELinux 시작 시 동작. 현재 설계는 Rocky 10.2 Minimal·Enforcing이다. 버전 조회만으로 평가 시점의 최신 안정 버전 조건을 검증한 것은 아니다.

| 종류·핵심 경로 | 역할 |
|---|---|
| 파일 `/etc/os-release`, `/etc/selinux/config` | 배포판 식별, SELinux 부팅 설정. 현재 실행 모드와 구분 |
| 디렉터리 `/usr/lib/systemd/system`, `/etc/systemd/system` | 패키지 unit, 로컬 설정·활성화 연결. 파일 존재와 실행 상태는 별개 |

**조회 — 게스트:**

```bash
whoami
cat /etc/os-release
systemctl get-default
systemctl is-active sshd firewalld crond
systemctl is-enabled sshd firewalld crond
getenforce
```

`jungblee`, Rocky 10.2, `multi-user.target`, 세 서비스 `active`·현재 구성의 `enabled`, SELinux `Enforcing`을 대조한다. target 하나만으로 GUI 전체 부재를 증명하지 않는다. 이 묶음은 설정 변경 시연이 아니며, 조회가 다르면 즉시 enable·보안 완화를 일괄 실행하지 말고 [디펜스 기본 부팅](defense.md#3-기본-부팅과-로그인)과 원인을 확인한다.

## 2. 계정·그룹·기간·품질 PAM

| 종류·핵심 경로 | 역할 |
|---|---|
| 파일 `/etc/passwd`, `/etc/group` | 계정·그룹 식별 정보. 조회는 `id`, `getent` 사용 |
| 파일 `/etc/shadow`, `/etc/gshadow` | 제한된 비밀번호·그룹 인증 정보. 내용 출력 대신 `chage` 등 목적별 조회 |
| 파일 `/etc/login.defs` | 새 계정의 기간 기본값. 기존 계정에 자동 소급되지 않음 |
| 디렉터리 `/etc/pam.d`; 파일 `passwd`, `system-auth` | 비밀번호 변경 진입점과 공통 PAM 스택. authselect 관리 여부 확인 |
| 파일 `/etc/security/pwquality.conf.d/90-local.conf` | b2br 품질 설정. 본체 `/etc/security/pwquality.conf`·PAM 인수와 함께 적용 순서 판단 |

**조회 — 게스트:**

```bash
id jungblee
getent group sudo user42
chage -l jungblee
sudo chage -l root
sudo authselect current
sudo authselect check
cat /etc/pam.d/passwd /etc/pam.d/system-auth
cat /etc/security/pwquality.conf.d/90-local.conf
```

**필수 판단:** 본인 계정은 `sudo,user42`에 속해야 한다. 기록에는 `wheel,user42,sudo`가 모두 있으며 **그룹 가입과 관리자 권한은 별개**다. 기간은 최소 2일·최대 30일·7일 경고. 품질은 길이 10 이상, 대문자·소문자·숫자, 같은 문자 연속 최대 3, 사용자명 배제, 이전 비밀번호와 차이 7이다. root도 품질을 적용하되 이전 비밀번호 비교는 과제의 예외다. 설정 파일만으로 모든 거절 동작이 검증되지는 않는다. [정책·우선순위 설명](targets.md#3-비밀번호pam).

**시연 변경 — 게스트, 새 사용자·그룹:** 먼저 아래 조회에서 이름 충돌을 확인한다. 계정이 있으면 다른 새 이름을 정하고, `evaluating`이 이미 있으면 groupadd를 생략한다. 조회 오류와 “없음”도 구분한다.

```bash
getent passwd defenseuser
getent group evaluating
```

```bash
# 실제로 없는 새 이름에만 실행
sudo useradd -m defenseuser
sudo passwd defenseuser
# evaluating이 없을 때만 실행
sudo groupadd evaluating
sudo usermod -aG evaluating defenseuser
id defenseuser
sudo chage -l defenseuser
```

`-aG`는 기존 보조 그룹을 보존하며 추가한다. 새 계정의 2/30/7 적용 결과도 확인한다. sudo 그룹 추가는 다음 sudo 단계에서 시연한다. 모든 새 계정에 user42를 추가해야 한다는 별도 조건은 아니다. 정책 기본값이 다르면 원인부터 조사한다. 원래 관리자 셸을 유지하고 새 SSH 세션에서 해당 계정의 신원을 확인한다. 비밀번호는 대화형 프롬프트에만 입력하고 명령 인자·문서에 적지 않는다.

**원복:** SSH·sudo·monitoring 시연까지 끝낸 뒤에만 정리한다. 해당 계정의 로그아웃·프로세스 종료와 보존할 파일을 확인하고, 이번에 만든 일회성 계정에 한해 `sudo userdel -r defenseuser`를 사용한다. `-r`은 홈 등 데이터를 삭제한다. `evaluating`도 이번에 만들었고 다른 사용처가 없을 때만 `sudo groupdel evaluating`로 정리한다. 기존 계정·그룹은 삭제하지 않는다.

## 3. sudo 정책과 로그

**평가 흐름 연결:** 앞에서 만든 같은 계정에 `sudo usermod -aG sudo defenseuser`를 실행하고 `id defenseuser`로 확인한다. SSH·monitoring까지 이 계정을 유지한다.

| 종류·핵심 경로 | 역할 |
|---|---|
| 파일 `/etc/sudoers`, `/etc/sudoers.d/90-b2br` | 허용 규칙과 과제 Defaults. 변경 시 `visudo` 대상 |
| 디렉터리 `/var/log/sudo` | sudo 입출력 세션 기록. 정책 파일과 구분 |

**조회 — 게스트:**

```bash
rpm -q sudo
sudo visudo -c
sudo -l
sudo cat /etc/sudoers.d/90-b2br
sudo ls -ld /var/log/sudo
sudo sudoreplay -d /var/log/sudo -l
```

**필수 판단:** 문법 검사 통과, 인증 3회 제한·사용자 정의 실패 문구·TTY 요구·secure_path·입출력 기록을 설명한다. secure_path는 검색 경로이며 명령 허용 목록이 아니다. 기존 실행 기록은 `%wheel` 규칙에 따른 것이다. 과제용 권장안은 기존 Defaults를 유지한 채 `sudo visudo -f /etc/sudoers.d/90-b2br`에서 `%sudo ALL=(ALL:ALL) ALL`을 추가하고 `sudo visudo -c`로 검사하는 것이다. 이는 아직 VM 적용·검증 전이다. 새 계정은 wheel에 넣지 않고 sudo 그룹 가입 후 `su - defenseuser`로 새 로그인해 `id`, `sudo -l`, `sudo whoami`(기대값 root)를 확인하고 `exit`로 돌아온다. 새 사용자 실행의 로그도 확인한다. [원문·평가 해석과 절차](targets.md#sudo-그룹의-권한-연결과-평가-해석). 실제 정책을 바꾸면 검증·정리 후 최종 디스크 백업·SHA-1·제출 signature를 다시 맞춘다.

**로그 시연 — 게스트:** `sudo cat`을 실행해 비밀 없는 시험 문장을 입력하고 Ctrl+D로 끝낸다. 다시 `sudo sudoreplay -d /var/log/sudo -l`에서 그 명령의 ID를 찾는다. `sudo sudoreplay -d /var/log/sudo ID`로 출력, `sudo sudoreplay -d /var/log/sudo -f stdin,ttyin ID`로 입력을 대조한다. 조회 자체도 로그를 늘릴 수 있다. 다른 세션을 무차별 재생하거나 로그를 원복 명목으로 삭제하지 않는다. 세부 확인은 [디펜스 sudo](defense.md#6-sudo-정책과-로그)에 있다.

## 4. hostname과 저장장치

| 종류·핵심 경로 | 역할 |
|---|---|
| 파일 `/etc/hostname` | 지속 호스트명. 필수 형식은 본인 login+42 |
| 파일 `/etc/fstab`, `/etc/crypttab` | 마운트·swap 설정, 암호화 장치 연결 설정. 조회 때 키 경로 등 민감한 내용 공유에 주의 |
| 장치 경로 `/dev/mapper`; 마운트 지점 `/`, `/home`, `/var/log`, `/boot` | 열린 암호화 장치·LV와 파일시스템의 연결. 디렉터리마다 반드시 별도 디스크인 것은 아님 |

**조회 — 게스트:**

```bash
hostname
cat /etc/hostname
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo pvs
sudo vgs
sudo lvs
findmnt /
findmnt /home
findmnt /var/log
```

**필수 판단:** LVM을 이용한 최소 2개 암호화 파티션 요구를 실제 트리와 연결해 설명한다. b2br은 BIOS Boot·비암호화 `/boot`, LUKS 안 PV→VG→LV, `/`·`/home`·`/var/log`의 ext4와 swap 구성이다. VG 여유와 파일시스템 여유는 다르다. 파티션 크기는 이 구성의 선택이지 과제 고정값이 아니다. 조회 시연을 위해 파티션·암호화·LV를 다시 만들지 않는다.

**시연 변경 — 게스트:** `reviewer42`를 평가자 login+42로 바꾼다.

```bash
sudo hostnamectl set-hostname reviewer42
sudo reboot
```

콘솔에서 LUKS 해제 후 다시 로그인해 `hostname`·`cat /etc/hostname`으로 새 이름의 재부팅 유지를 확인한다. **원복 — 게스트:**

```bash
sudo hostnamectl set-hostname jungblee42
hostname
cat /etc/hostname
```

프롬프트만 보고 판단하지 않는다. 원복 후 재부팅 유지 확인도 권장하지만 변경 시 재부팅 요구와 구분한다. [디펜스 hostname·파티션](defense.md#5-hostname-변경재부팅과-파티션).

## 5. SSH와 firewalld

| 종류·핵심 경로 | 역할 |
|---|---|
| 파일 `/etc/ssh/sshd_config`, `/etc/ssh/sshd_config.d/00-b2br.conf` | 서버 포트·root 로그인 정책. 개인 호스트 키 파일을 출력하는 절차가 아님 |
| 디렉터리 `/etc/firewalld`, `/usr/lib/firewalld` | 로컬 영구 정책과 공급자 정의. 현재 runtime 규칙과 구분 |
| 디렉터리 `/etc/selinux`, `/var/lib/selinux` | SELinux 설정·정책 자료. 포트 허용은 방화벽과 별도이며 관리 도구로 확인 |

**조회 — 게스트:**

```bash
sudo sshd -t
sudo sshd -T | grep -E '^(port|permitrootlogin) '
sudo ss -lntp
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
# ZONE을 실제 게스트 NIC가 속한 zone으로 바꾼다. 현재 구성은 enp0s3/public.
sudo firewall-cmd --zone=ZONE --list-all
sudo firewall-cmd --permanent --zone=ZONE --list-all
```

**필수 판단:** sshd 문법 오류 없음, TCP 4242 리스닝, `permitrootlogin no`, 방화벽 running·실제 zone의 4242 허용을 본다. `--list-ports`만 보면 서비스 이름으로 허용한 통신을 놓치므로 `--list-all`로 읽는다. Match가 있다면 실제 접속 문맥도 대조한다. 4242가 설정됐어도 SELinux·NAT·계정 인증이 각각 맞아야 접속된다.

**시연 — 호스트 새 터미널:** 현재 전달 구성은 호스트 127.42.42.42:4242 → 게스트 4242다. 기존 관리자 세션을 유지한다.

```bash
ssh -p 4242 defenseuser@127.42.42.42
# 접속 성공 후 게스트·defenseuser 셸에서
whoami
id
# 이 세션은 monitoring 수신용으로 유지한다
# 호스트의 다른 터미널: root 로그인은 거절되어야 한다
ssh -p 4242 root@127.42.42.42
```

서버 키 확인이 나오면 콘솔의 공개키 지문과 대조한다. 새 계정 접속 성공과 root 인증 거절을 구분하며 timeout 등 네트워크 실패를 root 정책 성공으로 보지 않는다. 서버·NAT 설정 변경은 이 시연의 필수 절차가 아니다.

**8080 시연 변경·원복 — 게스트:** 원래 8080 규칙이 없는 것을 먼저 확인한다. 아래는 일시적인 runtime 변경이며 permanent 변경을 요구하지 않는다. 한 줄씩 결과를 본다.

```bash
sudo firewall-cmd --zone=ZONE --add-port=8080/tcp
sudo firewall-cmd --zone=ZONE --list-ports
sudo firewall-cmd --zone=ZONE --remove-port=8080/tcp
sudo firewall-cmd --zone=ZONE --list-all
```

추가 후 8080/tcp가 보이고 삭제 후 원래 4242 정책으로 돌아와야 한다. 기존 규칙은 임의 삭제하지 않는다. 별도로 permanent를 수정했다면 그 변경도 원복한다. 8080 서버 설치나 NAT 규칙 추가는 요구가 아니다. [디펜스 방화벽·SSH](defense.md#7-firewalld-4242-확인--8080-추가--삭제).

## 6. monitoring과 cron

| 종류·핵심 경로 | 역할 |
|---|---|
| 파일 `/usr/local/sbin/monitoring.sh` | 실제 실행본. 홈 등의 작업본과 구분; root 소유 755가 현재 구성 |
| 파일 `/var/spool/cron/root` | Cronie의 root 사용자 예약 저장 위치. 직접 편집 대신 `sudo crontab -e` 사용 |
| 디렉터리 `/var/log/sudo`; 가상 파일 경로 `/proc` 계열 | sudo 집계 자료와 자원 통계의 기반. 출력 코드·필드 해석은 디펜스 참조 |

**조회 — 게스트:**

```bash
sudo crontab -l
systemctl is-active crond
stat -c '%U:%G %a %n' /usr/local/sbin/monitoring.sh
sha256sum /usr/local/sbin/monitoring.sh
sudo cat /usr/local/sbin/monitoring.sh
```

**필수 판단:** root cron의 PATH와 아래 주기·부팅 예약, 전체 요구 지표·wall 방송을 확인한다. 현재 문서 코드는 export 한 번 → 모든 항목을 변수에 수집 → 마지막 printf 그룹으로 방송하는 구조다. memory·disk는 MiB의 used/total 표시 문자열이고 실패 처리를 추가한 버전은 아니다. root cron 내부에 sudo를 다시 넣지 않는다. 코드·지표 해석은 [디펜스 monitoring](defense.md#9-monitoring-코드--매분--중단-재부팅--복구), 전체 코드는 [치트시트](cheatsheet.md#34-전체-출력을-wall-한-번으로-전달)에 둔다.

```cron
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

**시연 변경·원복 — 게스트:** 변경 전에 root crontab 원문과 위 경로·소유자·권한·SHA-256을 보관한다. 아래 모든 편집은 `sudo crontab -e`, 확인은 `sudo crontab -l`로 한다. 다른 작업과 PATH는 보존한다.

| 단계 | 바꿀 부분·확인 |
|---|---|
| 매분 시연 | 기존 `*/10` 주기 줄만 `* * * * * /bin/bash /usr/local/sbin/monitoring.sh`로 교체. 기존 10분 줄을 함께 남기지 않음. 분 경계를 두 번 관찰하고 콘솔·SSH에서 수신 및 값의 원자료를 대조 |
| 중단 | monitoring 주기 줄과 `@reboot` 줄 **둘 다** 줄 앞에 `#`를 넣음. 스크립트 수정·이동·chmod나 crond 전체 중지로 대신하지 않음. 이미 시작한 작업을 종료하는 명령은 아님 |
| 중단 후 재부팅 | 주석 조회 후 `sudo reboot`. 재로그인해 같은 `stat`·`sha256sum`으로 위치·소유자·권한·내용 유지 확인. `sudo journalctl -b`로 이번 부팅만 보고 `/monitoring.sh`로 검색. 호출 부재와 다음 분 경계의 상태를 함께 관찰 |
| 원복 | 원래 PATH와 위 10분·`@reboot` 두 줄을 활성화. 매분 줄·중복이 없어야 함. 다음 10분 경계의 실제 방송까지 확인 |

조용한 화면만으로 실행 중단을 증명하지 않는다. 부팅 때 수신 터미널이 없을 수 있다. cron의 CMD·END 기록은 호출·종료 근거이지 12개 지표의 성공이나 터미널 수신 증거는 아니다. 불필요한 로그 전체를 공유하지 말고 해당 작업·시각만 대조한다.

## 7. 제출·서명·VDI 보존

| 종류·핵심 경로 | 역할 |
|---|---|
| 호스트 파일 `~/goinfre/VirtualBox VMs/b2br/b2br.vdi` | 알려진 디스크 후보. 먼저 VirtualBox Storage에서 실제 연결된 파일인지 확인 |
| 실제 제출 저장소 루트 파일 `README.md`, `signature.txt` | 독립 설명 문서와 실제 가상 디스크 SHA-1. private 학습 저장소와 구분 |
| 학습 저장소 파일 `submission/README.md` | 실제 제출 루트로 복사할 문서 원본. VM·VDI는 Git 제출 대상이 아님 |

**필수 조회 — 호스트:** VM이 정상 종료된 **Powered Off**, 실제 연결 경로 확인, 평가 시작 시 스냅샷 없음 상태에서 **부팅 전에** 비교한다. 아래 후보 경로가 다르면 확인한 실제 경로로 바꾼다.

```bash
# 호스트: 실제 제출 저장소의 루트에서
sha1sum "$HOME/goinfre/VirtualBox VMs/b2br/b2br.vdi"
cat signature.txt
```

SHA-1 해시 부분끼리 비교하며 과거 값을 대신 쓰지 않는다. README의 지정 이탤릭 첫 줄·필수 절·설계·네 비교·AI 활용 설명은 [제출용 README](../submission/README.md)와 대조한다. 해시 조회는 signature.txt 생성·갱신이나 제출 완료를 뜻하지 않는다.

**보존·원복:** 평가 전 기준 사본·검증을 준비하고 상세 [VDI 보존 절차](targets.md#평가-사이의-디스크-보존과-복원)를 따른다. `cp -i`는 복사·덮어쓰기 변경, `cmp`·`sha1sum`은 내용 대조라는 차이를 기억한다. Running·Paused·Saved 상태나 차등 디스크가 남은 상태에서 단일 VDI를 복사·덮어쓰지 않는다. 전체 복제·UUID 변경은 원본 바이트 동일성을 보장하지 않는다.

원문은 **각 평가 시작 시 스냅샷 없음**과 **해당 평가용 스냅샷 생성·종료 후 삭제**를 구분한다. Delete만으로 변경 전 내용이 복원되는 것은 아니다. 평가용 상태 복원·삭제 또는 검증한 오프라인 기준 사본 복원 뒤에도 SHA-1을 다시 대조한다. hostname·방화벽·cron을 원복했다는 사실만으로 디스크 해시까지 돌아온 것은 아니다. 이 문서는 실제 디스크 복사·복원·제출을 수행한 기록이 아니다.
