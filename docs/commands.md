# 명령어 중심 참조: 이름·인수·옵션·출력 읽기

`systemctl`을 보았을 때 무엇을 관리하는 도구인지, 왜 `is-active`와 `enable`이 다른지 찾는 문서다. **명령 이름을 기준으로 설명**하며 여러 명령을 실행하는 순서는 [목적별 필수 참조](essentials.md), 신규 구축은 [스피드런](speedrun.md), 평가 시연·원복은 [디펜스](defense.md)에 둔다. 경로의 역할은 [디렉터리 지도](directories.md), 원리는 [개념 설명서](concepts.md)를 읽는다.

기존 치트시트·monitoring 코드·디펜스·디렉터리 관찰 예시에서 사용한 명령과 혼동을 풀기 위한 보조 명령을 골랐다. **핵심**은 과제 관리·평가에 직접 쓰는 도구, **수집·가공**은 monitoring의 자료를 만들거나 해석하는 도구, **보조**는 비교·탐색에 필요한 도구다. 모든 명령의 설치·사용이 과제의 개별 의무라는 뜻은 아니다.

각 항목은 **조회 / 변경 / 혼합 / 출력·통신**으로 영향을 표시한다. 별도 표시가 없으면 게스트의 일반 셸을 기준으로 읽고, `sudo` 예시는 관리자 권한이 필요하다. SSH 접속과 VDI 해시는 호스트에서 실행하는 예를 명시했다. 조회도 로그·접근 시각을 남길 수 있으므로 제출 디스크의 바이트 불변을 보장하지 않는다. **이 문서 작성에서는 실제 VM 명령을 실행하지 않았다.**

공식 도구 매뉴얼을 기준으로 필요한 옵션만 정리했다. GNU·procps-ng·util-linux·iproute2·shadow 등 Linux 구현을 기준으로 하며 macOS/BSD의 동명 명령과 옵션이 다를 수 있다. 웹 매뉴얼은 Rocky 10.2의 설치 버전과 같다고 가정하지 않는다. 해당 VM의 `man`, `--help`, 버전별 문서가 실제 지원 여부의 기준이다. systemd는 v257 문서를 비교 기준으로 사용하며 VM 버전을 새로 확인했다는 뜻은 아니다.

## 명령군 지도

