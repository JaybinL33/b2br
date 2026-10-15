# Born2beRoot 디펜스 핵심 정리

평가 순서대로 **말로 설명 → 직접 확인 → 요청한 변경 → 복구**한다. 개념을 처음 배우려면 [개념 설명서](concepts.md), 자세한 명령은 [치트시트](cheatsheet.md), 장애 판단은 [targets](targets.md)를 읽는다. 이 문서는 필수 평가 대비이며 보너스 구축 안내가 아니다.

**출처 표기:** ‘평가지 p.N’은 사용자가 첨부한 `Projects Born2beroot En.pdf`의 PDF 1~5쪽이다. 전체 텍스트와 렌더링한 5쪽을 대조했다. 오른쪽 일부 문장이 실제로 잘려 있어 누락 부분을 복원하거나 인용하지 않는다. ‘과제 p.N’은 저장소 [원문 v5.2](../en.subject.pdf)의 인쇄 쪽 번호다. 평가 화면의 다른 사람 프로필·제출 주소는 옮기지 않았다.

**읽는 기준:** ‘평가 요구’는 첨부에서 확인한 항목, ‘대비 방법’은 이를 보여주기 위한 권장 명령이다. 명령 자체가 평가표의 지정 명령인 것은 아니다. 아래 코드는 VM에서 실행한 기록이 아니며, 한 블록씩 상황에 맞게 사용할 시연안이다. 현재 사실은 [학습 기록](learning-log.md)이 기준이다.

목적별 경로·최소 명령·기대값을 빠르게 찾으려면 [과제 필수 참조](essentials.md)를 사용한다. 아래 문서는 평가 순서와 설명·시연 근거를 맡는다.

## 먼저 확인할 준비 공백

| 항목 | 기록으로 확인된 범위 | 아직 준비·확인할 범위 |
|---|---|---|
| 계정 그룹 | 10월 13일 `id jungblee`에 `wheel,user42,sudo`가 모두 있음 | 새 로그인 셸의 `id`; 평가용 새 계정의 `evaluating`·`sudo` 가입 시연 |
| 관리자 권한 | `%wheel ALL=(ALL) ALL`과 sudo 사용 확인 | `sudo` 그룹 이름만으로 권한이 생긴다고 설명하지 않기; 새 계정의 실제 허용 정책 확인 |
| hostname | `check42`로 변경·SSH 재접속·`jungblee42` 복원 확인 | 변경한 이름으로 **재부팅 후** 유지 확인(평가지 p.3) |
| 방화벽 | 현재 4242 중심 구성 기록 | 8080 추가·목록·삭제 시연 결과는 미확인(p.4) |
| 새 사용자 SSH | 과거 `b2brtest` 접속 성공 후 계정 삭제 확인 | 평가 당일 새로 만든 계정으로 다시 시연(p.4) |
| monitoring | 30분 중단 및 복구 후 방송 수신 확인 | 매분 실행, 중단 상태의 재부팅, 모든 터미널 수신, 부팅 방송 수신(p.4 및 과제 p.10) |
| 제출·디스크 | 당시 스냅샷 없음 화면, Powered Off에서 SHA-1 계산 확인 | 실제 Storage 연결·기준 사본·복원 검증, 학교 저장소와 `signature.txt` 제출 상태 |
| 추가 권장 확인 | Rocky 10.2 조회, 일부 GUI 패키지 부재, 일부 품질 거절 확인 | 과제의 최신 안정 버전 조건 대조, GUI 전체 부재, 나머지 품질·root 적용 시험 |

과거 확인을 현재 VM의 실시간 상태로 단정하지 않는다. 아래 준비를 읽었다고 학습 로그의 미확인을 완료로 바꾸지 않는다.

## 1. 평가 시작 전: 저장소·디스크·README

**평가 요구 — p.1~2:** 본인 참석, 본인의 올바른 제출 저장소를 빈 폴더에 clone, 속이는 alias 없음, 보조 검사 스크립트 함께 검토. 루트 `signature.txt`와 실제 디스크의 서명 일치, 시작 시 스냅샷 없음, 미리 실행 중인 VM은 받지 않음. **부팅 전에** 대조한 뒤 평가용 VM을 시작한다.

**대비 방법 — 호스트:** VirtualBox Storage에서 실제 연결된 VDI를 먼저 식별한다. 알려진 후보는 `~/goinfre/VirtualBox VMs/b2br/b2br.vdi`지만 연결 확인을 대신하지 않는다. Powered Off 상태에서 실제 경로의 `sha1sum` 결과 중 **해시만** 제출 파일과 비교한다. 경로 문자열까지 비교하지 않는다. 과거 계산값을 현재 값 대신 쓰지 않는다.

```bash
# 호스트: 실제 제출 저장소의 루트, 확인한 디스크 경로에서만
sha1sum "$HOME/goinfre/VirtualBox VMs/b2br/b2br.vdi"
cat signature.txt
```

기준 디스크를 유지하는 평가용 cold snapshot 또는 다른 디렉터리의 디스크 복사본 사용이 평가지 p.2에 나온다. 과제 p.16~17도 시작 시 스냅샷 없음과 평가용 스냅샷 생성·종료 시 삭제를 구분한다. **‘모든 스냅샷은 언제나 금지’로 외우지 않는다.** 삭제 버튼만 누르면 원본 바이트가 복원된다는 뜻도 아니다. 실제 보존·대조·복원은 [디스크 보존 절차](targets.md#평가-사이의-디스크-보존과-복원)를 따른다. 복사한 VDI의 UUID를 임의 변경하면 바이트·해시도 달라질 수 있다.

**README 요구 — p.2, 과제 p.12~13:** 제출 저장소 루트에 README가 있어야 한다. 첫 줄은 아래 형식이며 이탤릭이다.

```markdown
*This project has been created as part of the 42 curriculum by jungblee.*
```

프로젝트·OS 선택과 장단점·주요 설계(파티션, 보안, 사용자, 서비스), 네 가지 비교(Debian/Rocky, AppArmor/SELinux, UFW/firewalld, VirtualBox/UTM)를 확인한다. 과제의 Description·Instructions·Resources와 AI 활용 설명도 함께 대조한다. [제출용 README](../submission/README.md)를 실제 제출 루트에 복사하는 구조이며 학습 저장소 루트 안내문으로 대신하지 않는다.

## 2. 프로젝트 개요: 짧게 말로 답하기

**평가 요구 — p.2:** VM 동작·용도, OS 선택, Rocky/Debian 차이, Rocky 선택 시 SELinux·DNF를 설명한다. 설명이 명확하지 않으면 중단한다고 명시한다. 평상시 monitoring은 10분마다 동작해야 한다.

| 예상 질문 | 말로 답할 핵심 |
|---|---|
| VM은 무엇이고 왜 쓰나? | 호스트 자원을 가상 CPU·메모리·디스크로 제공해 별도 게스트 커널과 OS를 실행한다. 격리된 실습·다른 OS 실행·환경 재현에 유용하지만 자원 비용과 호스트 장애 영향이 있다. VirtualBox가 그 실행을 중재한다. |
| Docker와 같은가? (권장 확장 질문) | VM은 독립 커널을 부팅하고, 일반적인 Linux 컨테이너는 실행 환경을 분리하되 커널을 공유한다. Docker는 이미지로 컨테이너를 배포·실행하는 도구다. [가상화 장](concepts/02-virtualization.md) |
| 왜 Rocky 10.2 Minimal인가? | RHEL 계열의 DNF·SELinux·firewalld를 함께 다루는 최소 서버 구성이다. GUI 없이 필요한 서비스만 두며 보안 정책과 서비스의 관계를 배운다. 패키지·정책을 함께 이해해야 하는 관리 비용이 있다. 버전 조회와 ‘최신 안정 버전’ 충족 검증은 별개다. |
| Debian과 Rocky 차이는? | Debian은 커뮤니티 범용 배포판과 deb/APT/dpkg 생태계, Rocky는 RHEL 호환 지향과 RPM/DNF 생태계다. 이 과제는 Debian에 UFW, Rocky에 firewalld를 요구한다. Debian 계열 AppArmor와 Rocky SELinux의 정책 모델도 비교한다. 어느 쪽이 무조건 안전하거나 빠르다는 비교는 하지 않는다. |
| DNF는 무엇인가? | 저장소에서 패키지를 찾아 의존성을 해결하고 설치·갱신을 조정한다. RPM은 패키지 형식이자 하위 관리 도구다. [패키지 장](concepts/13-packages.md) |
| SELinux는 무엇인가? | 소유자·rwx 권한 외에 프로세스와 파일·포트의 보안 문맥 및 정책으로 접근을 제한한다. Enforcing은 거절을 적용하고 Permissive는 기록하되 차단하지 않는다. sshd 포트·firewalld 허용·SELinux 포트 타입은 별도 조건이다. |
| 다른 필수 비교는? | AppArmor는 경로 중심 프로파일, SELinux는 레이블·타입 중심 정책. UFW는 간단한 규칙 인터페이스, firewalld는 zone·서비스 및 runtime/permanent 관리. VirtualBox는 이 호스트의 가상화 도구, UTM은 Mac의 가상화·에뮬레이션 도구이며 과제의 사용 허용 조건이 있다. [제출용 비교표](../submission/README.md#운영체제-선택과-비교) |

## 3. 기본 부팅과 로그인

**평가 요구 — p.2~3:** GUI 없는 시작, 암호 입력과 일반 사용자 로그인, 허용 OS·방화벽·SSH 동작 확인. p.3의 Simple setup에는 UFW라고 쓰였지만 p.4와 **과제 p.8은 Rocky에 firewalld**를 명시하므로 UFW를 추가 설치하는 의미가 아니다.

**대비 방법 — VM 콘솔/일반 사용자:** LUKS 해제 암호와 계정 로그인 암호의 역할을 구분하고 root로 일상 로그인하지 않는다.

```bash
whoami
id
cat /etc/os-release
systemctl get-default
systemctl is-active sshd firewalld crond
getenforce
```

기대값은 `jungblee`, Rocky 10.2, 서버용 `multi-user.target`, 각 서비스 `active`, `Enforcing`이다. 화면·설치 패키지·실행 서비스를 함께 확인한다. target 하나나 패키지 몇 개의 부재만으로 GUI 전체 부재를 증명하지 않는다. LUKS는 꺼진 디스크의 내용을 보호하고 로그인 인증은 열린 시스템의 사용자를 확인한다.

## 4. 사용자·그룹·비밀번호 정책

**평가 요구 — p.3:** 본인 로그인 계정의 `sudo`·`user42` 가입, 새 계정 생성·정책에 맞는 암호 지정, `evaluating` 그룹 생성과 배정, 정책 설정과 장단점 설명.

```bash
id jungblee
getent group sudo user42 evaluating
sudo authselect current
sudo authselect check
cat /etc/pam.d/passwd
cat /etc/pam.d/system-auth
cat /etc/security/pwquality.conf.d/90-local.conf
chage -l jungblee
sudo chage -l root
```

**수치 근거 — 과제 p.8~9:** 다음 값은 잘린 평가 문장을 복원한 것이 아니라 과제 본문과 기존 설정을 대조한 것이다.

**설명:** PAM의 password 스택에서 pam_pwquality가 새 암호를 검사하고 pam_unix가 저장한다. 본체·drop-in·PAM 인수의 실제 우선순위를 함께 읽는다. 기간은 최소 2일·최대 30일·7일 경고, 품질은 길이 10 이상·대문자/소문자/숫자·동일 문자 연속 최대 3·사용자명 배제·이전 암호와 차이 7이다. root도 품질을 적용하되 이전 암호 비교는 과제의 예외다. `enforce_for_root`와 `difok`의 경로 차이는 [PAM 설명](targets.md#3-비밀번호pam)을 따른다.

**장단점:** 짧고 추측하기 쉬운 암호와 재사용을 줄이고 유출된 암호의 장기 사용을 제한하려는 정책이다. 반면 잦은 변경은 예측 가능한 변형·사용 부담을 낳을 수 있고, 최소 2일은 즉시 재변경을 막는다. 만료 정책만으로 도난 계정·피싱을 해결하지 못한다. ‘과제라서’만으로 설명하지 않는다.

**변경 시연안:** `defenseuser`는 예시다. 평가자와 정한 새 이름으로 바꾸고 먼저 `getent passwd defenseuser`로 기존 계정이 아닌지 확인한다. 기존 `evaluating`이 있으면 다시 만들거나 삭제하지 않는다.

```bash
sudo useradd -m defenseuser
sudo passwd defenseuser
sudo chage -l defenseuser
# evaluating 그룹이 없을 때만
sudo groupadd evaluating
sudo usermod -aG evaluating defenseuser
id defenseuser
```

새 계정의 기간이 2/30/7인지 기본값 적용 결과를 확인한다. 다르면 생성 기본값부터 조사한다. 본인 계정의 `sudo,user42` 요구는 과제 p.8에도 명시되어 **wheel로 대체했다고 설명하면 안 된다.** 기존 가입 기록은 확인되어 있지만 새 계정과 새 세션은 별개다. `-aG`의 `-a`는 기존 보조 그룹을 유지한다. root가 새 계정 암호를 설정하는 시험만으로 이전 암호 유사성 거절을 검증할 수는 없다.

## 5. hostname 변경·재부팅과 파티션

**평가 요구 — p.3:** `login42` 형식, 평가자의 로그인에 맞는 이름으로 변경 후 **재부팅**, 유지 확인 후 원래 이름으로 복원. 파티션을 보여주고 LVM을 설명한다.

```bash
hostname
cat /etc/hostname
# reviewer42를 평가자의 실제 로그인+42로 바꾼다
sudo hostnamectl set-hostname reviewer42
sudo reboot
```

재부팅은 SSH를 끊는다. 콘솔에서 디스크 암호 입력 후 다시 접속해 `hostname`·`cat /etc/hostname`이 새 이름인지 확인한다. 프롬프트만 보지 않는다. 복원은 `sudo hostnamectl set-hostname jungblee42` 후 두 조회로 확인한다. 복원 후 재부팅 유지 확인도 권장하지만 평가표의 변경 후 재부팅과 구분한다.

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo pvs
sudo vgs
sudo lvs
findmnt /
findmnt /home
findmnt /var/log
```

각 마운트 지점과 해당 LV·ext4의 연결을 대조한다.

**설명:** BIOS Boot 영역·비암호화 `/boot`, 암호화 LUKS 안에 PV→VG→LV, 각 LV의 ext4 또는 swap이 이어진다. `/`, `/home`, `/var/log` 분리는 OS·사용자 데이터·로그의 용량 영향을 나누고 VG 여유는 나중 확장에 둔다. LV 확대와 파일시스템 확대는 별도이며 LVM은 암호화나 백업 자체가 아니다. [저장장치 장](concepts/07-storage.md). 필수 구조를 설명하며 보너스 그림을 그대로 만들었다고 주장하지 않는다.

## 6. sudo 정책과 로그

**평가 요구 — p.3~4:** sudo 설치, 새 사용자를 `sudo` 그룹에 배정, sudo의 용도·예시와 엄격한 규칙 설명, `/var/log/sudo/`의 존재·내용·새 명령 후 기록 확인.

```bash
rpm -q sudo
sudo usermod -aG sudo defenseuser
id defenseuser
sudo visudo -c
sudo -l
sudo cat /etc/sudoers.d/90-b2br
sudo ls -la /var/log/sudo
sudo sudoreplay -d /var/log/sudo -l
```

**정책 수치 근거 — 과제 p.9.**

**설명:** sudo는 정책에 따라 다른 사용자 권한으로 명령을 실행한다. 3회 인증 제한·사용자 정의 실패 문구·TTY 요구·secure_path·입출력 로그를 각각 보여준다. secure_path는 명령 검색 경로이며 명령 허용 목록이 아니다. 현재 관리 권한은 `%wheel` 규칙에서 온다. 새 계정을 `sudo` 그룹에 넣어도 그 그룹을 허용하는 규칙이 없다면 sudo 권한은 생기지 않는다. 평가에서 새 계정의 관리자 실행도 보여줄 경우 정책 확인 후 해당 **일회성 계정만** wheel에 추가하는 방법을 설명하고 원래 관리자 세션을 유지한다. 그 추가를 이미 수행했다고 기록하지 않는다.

비밀 없는 `sudo cat`에 시험 문장을 입력하고 Ctrl+D로 끝낸 뒤 `sudoreplay -l`에서 해당 명령의 TSID를 찾아 재생한다. `sudo sudoreplay -d /var/log/sudo ID`는 출력, `-f stdin,ttyin ID`는 입력을 확인한다. 조회 자체도 sudo 기록을 늘릴 수 있다. 문법 검사·명시적 TTY 거절·입출력 확인의 상세 절차는 [치트시트](cheatsheet.md#9-로그-생성과-출력-재생)를 따른다.

## 7. firewalld: 4242 확인 → 8080 추가 → 삭제

**평가 요구 — p.4:** 설치·동작·역할 설명, 활성 규칙에 4242, 8080 허용 규칙 추가 후 목록 확인, 새 규칙 삭제.

```bash
rpm -q firewalld
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
# ZONE은 실제 게스트 NIC가 속한 zone 이름으로 바꾼다
sudo firewall-cmd --zone=ZONE --list-all
sudo firewall-cmd --permanent --zone=ZONE --list-all
```

**대비 방법:** 일회성 시연은 runtime 규칙으로 충분하다. 평가표가 영구 8080 허용을 요구한다고 쓰지 않는다. 먼저 8080이 원래 없음을 확인하고 다음을 개별 실행한다.

```bash
sudo firewall-cmd --zone=ZONE --add-port=8080/tcp
sudo firewall-cmd --zone=ZONE --list-ports
sudo firewall-cmd --zone=ZONE --remove-port=8080/tcp
sudo firewall-cmd --zone=ZONE --list-all
```

추가 시 `4242/tcp 8080/tcp`, 삭제 후 원래 규칙을 기대한다. 기존 규칙이 있으면 임의로 삭제하지 않는다. permanent를 바꾼 시연이라면 그쪽도 원복하고 적용 결과를 대조한다. 포트를 허용해도 그 포트의 서버가 생기지 않는다. 평가용 8080 서버 설치·NAT 포워딩 추가는 이 항목의 요구가 아니다. [runtime/permanent 근거](https://firewalld.org/documentation/man-pages/firewall-cmd.html).

## 8. SSH: 새 사용자 접속과 root 거절

**평가 요구 — p.4:** 설치·동작·역할, 게스트 SSH가 4242만 사용, 새 사용자 원격 로그인, root SSH 불가.

```bash
rpm -q openssh-server
systemctl is-active sshd
sudo sshd -t
sudo sshd -T | grep -E '^(port|permitrootlogin) '
sudo ss -lntp
```

4242 리스닝과 `permitrootlogin no`를 기대한다. Match 조건이 있으면 실제 사용자·주소의 적용값도 조사한다. 설정만 보거나 모든 접속 실패를 root 거절 증거로 취급하지 않는다.

```bash
# 호스트의 새 터미널
ssh -p 4242 defenseuser@127.42.42.42
# 접속한 새 사용자 셸
whoami
id
exit
# 호스트: root는 로그인되어서는 안 된다
ssh -p 4242 root@127.42.42.42
```

**설명:** SSH는 암호화된 원격 채널과 서버·사용자 인증을 제공한다. 호스트 127.42.42.42:4242 NAT 전달 → 게스트 방화벽 → sshd → 계정 인증 순서다. 게스트 자기 루프백과 호스트 주소는 다르다. 키 확인이 나오면 콘솔의 호스트 키 지문과 대조한다. 공개키 또는 비밀번호 인증은 평가지가 허용하지만 현재 실습은 비밀번호 인증이다. [SSH 개념](concepts/12-security.md#ssh는-서버-확인과-사용자-로그인을-연결한다).

## 9. monitoring: 코드 → 매분 → 중단 재부팅 → 복구

**평가 요구 — p.4:** 코드 동작·cron·10분 실행 설정 설명, 정상 동작을 확인한 후 매분 실행과 동적 값 확인, 스크립트 자체를 바꾸지 않고 시작 후 실행 중단, 재부팅 후 같은 위치·같은 권한·내용 유지 확인.

```bash
sudo cat /usr/local/sbin/monitoring.sh
sudo crontab -l
systemctl is-active crond
stat -c '%U:%G %a %n' /usr/local/sbin/monitoring.sh
sha256sum /usr/local/sbin/monitoring.sh
```

**지표 목록 근거 — 과제 p.10, 아래 계산 방식은 현재 치트시트 구현.**

**설명:** root cron이 Bash 스크립트를 실행하고 `{ ...; } | wall`로 한 번 방송한다. root cron 안에 sudo를 또 넣지 않는다. cron은 달력 기준이며 `*/10`은 매시 0·10·20…분, 부팅 즉시 실행은 별도의 `@reboot`다. 다음 자료가 어떤 출력이 되는지 코드를 짚어 설명한다.

| 출력 | 이 구현의 원자료·해석 |
|---|---|
| Architecture / CPU physical / vCPU | `uname -a`; `lscpu -p`의 고유 온라인 소켓 수·온라인 CPU 행 수. 호스트 물리 CPU를 직접 센 값은 아님 |
| Memory / Disk | `free -m`, `df -m --total`의 used/total. 실제 MiB·GiB와 표기 차이, tmpfs/devtmpfs 제외 설명 |
| CPU load / Last boot | `vmstat 1 2` 마지막 표본의 `100-id`(직전 1초, 순수 계산 시간만은 아님), `uptime -s` |
| LVM / TCP | `lsblk` TYPE에 lvm 존재; `ss`의 ESTABLISHED TCP 소켓 행 수 |
| User log / Network | `who`의 고유 사용자명 수; enp0s3의 IPv4·MAC. 세션 수와 사용자 수 구별 |
| Sudo | `/var/log/sudo`에 보관된 `sudoreplay -l` 세션 수. 보존 범위가 있으며 시스템 수명 전체 명령 횟수와 같지 않을 수 있음 |

**변경과 검증 순서:** 먼저 crontab 원문·스크립트 해시·소유자/권한/경로를 보관한다. `sudo crontab -e`에서 **기존 주기 줄만** `* * * * * /bin/bash /usr/local/sbin/monitoring.sh`로 바꾼다. PATH와 다른 예약은 보존하고 10분 줄을 동시에 남기지 않는다. 1분 경계를 두 번 관찰하고 콘솔·SSH 각각의 수신을 확인한다. 다른 사용자 SSH 세션의 접속/종료나 비밀 없는 sudo 명령으로 TCP·사용자·sudo 수의 변화를 원자료와 대조한다. CPU·메모리가 반드시 증가해야 한다고 가정하지 않는다.

중단은 같은 편집기에서 monitoring의 주기 줄과 `@reboot` 줄을 **모두 주석 처리**한다. crond 전체 중지, 스크립트 이동·수정·chmod로 우회하지 않는다. `sudo crontab -l` 확인 후 `sudo reboot`한다. 재로그인 후 같은 `stat`·`sha256sum`과 경로로 불변을 확인하고, 현재 부팅의 작업 기록과 분 경계 관찰을 함께 본다.

```bash
sudo journalctl -b -t CROND --no-pager
```

스크립트 호출이 없어야 하며, 조용한 화면만으로 실행 중단을 증명하지 않는다. 아직 로그인한 사람이 없는 `@reboot` 방송은 실행돼도 보이지 않을 수 있다. 필요하면 [유지 관리 진단](operations/maintenance.md)을 따른다.

**복구:** 원래 PATH 및 아래 두 줄을 활성화하고 중복·매분 줄이 남지 않았는지 조회한다. 다음 10분 경계의 실제 방송까지 확인한다.

```cron
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

cron 문법 근거는 [crontab(5)](https://man7.org/linux/man-pages/man5/crontab.5.html), 원래 전체 코드는 [치트시트](cheatsheet.md#34-전체-출력을-wall-한-번으로-전달)에 있다.

## 10. 종료·복구와 중단 조건

**권장 복구 체크:** hostname `jungblee42`, 8080 제거와 원래 4242 정책, monitoring 10분+부팅 예약·방송, 스크립트 경로·소유자·권한·내용을 대조한다. 시험 계정은 시연이 끝난 뒤 로그아웃·남은 프로세스·필요 데이터부터 확인하고 이번에 만든 계정만 삭제한다. `evaluating`도 이번에 만들었고 다른 용도가 없을 때만 삭제한다. 실제 평가 요구가 끝나기 전에 정리하지 않는다. 기준 디스크 보존·복원은 별도 절차이며 설정을 되돌렸다는 사실만으로 기존 SHA-1이 복구되지 않는다. 평가 전 실제 그룹·보안 설정을 새로 고쳐 제출 기준으로 삼는다면, 수정 완료 후 종료된 최종 디스크의 백업과 SHA-1·signature.txt를 같은 시점으로 다시 맞춰야 한다. 이 문서 작성은 그 변경이나 제출 갱신을 수행한 것이 아니다.

| 구분 | 평가지에 명시된 조건과 대응 |
|---|---|
| 진행 전제 / 일반 플래그 (p.1~2) | 당사자 참석과 올바른 제출물 필요. 빈 저장소·잘못된 제출 등은 종료 대상. 일반 안내에는 0점, 부정행위는 -42가 명시되어 있고 의심 시 Cheat 플래그와 중단을 신중하게 판단하도록 한다. |
| 시작 단계 중단 (p.2) | 실행 중인 VM은 수락하지 않음. 서명 불일치 또는 예상대로 동작하지 않으면 중단. 시작 시 스냅샷 없음과 평가용 보존 절차를 먼저 확인한다. |
| README 중단 (p.2) | 지정 첫 줄·필수 내용·네 비교 중 누락이 있으면 즉시 종료한다고 명시한다. |
| 필수 기능·설명 중단 (p.2~4) | 각 단계의 동작 실패나 설명 불명확에 중단 문구가 반복된다. 특히 hostname 재부팅 유지, 정책·그룹, sudo 로그, 방화벽 변경, SSH, monitoring 변경·중단을 설명과 함께 준비한다. |
| 보너스 (p.5) | 필수 부분이 완전해야 평가한다. 이번 목표가 아니므로 서비스·포트를 추가하지 않는다. 보너스 미구현을 필수 실격과 혼동하지 않는다. |

평가지의 Ratings에는 설명 불가·미완료 등 플래그도 있다. 잘린 문장의 세부 조건이나 모든 실패의 점수를 추정하지 않는다. 평가 당일 표시되는 완전한 평가표·과제와 차이가 있으면 평가자와 원문을 먼저 대조한다.