| 알고 싶은 것 | 명령군 | 관련도 |
|---|---|---|
| 지금 실행 중인가, 다음 부팅에도 시작하는가 | [systemctl·journalctl·hostnamectl](#서비스로그호스트명) | 핵심 |
| 누가 로그인하고 어떤 그룹·암호 정책을 쓰는가 | [계정·그룹·PAM](#계정그룹인증) | 핵심 |
| 누가 관리 명령을 실행하고 어디에 기록하는가 | [sudo·visudo·sudoreplay·su](#관리-권한과-기록) | 핵심, su는 비교 |
| 어디로 접속하며 무엇이 통신을 허용하는가 | [SSH·소켓·방화벽·SELinux](#접속과-보안) | 핵심 |
| 디스크·마운트·파일 사용량은 어떻게 다른가 | [블록 장치·LVM·용량](#저장장치와-파일시스템) | 핵심, du는 비교 |
| monitoring의 숫자는 어디서 오는가 | [CPU·메모리·네트워크·사용자](#자원과-세션-수집) | 수집·가공 |
| 언제 실행하고 누구에게 보여 주는가 | [crontab·wall](#예약과-방송) | 핵심 |
| 파일을 설치하고 권한·서명을 확인하는가 | [파일·패키지 관리](#파일설치권한서명패키지) | 핵심·보조 |
| 텍스트에서 어떤 행·열을 고르는가 | [자료 처리·읽기](#텍스트-처리와-탐색) | 수집·가공·보조 |

[명령 이름 색인](#명령-이름-색인)에서 개별 명령으로 바로 이동할 수 있다.

## 명령줄을 읽는 기준

`sudo systemctl enable --now crond`에서 `sudo`는 권한 전환 실행 도구, `systemctl`은 실행할 명령, `enable`은 하위명령, `--now`는 동작을 보충하는 옵션, `crond`는 대상 unit 이름이다. 옵션이 받는 값과 실제 대상 인수를 구분한다. 예를 들어 `ssh -p 4242 user@host`에서 4242는 포트 옵션의 값이고 user@host는 접속 대상이다.

문법 표기의 `[옵션]`은 생략 가능, `파일...`은 여러 개 가능, `사용자`·`경로` 같은 말은 실제 값으로 바꾸라는 뜻이다. 대괄호와 말 자체를 그대로 입력하지 않는다. 짧은 옵션의 뜻은 도구마다 다르다. `ssh -p`는 포트지만 `ss -p`는 프로세스 정보, `sudo -l`은 허용 목록이지만 `ls -l`은 긴 목록이다. `--`는 많은 도구에서 옵션 해석의 끝을 표시한다.

### 셸 문법과 명령은 다르다

| 표현 | 셸이 하는 일 | b2br에서 읽는 관점 |
|---|---|---|
| `A \| B` | A의 표준출력을 B의 표준입력으로 연결 | `ss ... \| wc -l`은 소켓 조회 결과의 줄을 셈. `\|` 자체는 명령이 아님. `sudo A \| B`에서도 B까지 자동으로 root가 되지 않음 |
| `> 파일`, `>> 파일`, `2> 파일` | 출력 덮어쓰기·추가·오류 출력 연결 | `>`는 파일을 잘라낼 수 있음. `sudo 명령 > 파일`의 파일 열기는 보통 현재 셸 권한 |
| `$(명령)` | 명령 출력을 가져와 다른 문자열·인수를 구성 | `$(uptime -s)`는 부팅 시각 출력으로 치환. 파이프와 달리 셸이 결과를 받아 사용 |
| `"$변수"` | 변수 값을 확장하되 공백·패턴 때문에 인자가 갈라지는 것을 막음 | VDI 경로의 공백을 하나의 인자로 유지 |
| `LC_ALL=C 명령` | 그 실행의 로케일을 지정 | `Mem:`·`total` 등 출력 해석을 일정하게 함. 전체 시스템 언어 변경이 아님 |
| `{ ...; } \| wall` | 묶은 명령의 표준출력을 한 통로로 연결 | monitoring 여러 줄을 wall 한 번으로 전달 |

`cd`, `exit`, `command`, `type`은 Bash 내장 명령이다. `printf`도 Bash 내장과 외부 실행 파일이 모두 있을 수 있다. `type printf`는 현재 셸이 무엇을 실행하는지, `command -v ssh`는 명령 탐색 결과를 보여 준다. `cd`는 현재 셸의 작업 디렉터리를 바꾸므로 외부 프로그램의 작업과 구분한다. `exit`는 현재 셸을 끝내며 SSH 안에서는 보통 호스트로 돌아간다. `sudo exit`로 원격 셸을 끝내는 것이 아니다. 셸의 전체 흐름은 [셸 장](concepts/04-shell.md), 문법 근거는 [Bash 매뉴얼 원문](https://man7.org/linux/man-pages/man1/bash.1.html)에 있다.

### 닮은 명령을 구분한다

| 비교 | 다른 질문 |
|---|---|
| systemctl / journalctl | 서비스 상태·관리 / 발생한 사건의 로그 |
| sudo / su | 정책에 따라 한 명령을 다른 자격으로 실행 / 다른 사용자 자격의 셸·명령으로 전환 |
| chmod / chown | 읽기·쓰기·실행 모드 / 소유 사용자·그룹 |
| lsblk / df / du | 블록 장치 연결 구조 / 마운트된 파일시스템 사용량 / 경로 아래 파일의 공간 사용 |
| grep / awk | 조건에 맞는 행 선택 / 행의 필드 선택·계산·출력 구성 |
| sort -u / uniq | 정렬하며 중복 제거 / 서로 붙어 있는 중복 행만 제거 |
| sha1sum / sha256sum / cmp | 과제 디스크 SHA-1 / 스크립트 내용 비교용 SHA-256 / 두 파일의 바이트 직접 비교 |

## 서비스·로그·호스트명

### systemctl

**핵심 · 혼합.** `systemctl [옵션] 하위명령 [unit...]`은 systemd 관리 대상에 질문하거나 상태를 바꾼다. b2br에서 `sshd`, `firewalld`, `crond`는 보통 `.service`를 생략한 이름이다. unit 정의는 `/usr/lib/systemd/system`, 로컬 변경은 `/etc/systemd/system` 계열에서 만난다.

| 하위명령·옵션 | 질문·효과와 읽는 법 |
|---|---|
| `status sshd` | 현재 상태와 최근 로그를 사람이 읽게 보여 줌. 장애 조사 시작점이며 전체 로그 조회는 아님 |
| `is-active sshd` | 지금 active인지 조회. 출력과 종료 상태로 판단하며 여러 unit을 주면 종료 상태 하나만으로 모두 성공했다고 보지 않음 |
| `is-enabled sshd` | 시작 경로에 연결됐는지 조회. enabled와 현재 active는 독립이며 static 같은 다른 상태도 존재 |
| `start` / `stop` | 지금 시작·정지. 다음 부팅의 활성화 설정을 자동 변경하지 않음 |
| `enable` / `disable` | 부팅 등의 시작 연결 생성·제거. 기본적으로 현재 프로세스를 시작·정지하지 않음 |
| `enable --now crond` | 활성화와 지금 시작을 함께 수행하는 **변경** |
| `restart` / `reload` | 다시 시작 / 서비스가 지원하는 설정 재읽기. 연결 영향·지원 여부가 다름 |
| `daemon-reload` | systemd가 unit 정의를 다시 읽음. sshd_config 재읽기 명령과 다름 |
| `get-default` | 기본 target 조회. multi-user.target만으로 그래픽 패키지 전체 부재를 증명하지 않음 |
| `show -p FragmentPath -p DropInPaths sshd` | 선택한 속성으로 unit 공급 파일·추가 설정 경로 확인 |

b2br의 `systemctl is-active sshd firewalld crond`는 세 서비스의 현재 동작을 묻는다. 시작 실패를 확인하지 않고 enable을 반복하는 것으로 설정 오류를 해결할 수 없다. [systemctl 공식 매뉴얼](https://github.com/systemd/systemd/blob/v257/man/systemctl.xml).

### journalctl

**핵심 · 조회.** `journalctl [필터] [출력옵션]`은 systemd journal을 읽는다. `-b`는 이번 부팅, `-u crond`는 unit, `-t CROND`는 syslog 식별자, `-n 20`은 최근 20개, `--since '시간'`은 시작 시각, `--no-pager`는 pager 없이 출력한다. 필터를 함께 쓰면 범위를 좁힌다.

`sudo journalctl -b -t CROND --no-pager`는 이번 부팅의 cron 작업 기록을 본다. 데몬 시작 로그와 작업 실행 로그는 식별 방식이 다를 수 있다. CMD 기록은 호출 근거이지 모든 지표 수집·wall 수신 성공 증거는 아니다. `/run/log/journal`과 `/var/log/journal` 보존 차이도 있다. `--vacuum-*`는 삭제 기능이므로 조회 옵션과 혼동하지 않는다. [journalctl](https://github.com/systemd/systemd/blob/v257/man/journalctl.xml).

### hostnamectl

**핵심 · 혼합.** `hostnamectl`은 호스트명·시스템 정보를 조회하고, `hostnamectl set-hostname 이름`은 이름을 바꾼다. `sudo hostnamectl set-hostname reviewer42`에서 reviewer42는 설정값이다. `/etc/hostname`의 지속 이름과 실제 표시를 대조한다. 평가에서는 변경 후 재부팅 유지와 `jungblee42` 복원이 연결된다. 접속 중 셸의 프롬프트가 즉시 바뀌지 않아도 설정 실패로 단정하지 않는다. [hostnamectl](https://github.com/systemd/systemd/blob/v257/man/hostnamectl.xml).

### hostname

**핵심 · 혼합.** `hostname`은 현재 호스트명을 조회한다. 인수를 주면 이름 변경 동작이 될 수 있어 b2br에서는 조회 형태만 사용하고 지속 변경은 hostnamectl로 한다. `cat /etc/hostname`은 저장된 설정을 읽는 별도 확인이다. [hostname(1)](https://man7.org/linux/man-pages/man1/hostname.1.html).

### reboot

**핵심 · 변경.** `sudo reboot`는 시스템 재부팅을 요청한다. SSH가 끊기고 암호화 디스크 해제를 위해 VM 콘솔이 필요할 수 있다. hostname 변경 유지·monitoring 중단 후 재부팅 시연에 쓰며, 문법 확인용으로 실행하지 않는다. 강제 옵션은 이 과제 참조에 필요 없다. [reboot(8)](https://man7.org/linux/man-pages/man8/reboot.8.html).

## 계정·그룹·인증

### id

**핵심 · 조회.** `id [사용자]`는 UID·GID·그룹을 보여 준다. 인수 없는 `id`는 현재 실행 자격, `id jungblee`는 이름으로 조회한 계정 정보다. `-u`는 UID만, `-G`는 그룹 ID들, `-n`을 함께 쓰면 이름으로 출력한다. 그룹 DB를 바꾼 뒤 기존 셸의 그룹 자격은 바로 바뀌지 않을 수 있으므로 새 로그인도 구분한다. b2br은 sudo·user42 가입과 wheel 권한 연결을 따로 설명한다. [id(1)](https://man7.org/linux/man-pages/man1/id.1.html).

### whoami

**핵심 · 조회.** `whoami`는 현재 유효 사용자 ID에 해당하는 이름을 출력한다. SSH 뒤 defenseuser인지 확인하거나 권한 실행 문맥을 구분할 때 쓴다. 접속 출발지·로그인한 모든 사용자·그룹 목록은 보여 주지 않으므로 who·id와 다른 질문이다. [whoami(1)](https://man7.org/linux/man-pages/man1/whoami.1.html).

### getent

**핵심 · 조회.** `getent 데이터베이스 [키...]`는 NSS를 통해 계정·그룹 등을 찾는다. `getent passwd defenseuser`는 계정 존재, `getent group evaluating`은 그룹 존재를 확인한다. 키를 생략하면 전체 열거가 될 수 있으므로 필요한 이름을 지정한다. 결과 없음과 조회 오류를 구별한 뒤 새 계정을 만든다. passwd DB의 필드는 UID·GID·홈·셸 등이며 비밀번호 평문이 아니다. shadow 조회·공유는 이 절차에 포함하지 않는다. [getent(1)](https://man7.org/linux/man-pages/man1/getent.1.html).

### useradd

**핵심 · 변경.** `useradd [옵션] 새이름`은 계정을 생성한다. `-m`은 홈 생성, `-d 경로`는 홈 경로, `-s 셸경로`는 로그인 셸, `-G 그룹목록`은 보조 그룹 지정이다. 이름·UID·홈 충돌을 먼저 확인한다. b2br의 `sudo useradd -m defenseuser` 다음에는 passwd로 비밀번호를 정하고 chage로 기본 기간을 확인한다. `/etc/login.defs`, `/etc/default/useradd`, `/etc/skel`의 역할은 같지 않다. [useradd(8)](https://man7.org/linux/man-pages/man8/useradd.8.html).

### usermod

**핵심 · 변경.** `usermod [옵션] 기존사용자`는 계정 속성을 바꾼다. `-G 그룹1,그룹2`는 보조 그룹 목록을 지정하며, **`-aG`는 기존 목록에 추가**한다. `-g`는 기본 그룹으로 다른 대상이다. `sudo usermod -aG user42,sudo jungblee`는 기본 그룹이나 비밀번호를 바꾸는 명령이 아니다. `-a`를 빠뜨리면 기존 wheel 등 보조 그룹을 잃을 수 있다. id와 새 로그인 자격을 대조한다. [usermod(8)](https://man7.org/linux/man-pages/man8/usermod.8.html).

### groupadd

**핵심 · 변경.** `groupadd [옵션] 새그룹`은 그룹을 생성한다. `-g 숫자`는 GID 지정이지만 이 과제에서는 자동 배정이면 충분하다. `sudo groupadd evaluating`은 그룹만 만들고 사용자를 가입시키지 않는다. 기존 그룹이면 재생성하지 않고 필요한 멤버십만 확인한다. `/etc/group` 직접 편집 대신 도구를 쓴다. [groupadd(8)](https://man7.org/linux/man-pages/man8/groupadd.8.html).

### passwd

**핵심 · 변경 중심.** `passwd [사용자]`는 비밀번호를 바꾼다. 일반 사용자의 인수 없는 호출은 자기 비밀번호, `sudo passwd root`는 관리자에 의한 root 비밀번호 설정이다. `-S`는 상태 조회, `-l`은 비밀번호 잠금이므로 단순 조회와 구분한다. 잠금이 모든 인증 수단의 차단을 뜻하지는 않는다.

입력한 새 비밀번호는 `/etc/pam.d/passwd`의 PAM 흐름에서 품질 검사·저장으로 이어진다. root가 다른 계정 암호를 정하는 경로는 이전 비밀번호를 받지 않으므로 이전·새 암호 비교 시험과 다르다. 암호는 대화형 프롬프트에 입력하고 명령 인자나 예시 문자열로 보관하지 않는다. [passwd(1)](https://man7.org/linux/man-pages/man1/passwd.1.html).

### chage

**핵심 · 혼합.** `chage -l 사용자`는 계정의 기간 정보를 조회한다. 변경 옵션 `-m`은 최소 일수, `-M`은 최대 일수, `-W`는 경고 일수다. `sudo chage -M 30 -m 2 -W 7 jungblee`는 기존 계정의 기간을 바꾸며 비밀번호 품질 규칙이나 현재 암호를 교체하지 않는다. `/etc/login.defs`의 새 계정 기본값과 계정별 실제 값은 별개다. 출력의 마지막 변경일·비밀번호 만료·계정 만료도 구분한다. [chage(1)](https://man7.org/linux/man-pages/man1/chage.1.html).

### userdel

**핵심 · 삭제.** `userdel [옵션] 사용자`는 계정을 삭제한다. `-r`은 홈·메일 스풀 등도 제거하며 다른 위치의 모든 소유 파일을 자동 청소한다는 뜻은 아니다. 평가 종료 후 이번에 만든 일회성 계정만 로그아웃·프로세스·보존 자료 확인 뒤 정리한다. 홈까지 지우는 동작을 계정 존재 확인 대신 실행하지 않는다. [userdel(8)](https://man7.org/linux/man-pages/man8/userdel.8.html).

### groupdel

**핵심 · 삭제.** `groupdel 그룹`은 그룹을 삭제하며 사용자 계정을 지우는 명령이 아니다. 누군가의 기본 그룹 등 사용 조건을 확인해야 한다. evaluating을 이번에 새로 만들었고 다른 사용처가 없을 때만 평가 후 정리한다. 기존 그룹은 남긴다. [groupdel(8)](https://man7.org/linux/man-pages/man8/groupdel.8.html).

### authselect

**핵심 · 혼합.** `authselect current`는 선택된 프로필·기능, `authselect check`는 생성된 인증 구성의 유효성을 확인한다. `select`·`apply-changes`는 설정을 바꾸는 하위명령이다. `/etc/authselect`와 연결된 PAM 파일을 관리하므로 system-auth를 임의 편집하면 관리 상태가 깨질 수 있다. 현재 local 프로필 조회는 정책의 모든 비밀번호 거절을 시험한 것과 다르다. 이 문서에서는 조회 두 동작만 사용한다. [authselect(8)](https://github.com/authselect/authselect/blob/master/src/man/authselect.8.adoc).

## 관리 권한과 기록

### sudo

**핵심 · 실행 권한 변경.** `sudo [옵션] 명령 [인수...]`는 sudoers 정책에 따라 다른 자격으로 명령을 실행한다. 기본 대상은 보통 root이며 `-u 사용자`로 바꿀 수 있다. `-l`은 자기 허용 목록, 관리자의 `-l -U 사용자`는 지정 사용자의 허용 조회, `-k`는 인증 캐시 무효화다. `-k`도 상태를 바꾼다.

`sudo` 그룹 가입 자체가 허용 규칙을 만들지는 않는다. b2br 권한은 wheel 규칙에서 오며 `/etc/sudoers.d/90-b2br`의 Defaults는 인증·환경·기록 조건을 더한다. sudo 다음의 명령 인수는 그 명령이 해석한다. root cron 안에서 다시 sudo를 부를 필요가 없고 requiretty와 충돌할 수 있다. [sudo 공식 매뉴얼](https://man7.org/linux/man-pages/man8/sudo.8.html).

### visudo

**핵심 · 혼합.** `visudo`는 sudoers를 잠금·편집·문법 확인 절차로 관리한다. `-c`는 편집 없는 검사, `-f 경로`는 대상 파일 지정, `-P`는 편집 파일의 기본 모드 적용·검사에 관여하고, 소유권은 별도 `-O` 옵션이다. b2br의 `sudo visudo -P -f /etc/sudoers.d/90-b2br`는 정책 파일 변경이고 `sudo visudo -c`는 전체 구성 검사다. `parsed OK`와 실제로 원하는 명령을 허용하는지는 별도다. 파일 한 개만 검사하고 포함 관계 전체를 검증했다고 하지 않는다. [visudo 공식 매뉴얼](https://man7.org/linux/man-pages/man8/visudo.8.html).

### sudoreplay

**핵심 · 조회·재생.** `sudoreplay [옵션] 세션ID`는 sudo I/O 기록을 재생한다. `-d 디렉터리`는 로그 위치, `-l`은 세션 목록, `-f stdin,ttyin`은 재생할 입력 스트림 선택이다. ID는 명령 이름이나 PID가 아니라 목록에서 고른 기록 식별자다.

`sudo sudoreplay -d /var/log/sudo -l`은 b2br 보관 세션을 나열한다. 기본 출력 재생과 입력 확인을 구분한다. 목록 행 수는 현재 보존된 세션 수이며 시스템 수명 전체의 sudo 횟수가 아니다. 자기 조회도 sudo 기록을 만들 수 있다. 비밀 없는 시험 세션만 선택한다. [sudoreplay 공식 매뉴얼](https://man7.org/linux/man-pages/man8/sudoreplay.8.html).

### su

**보조 · 실행 자격 변경.** `su [옵션] [사용자]`는 다른 자격으로 셸·명령을 실행한다. `su - 사용자`의 `-`는 로그인 셸과 비슷한 환경 초기화를 요청하고 `-c '명령'`은 명령 문자열을 실행한다. 일반적인 구성에서 sudo는 호출자의 비밀번호를, su는 전환할 계정의 비밀번호를 묻지만 root 호출·PAM·sudoers 설정에 따라 달라진다. 인증 대상과 허용 정책을 같은 것으로 보지 않는다. b2br 일상 관리에 su를 추가로 써야 하는 요구는 없으며, 셸이 바뀌면 HOME·PATH·현재 자격도 확인해야 한다. [su(1)](https://man7.org/linux/man-pages/man1/su.1.html).

### sudoedit

**핵심 · 파일 변경.** `sudoedit 파일...` 또는 `sudo -e 파일...`은 정책이 허용한 파일을 사용자의 편집기로 편집하고 결과를 반영한다. 편집기 전체를 관리자 권한으로 실행하는 것과 다르다. login.defs·SSH drop-in에는 이 방식을 쓰지만 sudoers는 문법 검사 흐름이 있는 visudo가 담당한다. 편집 성공은 실행 중 서비스 재읽기까지 뜻하지 않는다. [sudo의 편집 모드](https://man7.org/linux/man-pages/man8/sudo.8.html).

## 접속과 보안

### ss

**핵심·수집 · 조회.** `ss [옵션] [필터]`는 소켓 상태를 본다. `-l`은 리스닝만, `-t`는 TCP, `-n`은 숫자로 표시, `-p`는 연결된 프로세스, `-H`는 헤더 제거다. 결합한 `sudo ss -lntp`는 서버의 TCP 수신 주소·포트·프로세스를 조사한다. 권한이 부족하면 프로세스 정보가 빠질 수 있다.

monitoring의 `ss -Htn state established`에서 `state established`는 상태 필터다. 이후 `wc -l`은 해당 TCP 소켓 행을 세며 사용자 수나 리스닝 포트 수를 세지 않는다. 4242 LISTEN과 실제 원격 로그인 성공은 다른 단계다. [ss(8)](https://man7.org/linux/man-pages/man8/ss.8.html).

### ssh

**핵심 · 접속·원격 실행.** `ssh [옵션] [사용자@]호스트 [원격명령]`은 SSH 클라이언트다. `-p 포트`는 접속 포트, `-i 파일`은 사용자 인증용 개인키, `-o 이름=값`은 개별 설정, `-v`는 진단 출력이다. 개인키 파일 내용은 공유하지 않는다.

**호스트**에서 `ssh -p 4242 defenseuser@127.42.42.42`는 현재 NAT 전달 입구로 접속한다. user@host는 옵션이 아니라 접속 대상이다. 첫 서버 키 확인은 사용자 비밀번호 인증보다 앞서며 공개키 지문을 대조한다. `-o NumberOfPasswordPrompts=1`은 클라이언트의 비밀번호 질문 횟수만 제한하고 sudo나 서버 정책을 바꾸지 않는다. 접속하면 그 뒤 입력은 게스트 셸에서 실행된다. [OpenSSH ssh](https://man.openbsd.org/ssh).

### sshd

**핵심 · 서버 실행·검사.** sshd는 ssh와 달리 접속을 받는 서버다. `sshd -t`는 설정 문법·키 관련 검사를 수행하고, `-T`는 유효 설정을 출력한다. `-C user=...,host=...,addr=...`는 `-T`와 함께 Match 판단의 접속 조건을 지정할 때 쓴다. 검사와 실제 데몬 실행을 구분해 새 sshd를 임의로 하나 더 시작하지 않는다.

`sudo sshd -T`에서 port·permitrootlogin을 읽으면 현재 파일 해석을 볼 수 있지만 이미 실행 중인 서버의 재읽기 여부·실제 소켓까지 입증하지 않는다. `/etc/ssh/sshd_config`와 Include 파일 변경 후 문법 확인, 적절한 서비스 반영, ss와 새 접속을 구분한다. 기존 관리자 연결은 유지한다. [OpenSSH sshd](https://man.openbsd.org/sshd).

### ssh-keygen

**핵심 · 혼합.** 키 생성뿐 아니라 공개키 지문 조회도 한다. `ssh-keygen -l -f 공개키파일`에서 `-l`은 지문 목록, `-f`는 대상 파일이다. VM 콘솔에서 `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`는 해당 서버 공개키의 지문을 보여 준다. 접속 화면에서 제시한 키 종류와 일치하는 공개키를 비교한다. 인수 없는 실행은 새 키 생성 흐름이므로 조회 대용이 아니다. `-R 호스트`도 known_hosts 기록 삭제다. [ssh-keygen](https://man.openbsd.org/ssh-keygen).

### firewall-cmd

**핵심 · 혼합.** firewalld에 정책을 조회·변경하는 CLI다. `--state`는 서비스 상태, `--get-active-zones`는 인터페이스·출발지와 zone 연결, `--zone=public`은 작업 대상, `--list-all`은 그 zone의 서비스·포트 등 설정이다. `--list-ports`만 읽으면 서비스 이름으로 허용한 통신을 놓친다.

`--add-port=8080/tcp`와 `--remove-port=8080/tcp`는 각각 허용 추가·제거다. `/tcp`는 프로토콜이며 숫자만의 별명은 아니다. `--permanent` 없이 바꾸면 runtime, 붙이면 영구 설정을 바꾼다. **`--reload`는 영구 설정을 다시 읽어 runtime을 재구성하므로 runtime에만 있던 변경은 사라질 수 있다.** 반대로 `--runtime-to-permanent`는 현재 전체 runtime을 영구화하므로 임시 8080까지 남길 수 있다. b2br 시연은 기존 규칙을 확인한 뒤 임시 추가·조회·제거하며 서버나 NAT를 만들지는 않는다. [firewall-cmd](https://firewalld.org/documentation/man-pages/firewall-cmd.html).

### getenforce

**핵심 · 조회.** 인수 없는 `getenforce`는 현재 SELinux 모드 Enforcing·Permissive·Disabled를 출력한다. b2br 기대값은 Enforcing이다. `/etc/selinux/config`의 다음 부팅 설정과 현재 모드는 별도이며, 이 조회는 특정 SSH 포트 정책의 성공 검사가 아니다. [getenforce(8)](https://man7.org/linux/man-pages/man8/getenforce.8.html).

### sestatus

**핵심 · 조회.** `sestatus`는 SELinux 활성 상태·로드된 정책·현재 모드·설정 파일 모드 등을 묶어 보여 준다. getenforce보다 맥락이 많아 현재와 부팅 설정의 차이를 읽을 때 유용하다. 모든 파일의 레이블이나 모든 거부 원인을 나열하는 도구는 아니다. [sestatus(8)](https://man7.org/linux/man-pages/man8/sestatus.8.html).

### semanage

**핵심 · 혼합.** `semanage 대상종류 [동작] [인수]`는 SELinux의 관리 가능한 정책 설정을 다룬다. `port -l`은 포트 유형 목록, `port -a`는 추가, `port -m`은 기존 항목 변경, `-t ssh_port_t`는 유형, `-p tcp`는 프로토콜이다. `sudo semanage port -a -t ssh_port_t -p tcp 4242`는 SSH 포트의 정책 매핑을 바꾸며 방화벽 허용이나 sshd 리스닝을 만들지 않는다.

이미 매핑된 포트에 무조건 -a를 반복하지 않는다. 실제 기존 유형을 조사한 뒤 필요한 변경만 한다. Rocky에서는 관련 관리 패키지 설치가 필요할 수 있다. 생성된 정책 저장소를 직접 고치지 않는다. [semanage(8)](https://man7.org/linux/man-pages/man8/semanage.8.html) · [semanage-port(8)](https://man7.org/linux/man-pages/man8/semanage-port.8.html).

## 저장장치와 파일시스템

### lsblk

**핵심·수집 · 조회.** `lsblk [옵션] [장치...]`는 디스크·파티션·암호화 연결·LV 같은 블록 장치 관계를 보여 준다. `-o 열목록`은 필요한 열 선택, `-n`은 헤더 제거, `-r`은 raw 출력이다. `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS`는 크기·장치 종류·파일시스템·마운트를 나누어 읽게 한다.

monitoring의 `lsblk -rn -o TYPE`은 TYPE 값만 받아 lvm 행 존재를 검사한다. lvm이 하나 있다는 결과만으로 과제의 모든 암호화·분할 요구가 충족되지는 않는다. SIZE는 장치 용량이지 파일시스템의 남은 용량이 아니다. [lsblk(8)](https://man7.org/linux/man-pages/man8/lsblk.8.html).

### findmnt

**핵심 · 조회.** `findmnt [옵션] [대상]`은 마운트 관계를 찾는다. `findmnt /home`은 그 마운트를 확인하고, `findmnt -T /etc`는 지정 경로를 담는 파일시스템을 찾는다. `-o TARGET,SOURCE,FSTYPE,OPTIONS`로 출력 열을 고를 수 있다. `/etc`가 `/` 파일시스템에 포함되거나 `/var/log`가 별도 LV에 붙는 차이를 읽는다. fstab 설정과 현재 마운트 상태는 같지 않을 수 있다. [findmnt(8)](https://man7.org/linux/man-pages/man8/findmnt.8.html).

### pvs

**핵심 · 조회.** `pvs [옵션] [PV...]`는 LVM 물리 볼륨의 요약 보고서다. `-o pv_name,vg_name,pv_size,pv_free`는 열을 선택한다. PV가 어느 VG에 속하며 얼마나 배정되지 않았는지 읽는다. `/dev/mapper`의 열린 LUKS 장치가 PV가 될 수 있다. 이름이 비슷한 pvcreate는 생성 도구로, 이 조회와 다르다. [pvs(8)](https://man7.org/linux/man-pages/man8/pvs.8.html).

### vgs

**핵심 · 조회.** `vgs [옵션] [VG...]`는 볼륨 그룹의 공간 풀을 보고한다. `-o vg_name,vg_size,vg_free`로 총량·미배정 여유를 좁혀 읽는다. b2br의 VG 이름은 LVM이지만 일반 명령 이름이나 모든 VG의 고정 이름은 아니다. VFree는 LV 밖 풀의 공간이므로 df의 파일시스템 여유와 다르다. [vgs(8)](https://man7.org/linux/man-pages/man8/vgs.8.html).

### lvs

**핵심 · 조회.** `lvs [옵션] [LV...]`는 논리 볼륨을 보고한다. `-o lv_name,vg_name,lv_size`는 이름·소속·용량을 선택한다. root·home·var_log·swap을 각각의 용도와 연결한다. LV 크기가 커졌어도 내부 파일시스템 확대는 별도다. 생성·크기 변경 명령을 이 보고 도구와 혼동하지 않는다. [lvs(8)](https://man7.org/linux/man-pages/man8/lvs.8.html).

### df

**핵심·수집 · 조회.** `df [옵션] [경로...]`는 파일시스템 전체의 사용·가용 공간을 본다. `-h`는 사람이 읽는 단위, GNU `-m`은 MiB 단위, `-x 유형`은 해당 파일시스템 유형 제외, `--total`은 집계 행 추가다. b2br은 tmpfs·devtmpfs를 제외한 total 행을 읽는다. 모든 디렉터리 사용량을 개별 합산하는 명령은 아니다.

`df /var/log`는 그 경로를 담는 파일시스템의 전체 사용량이다. Used/Size 계산과 도구의 Use%는 예약 공간 등의 처리로 다를 수 있다. 별도 LV인 /home의 여유를 /var/log가 자동으로 빌려 쓰지 않는다. [df(1)](https://man7.org/linux/man-pages/man1/df.1.html).

### du

**보조 · 조회.** `du [옵션] [경로...]`는 경로 아래 파일들의 저장 공간을 집계한다. `-s`는 합계만, `-h`는 사람이 읽는 단위, `-x`는 다른 파일시스템으로 넘어가지 않는 옵션이다. `du -sh 경로`는 그 트리의 사용량을 묻고 df는 파일시스템 전체를 묻는다. 권한 때문에 빠진 파일·열린 채 삭제된 파일·메타데이터 등으로 두 숫자는 다를 수 있다. 오류를 숨기고 정확한 전체 합계로 읽지 않는다. [du(1)](https://man7.org/linux/man-pages/man1/du.1.html).

## 자원과 세션 수집

### free

**수집 · 조회.** `free [옵션]`는 메모리·swap 집계를 보여 준다. `-m`은 MiB, `-h`는 사람이 읽는 단위다. total·used·free·available은 다르며 available은 회수 가능한 캐시 등을 고려한 새 작업용 추정치다. b2br은 `LC_ALL=C free -m`의 Mem 행에서 used/total을 계산한다. 출력 열·used 정의는 설치된 procps 버전과 대조하며 free가 작다고 즉시 고갈이라고 하지 않는다. [free(1)](https://man7.org/linux/man-pages/man1/free.1.html).

### lscpu

**수집 · 조회.** `lscpu [옵션]`는 CPU 토폴로지 정보를 보여 준다. `-p=SOCKET`·`-p=CPU`는 지정 열의 파싱용 출력을 만든다. 주석 행을 제외하고 SOCKET의 고유 값 수를 세는 것과 CPU 행 수를 세는 것은 다른 집계다. b2br VM에서 보이는 토폴로지는 게스트에 제공된 것이며 호스트 물리 CPU를 직접 센 값이 아니다. 온라인/오프라인 범위도 옵션·출력 형식에 맞춰 해석한다. [lscpu(1)](https://man7.org/linux/man-pages/man1/lscpu.1.html).

### uname

**수집 · 조회.** `uname [옵션]`은 커널·시스템 정보를 출력한다. `-a`는 여러 항목, `-r`은 커널 릴리스, `-m`은 머신 하드웨어 이름이다. monitoring의 Architecture에는 -a 출력을 넣는다. Rocky 배포판 버전은 /etc/os-release가 답하므로 uname -r과 다른 값인 것이 정상이다. [uname(1)](https://man7.org/linux/man-pages/man1/uname.1.html).

### uptime

**수집 · 조회.** `uptime`은 가동 시간·로그인 수·load average 등을, `uptime -s`는 부팅 시각을 출력한다. b2br은 -s 결과에서 날짜·분까지 선택한다. 기본 출력의 load average는 CPU 사용률 %가 아니고, -s는 경과 시간 문자열이 아니다. [uptime(1)](https://man7.org/linux/man-pages/man1/uptime.1.html).

### who

**수집 · 조회.** `who [옵션]`는 로그인 기록의 사용자·터미널·시각 등을 보여 준다. 한 사람이 콘솔과 SSH에 들어가면 여러 행일 수 있다. monitoring은 첫 필드 사용자명을 뽑아 중복 제거 후 세므로 행 수와 다르다. 계정 DB의 전체 사용자 수나 모든 실행 프로세스 소유자를 나타내지는 않는다. [who(1)](https://man7.org/linux/man-pages/man1/who.1.html).

### vmstat

**수집 · 조회.** `vmstat [옵션] [간격 [횟수]]`는 프로세스·메모리·I/O·CPU 집계를 보여 준다. `vmstat 1 2`의 두 숫자는 옵션명이 아니라 **1초 간격, 두 보고**라는 인수다. CPU 등 기본 첫 통계 보고는 부팅 이후 평균이며 다음 보고는 구간 측정이라 b2br은 마지막 행을 쓴다. 프로세스·메모리 항목은 각 보고 시점의 값이다.

CPU의 us·sy·id·wa 등을 구분한다. `100-id`는 비유휴 시간이지 사용자 프로그램의 순수 계산 시간만이 아니다. `$15`를 id로 읽는 기존 코드는 설치된 출력 열과 함께 확인해야 한다. [vmstat(8)](https://man7.org/linux/man-pages/man8/vmstat.8.html).

### ip

**수집·핵심 · 혼합.** `ip [옵션] 객체 [명령]`은 네트워크 주소·장치·경로를 다룬다. `address show`는 주소 조회, `link show`는 장치, `route show`는 경로다. `-4`는 IPv4, `-o`는 한 줄 형식, `dev enp0s3`는 장치 선택이다. monitoring의 `ip -4 -o address show dev enp0s3`에서 네 번째 필드는 주소/접두부 길이다. 주소가 여러 개면 여러 행이 나올 수 있다.

`add`·`delete`·`set`은 변경이므로 show와 구분한다. 게스트 인터페이스 조회가 호스트 NAT 규칙 조회를 대신하지 않는다. [ip(8)](https://man7.org/linux/man-pages/man8/ip.8.html).

## 예약과 방송

### crontab

**핵심 · 혼합.** `crontab -l`은 호출자의 예약 목록, `-e`는 편집, `-r`은 전체 예약 삭제다. `sudo crontab -e`는 root 예약을 편집하며 일반 사용자의 crontab과 다르다. `crontab 파일`도 파일 내용을 예약으로 설치하는 변경이므로 단순 파일 조회가 아니다. monitoring 중단에는 -r 대신 해당 두 줄 주석을 쓴다.

**crond는 실행 데몬, crontab은 관리 명령이자 시간표 형식**이다. 사용자 crontab의 다섯 시간 필드 뒤에는 명령이 오고, `/etc/crontab`·`/etc/cron.d`의 시스템 형식에는 사용자 칸이 추가된다. 사용자 crontab에 root 칸을 덧붙이면 명령 부분으로 해석될 수 있다. `*/10`은 매시 0·10·20…분이며 이전 실행 종료 후 10분이 아니다. `@reboot`는 다섯 시간 필드 대신 부팅 시 실행을 지정하는 별도 예약이다.

cron 명령 부분의 **이스케이프하지 않은 `%`는 줄바꿈·표준입력 구분으로 특별 처리**된다. 셸 따옴표만으로 이를 막는다고 가정하지 않는다. b2br은 복잡한 printf를 crontab에 직접 넣지 않고 스크립트 파일 안에 둔다. PATH·작업 디렉터리·TTY도 대화형 셸과 같지 않다. [Cronie crontab 명령](https://github.com/cronie-crond/cronie/blob/master/man/crontab.1) · [파일 형식](https://github.com/cronie-crond/cronie/blob/master/man/crontab.5).

### wall

**핵심 · 출력·통신.** `wall [메시지]`는 로그인 터미널에 메시지를 방송한다. 인수가 없으면 표준입력을 읽으므로 `{ ...; } | wall`로 만든 보고서를 한 번 전달할 수 있다. 출력이 나온 터미널 하나만 확인해 모든 터미널에 도달했다고 하지 않는다. 권한·터미널 수신 설정·로그인 시점의 영향을 받으며, 부팅 때 수신자가 없을 수도 있다. 스크립트를 수동 실행하면 실제 방송이므로 문법 조회와 다르다. [wall(1)](https://man7.org/linux/man-pages/man1/wall.1.html).

## 파일·설치·권한·서명·패키지

### install

**핵심 · 변경.** `install [옵션] 원본 대상`은 파일을 복사하면서 소유자·그룹·모드를 지정한다. DNF처럼 저장소에서 패키지를 설치하는 도구가 아니다. `-o root`는 소유자, `-g root`는 그룹, `-m 755`는 모드다. `sudo install -o root -g root -m 755 monitoring.sh /usr/local/sbin/monitoring.sh`는 현재 디렉터리의 작업본을 운영 위치에 복사하고 속성을 설정한다. 대상이 있으면 덮어쓸 수 있다. 이후 작업본만 수정해도 설치본이 자동 변경되지는 않는다. [install(1)](https://man7.org/linux/man-pages/man1/install.1.html).

### chmod

**핵심 · 변경.** `chmod 모드 파일...`은 소유자가 아니라 접근 모드를 바꾼다. 숫자 755는 소유자 rwx·그룹 r-x·기타 r-x이며 r=4, w=2, x=1을 합친다. `u+x`는 소유자 실행 비트만 추가하므로 전체 모드를 지정하는 755와 범위가 다르다. 디렉터리의 x는 경로 통과·탐색 권한이다.

b2br의 monitoring 755와 sudoers 0440은 서로 다른 역할의 권한이다. `-R`은 하위까지 재귀 변경하므로 파일 하나의 문제를 디렉터리 전체 변경으로 해결하지 않는다. SELinux 정책이나 파일 소유권은 chmod로 대체하지 않는다. [chmod(1)](https://man7.org/linux/man-pages/man1/chmod.1.html).

### chown

**핵심 · 변경.** `chown 소유자[:그룹] 파일...`은 소유 정보를 바꾼다. `root:root`는 사용자와 그룹 둘 다, `:그룹`은 그룹을 지정한다. `sudo chown root:root 파일`이 파일을 실행 가능하게 만들지는 않는다. 모드는 chmod·stat로 따로 본다. `-R`은 재귀 범위를 넓히므로 b2br의 특정 파일 확인을 전체 /etc 변경으로 확장하지 않는다. [chown(1)](https://man7.org/linux/man-pages/man1/chown.1.html).

### mkdir

**보조 · 변경.** `mkdir [옵션] 디렉터리...`는 디렉터리를 만든다. `-p`는 필요한 부모를 만들고 이미 있는 디렉터리를 오류로 보지 않게 한다. 따라서 새 기준 백업 폴더의 충돌을 발견하려는 경우 -p가 목적에 맞지 않을 수 있다. b2br 품질 설정 폴더 준비와 새로운 VDI 보관 폴더 생성은 이런 조건이 다르다. [mkdir(1)](https://man7.org/linux/man-pages/man1/mkdir.1.html).

### sha1sum

**핵심 · 조회.** `sha1sum 파일...`은 각 파일 내용의 SHA-1과 파일명을 출력한다. 과제 서명 대상은 **호스트의 실제 연결된 VDI**이며 ISO·VM 설정 파일이 아니다. 공백 있는 경로는 큰따옴표로 묶고 정상 종료된 디스크를 대상으로 한다. 출력의 해시 부분을 signature.txt와 비교한다.

`-c 목록파일`은 파일명까지 포함된 체크섬 목록을 검증하는 기능이다. 해시만 담은 과제 signature.txt가 자동으로 그 목록 형식이 되는 것은 아니다. SHA-1은 과제 지정 형식이며 새로운 보안 서명 설계의 권장이 아니다. [sha1sum(1)](https://man7.org/linux/man-pages/man1/sha1sum.1.html).

### sha256sum

**핵심 · 조회.** `sha256sum 파일...`은 SHA-256 해시를 출력한다. b2br은 monitoring 시연 전후 내용이 유지됐는지 비교하는 데 쓴다. 스크립트 해시가 같아도 소유자·권한은 stat로 별도 확인한다. 디스크 제출의 SHA-1 요구를 이 명령으로 임의 대체하지 않는다. [sha256sum(1)](https://man7.org/linux/man-pages/man1/sha256sum.1.html).

### cp

**핵심 · 변경.** `cp [옵션] 원본 대상`은 복사한다. 마지막 대상이 디렉터리이면 그 안에 원본 이름으로 복사될 수 있으므로 원본·대상 방향을 먼저 읽는다. `-i`는 기존 파일 덮어쓰기 전 질문하며, 확인 없이 원복이 성공했다는 뜻은 아니다. `-a`는 속성·구조 보존을 시도하는 복사이고 백업의 일관성 검증을 대신하지 않는다.

VDI 복사·복원은 **호스트의 Powered Off·실제 연결 경로·스냅샷/차등 디스크 없음** 조건과 [보존 절차](targets.md#평가-사이의-디스크-보존과-복원)를 따른다. `--sparse=always`는 긴 0 구간의 실제 할당을 줄이려는 옵션이지 내용을 압축 파일로 바꾸는 것은 아니다. 과제 필수 옵션은 아니며 성공한 사본을 cmp·해시로 대조한다. [cp(1)](https://man7.org/linux/man-pages/man1/cp.1.html).

### cmp

**핵심 · 조회.** `cmp 파일1 파일2`는 바이트를 직접 비교한다. 같으면 보통 출력 없이 종료 상태 0, 다르면 1, 오류면 2다. `-s`는 출력 없이 상태로 판단하는 방식이다. VDI 복사 후 두 경로가 같은 파일을 가리키는 실수가 없는지도 확인한다. cmp가 같다고 VM 설정·연결 경로·부팅 성공까지 검증된 것은 아니다. [cmp(1)](https://man7.org/linux/man-pages/man1/cmp.1.html).

### rpm

**핵심 · 혼합.** `rpm -q 패키지`는 설치 상태·버전 조회, `-qf 경로`는 그 파일의 소유 패키지, `-ql 패키지`는 설치 파일 목록이다. 여기서 -f의 인수는 파일, -q 뒤의 기본 인수는 패키지 이름이다. `rpm --eval '%{_dbpath}'`는 RPM 매크로로 DB 경로를 조회한다.

b2br의 `rpm -q sudo openssh-server`는 설치 확인이고 `rpm -qf /usr/bin/bash`는 파일 출처 확인이다. 관리자가 설치한 monitoring이 RPM 소유로 나오지 않아도 출처를 따로 알면 이상으로 단정하지 않는다. `-i/-U/-e`는 설치·갱신·제거이므로 조회와 구분하며 저장소 의존성 처리는 보통 DNF를 이용한다. [rpm(8)](https://man7.org/linux/man-pages/man8/rpm.8.html).

### dnf

**핵심 · 혼합.** `dnf [옵션] 하위명령 [패키지...]`는 저장소의 패키지와 의존성을 관리한다. `info`는 설명, `list --installed`는 설치 목록, `repolist`는 저장소 조회, `install`은 설치, `upgrade`는 갱신, `remove`는 제거다. b2br에서 `sudo dnf install policycoreutils-python-utils`는 semanage 도구를 준비하는 변경이다.

명령에 준 패키지뿐 아니라 의존 패키지도 변경 계획에 포함될 수 있으므로 계획을 읽는다. `-y`는 확인 질문 자동 동의이며 조회 옵션이 아니다. DNF 계열·버전에 따라 하위명령과 출력이 다를 수 있다. `check-update`의 100은 업데이트 존재를 나타낼 수 있어 0이 아닌 값을 모두 장애로 해석하지 않는다. 조회도 저장소 메타데이터를 갱신할 수 있다. [DNF 명령 매뉴얼](https://github.com/rpm-software-management/dnf/blob/master/doc/command_ref.rst).

## 텍스트 처리와 탐색

### grep

**수집·가공 · 조회·출력.** `grep [옵션] 패턴 [파일...]`은 맞는 행을 선택한다. 파일을 생략하면 보통 표준입력을 읽는다. `-v`는 반대로 선택, `-E`는 확장 정규식, `-F`는 고정 문자열, `-x`는 행 전체 일치, `-q`는 출력 대신 성공 여부를 판단한다.

`grep -v '^#'`에서 `^`는 행 시작이므로 lscpu 주석을 제외한다. `grep -qx lvm`은 줄 전체가 lvm인 값의 존재를 검사한다. 단어 포함 검색과 다른 조건이다. 보통 종료값 0은 일치, 1은 없음, 2는 오류다. `if`에서 쓰는 검사는 “출력이 있나”보다 종료 상태의 의미를 읽는다. [grep(1)](https://man7.org/linux/man-pages/man1/grep.1.html).

### awk

**수집·가공 · 조회·출력.** `awk [옵션] '조건 {동작}' [파일...]`은 행을 레코드로, 그 안을 필드로 읽어 선택·계산·출력한다. 기본 공백 구분의 `$1`은 첫 필드, `$0`은 전체 행, `NF`는 필드 수, `NR`은 읽은 행 수다. `-F 구분자`로 구분 방식을 바꾸고 `END`는 입력을 다 읽은 뒤의 동작을 정한다.

`awk '$1 == "Mem:" {print $3}'`는 Mem 행의 세 번째 필드를 읽는다. `awk 'END {printf "%.1f\n", 100 - $15}'`는 마지막 행에 남은 필드로 계산하므로 앞 도구의 열 순서가 중요하다. 작은따옴표는 `$1`을 셸이 먼저 확장하지 않도록 보호한다. grep이 행 선택에 집중한다면 awk는 선택·필드·산술·서식을 함께 다룬다. [GNU awk 매뉴얼](https://man7.org/linux/man-pages/man1/gawk.1.html).

### sort

**수집·가공 · 조회·출력.** `sort [옵션] [파일...]`은 행을 정렬한다. `-u`는 정렬과 중복 제거, `-n`은 숫자 비교, `-k`는 정렬 키 선택이다. 기본 문자열 정렬과 숫자 정렬은 2·10의 순서부터 달라질 수 있고 로케일도 영향을 준다. b2br의 `sort -u`는 소켓 ID나 사용자명 종류를 세기 전 중복을 없앤다. 원본 파일을 자동 덮어쓰지 않는다. [sort(1)](https://man7.org/linux/man-pages/man1/sort.1.html).

### uniq

**보조 · 조회·출력.** `uniq [옵션] [입력 [출력]]`는 **인접한** 중복 행을 처리한다. `-c`는 반복 횟수, `-d`는 중복된 행만 출력한다. A·B·A에서 두 A가 떨어져 있으면 하나로 합쳐지지 않는다. b2br 현재 코드는 sort -u를 쓰며, uniq는 그 선택을 이해하기 위한 비교다. [uniq(1)](https://man7.org/linux/man-pages/man1/uniq.1.html).

### wc

**수집·가공 · 조회·출력.** `wc [옵션] [파일...]`는 줄·단어·바이트 등을 센다. `-l`은 개행 문자 수, `-w`는 단어 수, `-c`는 바이트, `-m`은 문자 수다. 개행 없는 마지막 조각이 있을 때 -l을 막연히 “보이는 행 수”라고 읽으면 오차가 난다. b2br은 헤더·주석을 제거한 한 대상당 한 행의 출력을 세도록 구성했다. 행 수가 무엇의 수인지는 앞 명령이 정한다. [wc(1)](https://man7.org/linux/man-pages/man1/wc.1.html).

### cut

**수집·가공 · 조회·출력.** `cut [옵션] [파일...]`은 각 행의 위치·구분 필드를 잘라 낸다. `-c 1-16`은 위치 범위, `-d / -f 1`은 `/` 구분자의 첫 필드다. 후자는 IP/접두부 길이에서 IP 부분을 얻는다. awk의 기본 공백 필드 분리와 달리 cut -d는 지정 구분자를 기준으로 하므로 정렬용 여러 공백을 곧바로 하나의 구분으로 보지 않는다. b2br 날짜 예시는 ASCII 출력 범위다. [cut(1)](https://man7.org/linux/man-pages/man1/cut.1.html).

### printf

**수집·가공 · 출력.** `printf 형식 [값...]`은 형식에 따라 값을 출력한다. Bash 내장과 외부 명령이 있고 awk 안의 printf는 awk 언어의 함수다. `%s`는 문자열, `%d`는 정수, `%.2f`는 소수 둘째 자리까지, `%%`는 퍼센트 문자, `\n`은 개행이다. 출력 끝에 개행이 자동으로 붙지 않는다.

`printf '#Architecture: %s\n' "$architecture"`는 고정 형식과 데이터를 분리한다. 변수 자체를 형식 문자열로 쓰면 값 안의 %가 서식으로 해석될 수 있다. 이 문법을 cron 줄에 직접 넣으면 cron의 % 처리까지 별도로 고려해야 한다. [printf(1)](https://man7.org/linux/man-pages/man1/printf.1.html) · [Bash 내장 문법](https://man7.org/linux/man-pages/man1/bash.1.html).

### cat

**핵심·가공 · 조회·출력.** `cat [파일...]`은 파일들을 순서대로 표준출력에 보낸다. 파일 인수가 없으면 표준입력을 읽어 출력하므로 sudo I/O 시험에서 비밀 없는 문장을 입력하고 Ctrl+D로 마친다. `cat /etc/hostname`은 파일 내용 조회이며 파일을 수정하지 않는다. 바이너리·장치 파일·개인키·shadow 전체를 같은 방식으로 출력하지 않는다. [cat(1)](https://man7.org/linux/man-pages/man1/cat.1.html).

### less

**보조 · 조회.** `less 파일` 또는 `명령 | less`는 긴 텍스트를 화면 단위로 읽는다. `/문자열`로 검색, `n`으로 다음 결과, `q`로 종료한다. 출력 전체를 터미널에 흘리는 cat과 사용 목적이 다르다. 로그·설정을 읽을 때 필요한 부분으로 이동하며 편집기는 아니다. pager를 종료한 것이 앞 서비스나 SSH 셸을 종료하는 것은 아니다. [less(1)](https://man7.org/linux/man-pages/man1/less.1.html).

### ls

**핵심 · 조회.** `ls [옵션] [경로...]`는 이름·속성을 나열한다. `-l`은 긴 형식, `-a`는 숨김 항목 포함, `-d`는 디렉터리 내용 대신 디렉터리 자체다. `ls -ld /etc/sudoers.d`는 그 디렉터리의 모드·소유자를 본다. 링크 출력의 화살표 대상과 파일 크기·실제 할당량을 구분한다. 출력 열을 기계 처리할 때 파일명 공백과 로케일 때문에 고정 열 가정을 주의한다. [ls(1)](https://man7.org/linux/man-pages/man1/ls.1.html).

### stat

**핵심 · 조회.** `stat [옵션] 파일...`은 파일 메타데이터를 보여 준다. GNU `-c 형식`에서 `%U:%G`는 소유자·그룹 이름, `%a`는 숫자 모드, `%n`은 경로, `%F`는 파일 종류다. `stat -c '%U:%G %a %n' /usr/local/sbin/monitoring.sh`로 root:root·755·위치를 대조한다. 내용 동일성은 해시, 링크 대상의 속성 조회는 -L 등의 별도 동작과 구분한다. macOS stat의 옵션을 그대로 가져오지 않는다. [stat(1)](https://man7.org/linux/man-pages/man1/stat.1.html).

### readlink

**보조 · 조회.** `readlink 링크`는 링크에 저장된 대상을, GNU `readlink -f 경로`는 링크와 경로 요소를 해석한 정규 경로를 출력한다. `readlink -f /bin`은 usr-merge의 실제 연결을 확인할 때 쓰지만 파일시스템 유형까지 알려 주지는 않는다. -f에서는 마지막 요소가 없더라도 경로가 출력될 수 있으므로 존재 검사를 대신하지 않는다. [readlink(1)](https://man7.org/linux/man-pages/man1/readlink.1.html).

### man

**핵심·보조 · 조회.** `man [절번호] 이름`은 설치된 매뉴얼을 읽는다. 1은 사용자 명령, 5는 파일 형식, 8은 관리 명령 등이며 `man 1 crontab`과 `man 5 crontab`은 다른 질문이다. SYNOPSIS에서 문법, OPTIONS에서 옵션 인수, EXIT STATUS에서 결과 판단, FILES에서 관련 경로를 읽는다. `man -k 검색어`는 설명 검색이다. Minimal 설치에 문서가 없으면 명령 미설치와 바로 동일시하지 말고 패키지·문서 제공 상태를 구분한다. [man(1)](https://man7.org/linux/man-pages/man1/man.1.html).

### bash

**핵심·보조 · 명령 해석·실행.** `bash 스크립트경로`는 파일을 Bash로 해석해 실행한다. `bash -n 파일`은 명령을 실행하지 않고 구문을 검사하지만 외부 도구 존재·권한·결과까지 검증하지 않는다. `bash -c 문자열`은 주어진 문자열을 코드로 실행하므로 데이터를 무심코 이어 붙이지 않는다. root cron의 `/bin/bash /usr/local/sbin/monitoring.sh`는 명시한 해석기로 설치본을 실행하는 구조다. 셸 내장 여부는 `type`·`help`로 확인한다. [bash(1)](https://man7.org/linux/man-pages/man1/bash.1.html).

### vi

**보조 · 파일 변경 가능.** `vi 파일`은 편집기를 연다. 일반적인 vi 계열에서 `i`는 입력 모드, Esc는 명령 모드 복귀, `:wq`는 저장 후 종료, `:q!`는 변경을 저장하지 않고 종료다. b2br 작업본을 편집하는 도구이지 실행·설치 도구가 아니다. 실제 `vi` 구현은 배포판 패키지에 따라 다를 수 있다. sudoers는 vi를 직접 열기보다 visudo, root crontab은 crontab -e의 관리 흐름을 따른다. [Vim 시작 안내](https://github.com/vim/vim/blob/master/runtime/doc/usr_02.txt).

## 명령 이름 색인

각 이름은 이 문서의 설명으로 이동한다. 셸 내장 `cd`·`exit`·`command`·`type`과 파이프·리다이렉션은 [셸 문법과 명령](#셸-문법과-명령은-다르다)에서 따로 찾는다.

| A–Z | A–Z | A–Z |
|---|---|---|
| [authselect](#authselect) | [awk](#awk) | [bash](#bash) |
| [cat](#cat) | [chage](#chage) | [chmod](#chmod) |
| [chown](#chown) | [cmp](#cmp) | [cp](#cp) |
| [crontab](#crontab) | [cut](#cut) | [df](#df) |
| [dnf](#dnf) | [du](#du) | [findmnt](#findmnt) |
| [firewall-cmd](#firewall-cmd) | [free](#free) | [getenforce](#getenforce) |
| [getent](#getent) | [grep](#grep) | [groupadd](#groupadd) |
| [groupdel](#groupdel) | [hostname](#hostname) | [hostnamectl](#hostnamectl) |
| [id](#id) | [install](#install) | [ip](#ip) |
| [journalctl](#journalctl) | [less](#less) | [ls](#ls) |
| [lsblk](#lsblk) | [lscpu](#lscpu) | [lvs](#lvs) |
| [man](#man) | [mkdir](#mkdir) | [passwd](#passwd) |
| [printf](#printf) | [pvs](#pvs) | [readlink](#readlink) |
| [reboot](#reboot) | [rpm](#rpm) | [semanage](#semanage) |
| [sestatus](#sestatus) | [sha1sum](#sha1sum) | [sha256sum](#sha256sum) |
| [sort](#sort) | [ss](#ss) | [ssh](#ssh) |
| [ssh-keygen](#ssh-keygen) | [sshd](#sshd) | [stat](#stat) |
| [su](#su) | [sudo](#sudo) | [sudoedit](#sudoedit) |
| [sudoreplay](#sudoreplay) | [systemctl](#systemctl) | [uname](#uname) |
| [uniq](#uniq) | [uptime](#uptime) | [useradd](#useradd) |
| [userdel](#userdel) | [usermod](#usermod) | [vgs](#vgs) |
| [vi](#vi) | [visudo](#visudo) | [vmstat](#vmstat) |
| [wall](#wall) | [wc](#wc) | [who](#who) |
| [whoami](#whoami) |  |  |

## 사용 예시를 설명으로 바꾸기

`lscpu -p=SOCKET | grep -v '^#' | sort -u | wc -l`을 읽을 때에는 먼저 각 명령이 받는 자료와 내보내는 자료를 따라간다. lscpu는 소켓 열을 만들고, grep은 주석 행을 제외하고, sort는 소켓 ID 종류를 남기고, wc는 그 결과의 개행 수를 센다. `-p`, `-v`, `-u`, `-l`을 서로 바꿔 써도 같은 뜻인 것이 아니다. 이 연결이 게스트에서 보이는 고유 소켓 수를 뜻하는 이유까지 설명할 수 있어야 한다.

다음으로 옵션을 바꾸면 질문이 어떻게 달라지는지 비교한다. `systemctl is-active`를 `is-enabled`로 바꾸면 현재 실행에서 시작 연결로, `ls -l`을 `ls -ld`로 바꾸면 디렉터리 내용에서 디렉터리 자체로, `id`를 `id jungblee`로 바꾸면 현재 실행 자격에서 지정 계정 조회로 관점이 바뀐다. 실제 설정 변경·시험은 [필수 참조](essentials.md)와 [디펜스](defense.md)의 선행 조건·원복 순서에서 수행한다.
