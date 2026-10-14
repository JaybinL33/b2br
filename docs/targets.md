# Born2beRoot — 직접 만들고 대조할 목표

**이 문서는 목표 상태·개념·선택 근거·읽을 범위·검증 기준을 정리한다.** 한 단위씩 직접 구성하고 관찰한 결과를 [학습 기록](learning-log.md)에 남긴다. 요구사항은 [원문 v5.2](../en.subject.pdf), 자세한 항목은 [기존 가이드](ko/Born2beRoot-v5.2-ko-guide.md)를 기준으로 한다. 학습 범위는 보너스를 제외한 필수 과제다. 아래는 필수 서버 관리·모니터링 실습에 맞춘 선택이며 보너스 그림을 복제하지 않는다.

## 전체 학습 지도

현재 진행 상태는 [README](../README.md), 실행 증거와 이력은 [학습 기록](learning-log.md), 반복 실습 절차는 [치트시트](cheatsheet.md)에 둔다. 아래 순서로 **전체 역할 파악 → 필요한 개념 읽기 → 한 부분 직접 적용 → 결과 확인 → 배운 내용 기록**을 반복한다. 필수 조건의 숫자와 확인 방법은 이어지는 각 구간에 둔다.

| 구간 | 목적·핵심 개념 | 끝났다고 판단할 상태 | 확인의 중심 | 이번에 미룰 심화 |
|---|---|---|---|---|
| [1. 설치 결과](#1-설치-결과와-디스크-확인) | 부팅, 마운트, LUKS·LVM, 패키지·서비스 | 최소 OS가 부팅되고 설계와 실제 디스크가 일치 | OS 정보, lsblk·vgs·df, 재부팅 | RAID, thin provisioning, 디스크 축소 실행 |
| [2. 계정·그룹·권한](#2-계정그룹권한과-호스트명) | UID/GID, 소유권, 기본·보조 그룹, root와 일반 사용자 | 로그인·그룹·호스트명 조건 충족, 새 계정·그룹과 호스트명 변경을 설명하며 수행 | id·getent·hostnamectl, 소유권·접근 결과 | LDAP/AD, 고급 ACL |
| [3. 비밀번호·PAM](#3-비밀번호pam) | 인증 경로, 품질과 만료의 차이, 기존 계정과 기본값 | 모든 암호 규칙과 root 예외 적용, 기존 계정 비밀번호 갱신 | chage와 실제 허용·거절 동작 | MFA, 사용자 PAM 모듈 작성 |
| [4. sudo·감사 로그](#4-sudo-정책과-감사-로그) | 권한 위임, TTY, 실행 경로, 명령 로그와 입출력 로그 | 제한·오류 메시지·입출력 기록이 실제 동작 | 문법·적용 정책·로그·실패 동작 | 중앙 로그 서버, 복잡한 역할 분리 |
| [5. 원격 접속·보안](#5-sshfirewalldselinux) | sshd, 포트, 방화벽 zone, SELinux 정책 | 4242 일반 사용자 접속, root 거절, 부팅 후 정책 유지 | 서비스·리스닝·접속·거절 로그 | VPN, 고급 라우팅, 자체 SELinux 정책 모듈 |
| [6. 모니터링·예약](#6-monitoringsh와-cronwall) | 지표의 뜻, 표본과 계산, cron 실행 환경, wall 방송 | 모든 원문 지표, 시작 시·10분마다 방송, 수정 없는 중단 | 원자료와 출력 대조, 여러 터미널·시간 관찰 | 외부 대시보드, 장기 통계 수집 |
| [7. 재현·제출](#7-평가-재현과-readme서명) | 재부팅 지속성, 설명·작은 변경, 디스크 서명 | 필수 조건 재현, README 완성, 서명 일치 | 직접 시연과 파일 대조 | 보너스 서비스, 설치 자동화 |

## 자료 선택과 읽는 깊이

아래의 **과제 필수**는 원문 v5.2의 조건이고, **반드시 읽기·설명하기·존재만 알기·이번에 미루기**는 그 조건을 직접 구현하고 설명하기 위한 학습 범위 판단이다. 분류 자체가 과제의 추가 요구는 아니다. 각 구간에 들어갈 때 지정한 부분을 읽으며, 뒤 구간의 안내는 준비 자료다.

자료는 **공식 입문·개념 가이드 → 설정할 때 설치 버전의 man/reference → 문서로 해결되지 않는 쟁점에만 구현 소스** 순으로 고른다. 매뉴얼 전체를 외우지 않고 적힌 제목·옵션을 찾는다. 단순 `cat` 조회와 실제 출력에서 출발하며 소스코드 분석이나 복잡한 검색식을 입문 조건으로 삼지 않는다.

자료 기준은 Rocky 공식 가이드와 RHEL **10** 문서다. Rocky 관리 입문서는 릴리스별로 고정된 책이 아니므로 개념에 사용하고, 설치 안내는 제목이 *Installing Rocky Linux 10*인 문서를 사용한다. RHEL 10의 설명도 Rocky의 패키지 빌드·기본값·현재 설정을 대신 증명하지 않는다. 설정 시 VM의 설치 버전과 해당 `man`을 대조한다. 최신 upstream 매뉴얼의 새 옵션을 Rocky 10에 그대로 있다고 가정하지 않는다.

**존재만 알기**는 용도와 필요해질 상황을 말할 수 있으면 충분하다. **이번에 미루기**는 현재 단일 VM·필수 과제의 범위에서 내부 원리나 확장 설정을 생략한다는 뜻이다. 각 구간에 다시 읽을 시점을 적었다. 읽거나 안내받은 내용과 직접 수행·검증한 내용은 [학습 기록](learning-log.md)에서 구분한다.

## 1. 설치 결과와 디스크 확인

**반드시 읽을 범위:** 원문 인쇄 7–8쪽의 OS·LVM 조건과 그림부터 본다. [Rocky 10 설치 안내](https://docs.rockylinux.org/guides/installation/)에서는 `Software Selection`, `Installation Destination`, `Network & Host Name`, `Root Password`, `User Creation`, `Log In`을 읽는다. 자동 파티션 예시를 우리 수동 배치와 같다고 보지 않는다. [Rocky File System](https://docs.rockylinux.org/books/admin_guide/07-file-systems/)에서는 `Logical Volume Manager (LVM)`의 개요와 조회 명령, `Preparation of the physical media`의 개요, `Organization of a file system`의 `/etc/fstab file`, `Mount management commands`를 읽는다. 생성·포맷 명령 예시는 지금 실행할 절차가 아니다.

**설명할 수 있어야 할 것:** 파티션·암호화 장치·PV/VG/LV·파일시스템·마운트는 서로 다른 층이며, VG 여유와 파일시스템 여유도 다르다. [RHEL 10 Disk partitions](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/disk-partitions) 3.1–3.3과 3.9, [LVM 개요](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_logical_volumes/overview-of-logical-volume-management) 1.1 `LVM architecture`, [LUKS](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/encrypting-block-devices-by-using-luks) 17.1–17.2로 이 관계와 LUKS 버전 구분을 보충한다. 이유는 암호화 LVM 요구를 만족하는 구조와 우리가 정한 용량의 근거를 설명해야 하기 때문이다. BIOS+GPT의 부팅용 영역과 `/boot`도 구분한다.

**존재와 용도만:** MBR은 GPT와 다른 파티션 방식, LVM 스냅샷은 볼륨 상태를 보존하는 기능이라는 정도로 둔다. 이전 디스크를 다루거나 복구 실습을 할 때 상세 내용을 읽는다. LVM 스냅샷은 평가 규정의 VM 스냅샷과 다르다.

**이번에 미룰 것과 이유:** inode·슈퍼블록의 내부 배치, RAID·thin provisioning, LUKS 재암호화 절차는 현재 단일 디스크 구성의 필수 조건이 아니다. 파일시스템 복구·다중 디스크·용량 과할당·키 관리가 목표가 될 때 돌아온다. ext4 축소의 순서·언마운트 필요성은 선택 이유로 이해하되 실제 축소 작업은 별도 복구 환경을 준비하는 때로 미룬다.

**과제 필수:** LVM을 사용한 암호화 파티션 최소 2개. **우리 선택:** BIOS+GPT, 새 30 GB VDI, VG 이름 `LVM`, ext4, VG 미할당 공간 유지. 아래 용량과 이름은 과제 고정값이 아니다.

다음은 `lsblk` 형태로 만든 **가상 목표 예시**다. `약`과 `<UUID>`는 설명용 표기이며 실제 출력이 아니다.

```text
NAME               SIZE TYPE  FSTYPE      MOUNTPOINTS
sda                 30G disk
├─sda1               1M part
├─sda2               2G part  ext4        /boot
└─sda3             약28G part  crypto_LUKS
  └─luks-<UUID>    약28G crypt LVM2_member
    ├─LVM-root       10G lvm   ext4        /
    ├─LVM-home        4G lvm   ext4        /home
    ├─LVM-var_log     2G lvm   ext4        /var/log
    └─LVM-swap        4G lvm   swap        [SWAP]
```

- `sda1`은 BIOS+GPT에 필요한 1 MiB BIOS Boot 영역이다. 일반 파일시스템·마운트·암호화가 없다. 위 열만으로 BIOS Boot 유형까지 판별할 수는 없다.
- `/boot`는 비암호화 Standard Partition이다. 나머지는 **LUKS → 열린 암호화 장치 → PV → VG → LV** 순서다. 개별 LV마다 LUKS를 추가하는 구조가 아니다.
- VG `LVM`에는 **약 8 GiB의 VFree**를 남긴다. VG 미할당 공간은 `lsblk`에 별도 행으로 나오지 않는다. 메타데이터·반올림에 따라 실제 값은 다르다. 계산은 디스크가 약 `30G`로 보이는 경우이며 GB/GiB 표기 차이가 있으면 잔여량도 달라진다.
- 장치명, 파티션 번호·순서, UUID, 암호화 매핑 이름은 달라질 수 있다. 글자 그대로보다 계층·파일시스템·마운트·용량을 비교한다. 설치 ISO의 `sr0` 등은 생략했다.
- root 10 GiB는 설치 후 사용량을 확인할 초기 배정이다. 용량 선택 이유는 아래 표와 대조한다. ext4 축소는 파일시스템을 언마운트하고 먼저 줄인 뒤 LV를 줄이는 순서이므로 실행 중인 root는 별도 부팅 환경이 필요하다.

### 이 목록과 크기를 고른 이유

모든 크기는 실측 최소값이 아닌 **초기 용량 예산**이다. 설치 후 사용량을 확인하고 VG 여유 공간으로 필요한 LV를 확장한다.

| 영역 | 역할과 선택의 대가 |
|---|---|
| root 10 GiB | OS·패키지와 별도 마운트하지 않는 `/var`의 캐시·상태 데이터, `/tmp` 등의 임시파일을 담는다. home보다 맡는 범위가 넓다. 이 영역의 증가량도 확인해야 한다. |
| home 4 GiB | 한 명의 실습 파일을 위한 초기 예산이다. 대용량 자료를 보관하기에 충분하다고 검증한 값은 아니다. |
| var_log 2 GiB | `/var/log/sudo/` 입출력 로그와 이 경로 아래 시스템 로그의 증가를 root 공간과 분리한다. 원문은 로그 경로를 요구하지만 별도 LV는 우리의 선택이다. |
| `/var`, `/tmp`, `/srv` | 별도 마운트하지 않아 root 공간을 공유한다. 서비스 데이터를 둘 계획이 없는 `/srv`에 전용 용량을 예약하지 않는다. |
| VG 여유 약 8 GiB | 당장 한 파일시스템에 몰아주지 않고 실제 사용량에 따라 확장할 공간을 남긴다. |

`/var/log` 2 GiB는 무한 보관을 보장하지 않는다. 사용량 확인과 보관·순환 정책이 필요하며, 로그 볼륨이 가득 차면 sudo 로그 기록과 명령 실행에도 영향을 줄 수 있다. 별도 LV는 root 보호에 도움이 되지만 로그 관리 부담을 없애지는 않는다. `df -h / /home /var/log`와 `sudo vgs`로 파일시스템 사용량과 VG 여유를 구분해 확인한다.

설치기에서는 VG의 `Modify → Encrypt`, 크기 정책 `As large as possible`을 확인하고 root 10 GiB, home 4 GiB, var_log 2 GiB, swap 4 GiB만 배정한다. **VG를 크게 만드는 것과 home에 잔여 전부를 주는 것은 다르다.** 설치 시작 전 파티션 목록과 VG 설정을 대조한다.

직접 설치한 VM에서 확인할 명령:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo vgs
sudo pvs
sudo lvs
```

`vgs`에서 이름 `LVM`, LV 4개, 남아 있는 VFree를 확인한다. BIOS Boot 유형은 설치기 또는 파티션 정보로, LUKS2 여부는 실제 암호화 파티션의 헤더 정보로 별도 확인한다.

## 2. 계정·그룹·권한과 호스트명

**반드시 읽을 범위:** 원문 8쪽의 사용자·그룹·호스트명 조건과 [Rocky User Management](https://docs.rockylinux.org/books/admin_guide/06-users/)의 `General`, `Group management` 중 `groupadd command`·`/etc/group file`, `User management` 중 `Definition`·`useradd command`·`usermod command`·`/etc/passwd file`, `File owners`, `id command`를 읽는다. [File System](https://docs.rockylinux.org/books/admin_guide/07-file-systems/)에서는 `Basic permissions of files and directories`와 `chmod command`만 연결한다. 실제 옵션은 VM의 `man usermod`에서 `-a`, `-G`, `man hostnamectl`에서 `hostname`/`set-hostname`을 찾는다.

**설명할 수 있어야 할 것:** UID/GID, 기본·보조 그룹, 소유자·그룹·권한, 계정 정보와 현재 세션의 그룹 차이, `wheel` 소속과 과제가 요구하는 `sudo` 그룹 이름의 차이다. `id jungblee` → 재로그인 → `id`를 왜 구분하는지 설명한다. 근거는 평가에서 새 사용자·그룹·호스트명 변경을 직접 수행해야 한다는 원문 조건이며, 그룹 존재만으로 권한 정책까지 증명되지 않기 때문이다.

**존재와 용도만:** `/etc/skel`은 새 홈의 초기 파일, NSS는 계정 조회 경로를 연결하는 체계임을 알아둔다. 새 계정 기본 환경이나 `getent`와 로컬 파일 결과 차이를 조사할 때 확장한다.

**이번에 미룰 것과 이유:** LDAP/AD 연결과 고급 ACL 설계는 이번 로컬 계정·기본 권한 실습에 필요하지 않다. 조직 계정 통합이나 기본 소유자·그룹으로 표현할 수 없는 공유 권한이 필요할 때 학습한다. 계정 데이터베이스를 직접 편집하는 방식을 입문 절차로 삼지 않는다.

| 구분 | 도달할 상태 | 확인 방법 |
|---|---|---|
| 과제 필수 | 최신 안정 Debian/Rocky, 최소 서비스, X.org·Wayland 등 그래픽 서버 없음 | `cat /etc/os-release`, 설치 패키지·실행 서비스 확인. 버전 출력만으로 최신 안정 여부나 그래픽 서버 부재를 증명하지는 못한다. |
| 우리 선택 | Rocky Minimal | 설치 매체와 실제 설치 OS 버전을 공식 릴리스 정보와 대조하고 최소 구성으로 부팅되는지 확인 |
| 과제 필수 + 로그인 대입 | root 외에 `jungblee`, 소속 그룹에 `user42`와 `sudo` 포함 | `id jungblee`, `getent group user42`, `getent group sudo`. Rocky의 `wheel`만으로 대신하지 않는다. |
| 과제 필수 + 로그인 대입 | hostname `jungblee42` | `hostnamectl --static`; 재부팅 뒤에도 확인 |

평가에서는 새 사용자를 만들고 그룹에 넣으며 호스트명을 변경할 수 있어야 한다. 이를 준비하며 사용자·그룹 생성, 기본·보조 그룹, 파일 소유권과 권한을 작은 실습으로 확인한다. 호스트명은 셸 프롬프트만 보지 않고 별도 조회와 재부팅 후 지속성을 확인한다. Rocky의 Kdump 설정은 과제에서 요구하지 않는다.

## 3. 비밀번호·PAM

**반드시 읽을 범위:** 원문 8–9쪽의 기간·품질·root 예외·기존 비밀번호 갱신 조건을 기준으로 [Rocky User Management](https://docs.rockylinux.org/books/admin_guide/06-users/)의 `passwd command`, `chage command`, `/etc/shadow file`, `/etc/login.defs file`을 읽는다. [Rocky PAM 입문](https://docs.rockylinux.org/guides/security/pam/)은 `Introduction`부터 `Control Indicators`까지 읽어 네 처리 유형과 제어 흐름을 잡는다. 뒤의 `pam_cracklib`·`pam_tally` 예시는 현재 Rocky 10 구성의 설치·설정 지침으로 사용하지 않는다. [RHEL 10 authselect](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_authentication_and_authorization_in_rhel/configuring-user-authentication-using-authselect)에서는 2.1과 2.1.1, 2.2의 **현재 구성 조회 부분만** 읽는다. 프로필 전환 예시는 지금 실행하지 않는다.

**설정할 때 반드시 찾을 항목:** VM의 `man chage`에서 `-l`, `-m`, `-M`, `-W`, `man login.defs`에서 `PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE`를 찾는다. 아래 연결한 libpwquality 공식 매뉴얼의 `DESCRIPTION`과 `minlen`, `dcredit`, `ucredit`, `lcredit`, `ocredit`, `maxrepeat`, `usercheck`, `difok`, `enforce_for_root`, `use_authtok`를 읽고 설치 버전과 대조한다. 값의 의미는 아래 품질 규칙 설명과 대조하며 작성한다. 파일 작성 안내와 실제 적용·검증은 구분한다.

**설명할 수 있어야 할 것:** 새 계정 기본값과 기존 계정 기간 값, 품질 검사와 저장, `passwd` → `system-auth`의 호출 관계, `requisite` 실패와 `use_authtok` 전달이다. root의 이전 비밀번호 비교 예외와 다른 품질 조건 적용도 구분한다. 특히 **login.defs/libeconf는 기본 파일 뒤 drop-in, libpwquality는 drop-in 뒤 본체이며 PAM 인수가 우선**이라는 차이는 반드시 알아야 한다. 아래 공식 근거처럼 실제 적용 값이 달라지므로 파일을 만들었다는 사실만으로 정책 적용을 판단할 수 없기 때문이다. 2/30/7과 마지막 변경일·실제 만료일도 함께 대조한다.

**존재와 용도만:** `auth`, `account`, `session`은 각각 인증·계정 허용·세션 처리를 맡고, authselect에는 사용자 정의 프로필이 있다. 로그인 실패 원인을 분리하거나 기존 프로필로 표현 못 하는 요구가 생길 때 더 읽는다.

**이번에 미룰 것과 이유:** PAM 모듈 C 구현, yescrypt 계산 내부, MFA·원격 인증 설계는 현재 연결된 품질 모듈에 과제 규칙을 적용하는 데 필요하지 않다. 자체 인증 개발이나 해시·인증 방식 변경 때 학습한다. shadow 소스는 drop-in 지원이 빌드 조건에 달려 있음을 확인하는 보조 근거이며 필독 소스 분석 과제가 아니다.

PAM은 서비스가 인증·계정·비밀번호·세션 처리를 연결하는 틀이다. 비밀번호 품질 검사와 계정별 만료 정보는 따로 확인한다. Rocky의 authselect 관리 여부와 실제 PAM 경로를 파악하고, 기본값 변경만으로 기존 계정도 바뀌었다고 가정하지 않는다.

`passwd`로 비밀번호를 바꿀 때는 PAM의 `password` 스택을 거친다. `pam_pwquality.so`는 새 비밀번호의 품질을 검사하고, 통과한 비밀번호를 뒤의 모듈에 전달해 실제 갱신을 이어 간다. 품질 검사와 저장을 같은 역할로 보지 않는다. 근거: [libpwquality의 pam_pwquality 매뉴얼](https://github.com/libpwquality/libpwquality/blob/master/doc/man/pam_pwquality.8.pod).

authselect가 관리하는 PAM 파일은 곧바로 편집하기보다 현재 프로필과 구성 상태, 실제 `password` 연결을 먼저 읽는다. [RHEL 10 authselect 공식 문서](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_authentication_and_authorization_in_rhel/configuring-user-authentication-using-authselect)도 관리 파일의 수동 편집을 경고한다. `cat /etc/pam.d/passwd`로 진입점을 읽고, 그 파일의 `password` 항목이 참조하는 `system-auth`를 `cat /etc/pam.d/system-auth`로 따라간다. `authselect current`와 `authselect check`는 관리 상태를 확인하는 조회다. 구체적인 순서는 [치트시트](cheatsheet.md)에 둔다.

`passwd`가 참조하는 비밀번호 스택에서 품질 검사와 로컬 갱신의 연결을 확인한다. 예를 들어 `pam_pwquality.so` → `pam_unix.so` → `pam_deny.so` 구조라면, `requisite`인 품질 모듈의 실패는 해당 스택을 즉시 실패로 돌려보낸다. `pam_unix.so`의 `use_authtok`는 앞에서 받은 새 비밀번호를 이용한다. 필요한 품질 모듈이 이미 연결되어 있다면 규칙 설정을 위해 PAM 파일 자체를 다시 편집할 필요는 없다.

품질 규칙 작성 전 본체·분리 파일·PAM 인수의 활성 설정을 함께 읽는다. libpwquality는 drop-in을 먼저 파일명 ASCII 순서로 읽고 본체를 나중에 읽으므로, 본체에 같은 활성 항목이 있으면 drop-in 값을 덮는다. PAM 모듈 인수는 설정 파일보다 우선한다. 따라서 본체의 중복 설정도 확인한다. 이는 앞의 login.defs/libeconf와 읽기 순서가 다르다. 근거: [pwquality.conf 공식 매뉴얼](https://github.com/libpwquality/libpwquality/blob/master/doc/man/pwquality.conf.5.pod), [pam_pwquality 공식 매뉴얼](https://github.com/libpwquality/libpwquality/blob/master/doc/man/pam_pwquality.8.pod). 각 품질 위반 조건의 거절은 별도로 검증한다.

`/etc/login.defs.d/*.defs` 지원은 shadow-utils의 libeconf 사용 빌드에 달려 있다. [shadow 4.15.0의 읽기 코드](https://github.com/shadow-maint/shadow/blob/4.15.0/lib/getdef.c)는 `USE_ECONF` 조건으로 libeconf를 호출하며, [libeconf 공식 설명](https://github.com/openSUSE/libeconf/blob/master/README.md)은 분리된 설정 파일의 읽기 구조를 설명한다. libeconf는 기본 파일 뒤에 drop-in을 읽으며 파일명 사전순으로 처리한 뒤의 값이 앞의 값을 덮는다. 따라서 `/etc/login.defs.d/90-local.defs`에 세 기간 값을 모을 수 있지만, 더 뒤의 파일에 같은 키가 있으면 그 값이 우선한다. VM에서는 패키지 의존성으로 지원 빌드인지 확인하고 별도 파일 방식을 사용한다. 미지원이면 원본 파일 편집을 대안으로 선택하며 두 절차를 연달아 수행하지 않는다. 원본의 기존 변경을 복구하거나 삭제할 필요는 없다. 어느 방식도 기존 계정의 기간 값을 바꾸는 `chage`를 대신하지 않는다.

**비밀번호 필수 목표:** 만료 30일, 변경 최소 간격 2일, 만료 7일 전 경고. 최소 10자이며 대문자·소문자·숫자를 포함하고 사용자명을 포함하지 않는다. 동일 문자 연속은 최대 3개다. 이전 비밀번호에 없던 문자 최소 7개가 필요하며 **이 조건만 root 예외**다. 정책 구성 후 root를 포함해 VM의 모든 계정 비밀번호를 변경해야 한다.

`sudo chage -l jungblee`, `sudo chage -l root`로 기간 정책을 확인한다. 문자 조건은 실제 적용되는 인증·품질 정책과 허용/거절 동작을 대조한다. `chage`만으로 길이·문자 조건은 검증되지 않는다. 비밀번호 자체는 문서나 증거에 남기지 않는다.

기간 검증 기준은 두 계정의 마지막 변경일이 실제 변경 날짜이고 만료일은 그로부터 30일 뒤이며 최소 2일·최대 30일·경고 7일이 유지되는 것이다. `Password inactive`와 `Account expires`는 각각 비밀번호 만료 후 비활성 기한과 계정 만료일로, 비밀번호 만료와 별개다. 이 과제 범위에서는 추가 만료 기한을 설정하지 않는다.

### 품질 규칙과 선택 이유

품질 규칙은 `/etc/security/pwquality.conf.d/90-local.conf`에 모으는 구성을 선택한다. 정확한 파일 내용과 명령은 [치트시트](cheatsheet.md#5-별도-품질-설정-파일-작성)에 둔다. 아래는 각 설정값이 과제 조건과 연결되는 이유다. 근거는 [pwquality.conf 매뉴얼](https://github.com/libpwquality/libpwquality/blob/master/doc/man/pwquality.conf.5.pod)의 각 옵션과 [pam_pwquality 매뉴얼](https://github.com/libpwquality/libpwquality/blob/master/doc/man/pam_pwquality.8.pod)의 credit·비교·root 처리 설명이다.

| 항목 | 의미와 선택 이유 |
|---|---|
| `minlen = 10` | 길이 기준을 10으로 둔다. 아래 credit을 모두 0 이하로 두어 문자 종류에 따른 길이 가산 없이 실제 길이를 검사한다. |
| `ucredit = -1`, `lcredit = -1`, `dcredit = -1` | 대문자·소문자·숫자를 각각 최소 1개 요구한다. 음수의 절댓값은 최소 개수이고, 양수는 길이 계산에 줄 수 있는 가산의 한도다. |
| `ocredit = 0` | 특수문자 등 기타 문자 종류를 강제하지 않고 길이 가산도 주지 않는다. 사용을 금지하는 뜻은 아니다. |
| `maxrepeat = 3` | 같은 문자 3개 연속은 이 규칙으로 거절하지 않고 4개 연속부터 거절한다. 다른 품질 검사까지 통과한다는 뜻은 아니다. |
| `usercheck = 1` | 사용자명 포함 여부를 검사한다. 기본값도 1이지만 과제 정책을 명시하려고 적는다. |
| `difok = 7` | 이전·새 비밀번호의 차이를 요구한다. PAM 매뉴얼은 삽입·삭제·치환에 따른 문자 변경 수로 설명하므로 단순 문자열 길이 차이나 집합 차이로 이해하지 않는다. |
| `enforcing = 1` | 품질 실패를 경고만 하고 넘기지 않고 거절한다. 기본값 1을 정책에 명시한다. |
| `enforce_for_root` | 안내에서는 값 없이 켜는 표기를 사용한다. root가 변경할 때도 품질 실패를 거절하지만, root는 이전 암호를 입력하지 않아 이전·새 비밀번호 비교는 생략된다. |

사전 검사 등 나머지 기본값은 유지한다. 일반 계정의 이전 비밀번호 비교는 사용자가 직접 `passwd`를 실행하는 경로에서 검증해야 한다. 관리자의 대리 변경으로는 `difok` 동작을 입증할 수 없다. 특히 `sudo passwd jungblee`도 root 권한으로 다른 계정의 비밀번호를 바꾸므로 이전 비밀번호 비교를 생략한다. sudo 인증에서 입력하는 암호는 PAM password 스택에 제공할 변경 대상의 이전 암호와 다르다. `enforce_for_root`는 이 비교 생략을 되돌리는 옵션이 아니다. 재실습 순서는 jungblee의 `passwd` 성공 → `sudo passwd root` 성공 → `chage -l jungblee`와 `sudo chage -l root` 조회다. 마지막 변경일이 실제 날짜로 바뀌고 만료일은 30일 뒤이며 2/30/7이 유지되는지 확인한다. 정상 변경 성공은 각 위반 조건을 거절한다는 증거와 별개다. 이후 재변경에는 최소 2일 제한이 영향을 줄 수 있다. 오류가 나면 비밀번호를 제외한 메시지를 보고 판단한다. 비밀번호는 기록하지 않는다.

## 4. sudo 정책과 감사 로그

**반드시 읽을 범위 — 이 구간에 들어갈 때:** 원문 9쪽과 [RHEL 10 Managing sudo access](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/security_hardening/managing-sudo-access) 8.1 `User authorizations in sudoers`, 8.2 `Granting sudo access to a user`로 권한 위임을 이해한다. 먼저 VM의 `man sudoers`에서 `User specification`, `Defaults`와 include 지시를 읽는다. [공식 sudoers 매뉴얼 원본](https://github.com/sudo-project/sudo/blob/main/docs/sudoers.man.in)에서도 같은 제목을 찾을 수 있다. 설정 시에는 `passwd_tries`, `badpass_message`, `requiretty`, `secure_path`, `log_input`, `log_output`, `iolog_dir`를 찾고, [공식 visudo 매뉴얼](https://github.com/sudo-project/sudo/blob/main/docs/visudo.man.in)과 VM의 `man visudo`에서 `DESCRIPTION`, `-f`, `-c`, `-P`를 읽는다. visudo는 동시 편집을 잠그고 저장 전 문법을 검사한다. 별도 파일 편집의 문법 검사와 전체 파일의 소유권·모드 검사를 구분한다. 공식 매뉴얼에서 `-P`의 기본 모드 강제는 파일 경로를 지정하지 않을 때 기본 활성이다. 이번 기본 절차는 `visudo -P -f`로 명시해 편집 시 기본 sudoers 모드도 맞춘다. 같은 도구에서 편집과 모드 처리를 함께 하여 빠뜨릴 단계를 줄이려는 선택이다. 파일을 지정하지 않는 `visudo -c`는 포함 파일의 문법과 소유권·모드를 함께 검사한다. `Including other files from within sudoers`의 파일명 규칙과 `I/O LOGGING`도 해당 부분만 읽는다.

**설명할 수 있어야 할 것:** 실행 사용자·대상 사용자·허용 명령, TTY, 명령 검색 경로, 명령 기록과 입출력 기록의 차이다. 이들은 원문이 직접 요구하는 제한과 로그를 검증하는 데 필요하다. 문법 검사 성공과 실패 시도·메시지·로그 동작 성공은 따로 확인한다.

**로그 검증에서 읽을 범위:** [공식 sudoreplay 매뉴얼](https://github.com/sudo-project/sudo/blob/main/docs/sudoreplay.man.in)과 설치된 `man sudoreplay`의 `DESCRIPTION`에서 ID·TSID 관계, `-d`의 디렉터리 지정, `-l`의 세션 목록을 읽는다. `-f`의 쉼표로 구분한 스트림 선택과 `stdin`·`ttyin`을 읽는다. 기본 재생 대상은 출력 스트림이며 입력은 별도로 선택한다. 고급 검색식·속도 조절은 로그가 많아지거나 긴 세션을 조사할 때 읽는다.

**이번에 미룰 것과 이유:** 중앙 로그 서버·LDAP sudo 규칙·RHEL system roles는 단일 VM의 로컬 정책에 필요하지 않다. 여러 서버에 같은 권한을 배포하거나 로그를 외부 보존할 때 돌아온다.

### 현재 정책부터 읽기

전체 목표를 세 묶음으로 나눈다. **권한 허용**은 누가 어느 호스트에서 어떤 사용자 권한으로 어떤 명령을 실행할 수 있는지, **실행 조건**은 실패 3회·오류 메시지·TTY·제한된 경로, **기록**은 입력·출력을 `/var/log/sudo/`에 남기는 것이다. 첫 묶음의 기존 정책을 읽은 뒤 실행 조건과 기록을 구성한다.

`sudo -l`로 적용되는 허용 명령과 Defaults를 보고, sudoers 본체와 포함 파일에서 해당 규칙의 근거를 찾는다. 그룹 가입 자체가 권한을 주는 것은 아니며 적용되는 sudoers 규칙이 결정한다. `%wheel`처럼 그룹을 가리키는 활성 규칙에서 **누가 → 어느 호스트 → 어떤 사용자로 → 어떤 명령**을 읽는다. 주석과 활성 규칙, 암호를 요구하는 규칙과 NOPASSWD 규칙을 구분한다.

본체의 `@include`·`@includedir` 또는 이전 표기 `#include`·`#includedir`도 찾는다. 후자의 `#`를 일반 주석으로 넘기지 않는다. 디렉터리에 파일이 있다는 것과 정책으로 읽힌다는 것은 다르므로 include 지시와 목록을 연결한다. `Defaults`는 실행 조건·동작 옵션을 정하며 허용 명령 규칙과 구분한다. `%wheel ALL=(ALL) ALL`은 wheel 구성원 → 모든 호스트 → 모든 대상 사용자 → 모든 명령이라는 네 부분으로 읽는다. 기존 `secure_path`는 sudo가 사용하는 PATH이며 절대 경로로 지정한 명령을 막는 허용 목록이 아니다. 이 실습의 경로 선택은 `/sbin:/bin:/usr/sbin:/usr/bin`이다. 기본 시스템 관리·실행 파일 디렉터리로 검색 범위를 한정한다. 필요한 권한 규칙과 이 경로가 이미 있으면 중복 추가하지 않는다.

**sudo 필수 목표:** 인증 실패 시 시도 3회 제한, 직접 정한 오류 메시지, 작업의 입력·출력 모두 `/var/log/sudo/` 아래 기록, TTY 모드 활성화, sudo에서 사용 가능한 경로 제한(`secure_path` 관련 개념). 오류 문구는 아래 간결한 문장을 선택했고, 경로는 위 네 디렉터리를 기준으로 기존 설정과 대조한다.

`sudo visudo -c`로 문법, `sudo -l`로 적용 정책을 살핀다. 별도로 실패 시도 제한·메시지·TTY 요구를 확인하고 직접 수행한 sudo 작업의 입출력 로그와 대조한다. 문법 통과만으로 동작 검증을 대신하지 않는다.

### 별도 sudo 파일 작성 안내

`sudo visudo -P -f /etc/sudoers.d/90-b2br`로 아래 옵션을 작성한다. `-P`는 편집 모드에서 파일 모드를 기본 sudoers 모드로 설정한다. 파일 내용은 [치트시트](cheatsheet.md#8-sudo-실행-조건과-입출력-로그-설정)에 둔다. `includedir`는 이름에 점이 들어간 파일을 건너뛰므로 `.conf` 확장자를 붙이지 않는다. 근거는 위 공식 sudoers·visudo 매뉴얼의 해당 옵션과 include 설명이다.

| 설정 | 선택 이유 |
|---|---|
| `passwd_tries=3` | 비밀번호 인증 실패 시 최대 3회 시도를 명시한다. |
| `badpass_message` | `Authentication failed. Please try again.`을 선택했다. 과제 고정 문구가 아닌 우리 선택이다. |
| `requiretty` | 호출 사용자의 TTY를 요구한다. 명령을 별도 가상 터미널에서 실행하는 `use_pty`와 다르다. |
| `log_input, log_output` | 입력과 출력을 모두 기록한다. |
| `iolog_dir="/var/log/sudo"` | 과제의 입출력 로그 경로다. 필요한 로그 디렉터리는 sudo가 생성하므로 별도 mkdir 단계는 두지 않는다. |

기본 재실습 흐름은 `sudo visudo -P -f /etc/sudoers.d/90-b2br` → 저장 → `sudo visudo -c` → `sudo -l`이다. 본체와 별도 파일 모두 `parsed OK`이고 오류가 없어야 전체 검사 통과다. 적용 목록 확인과 실패 횟수·TTY 거절·로그 동작 검증은 구분한다. 입출력 로그는 공개 Git에 올리지 않는다. 실제 검사 결과와 복구 이력은 [학습 기록](learning-log.md)에 둔다.

### 로그 생성과 출력 재생 확인

전체 검사 다음에는 비밀이 아닌 문장으로 기록을 만들어 본다. `sudo cat`을 실행하고 `b2br-log-test`를 입력해 Enter를 누르면 cat이 입력을 그대로 출력한다. 빈 줄에서 Ctrl+D로 종료한다. 이어 `sudo sudoreplay -d /var/log/sudo -l`로 세션 목록을 보고 cat 명령의 TSID를 찾는다. 재생 명령의 ID에는 그 실제 값을 넣는다. `-d`가 필요한 이유는 우리가 기본 경로 대신 `/var/log/sudo`를 선택했기 때문이다.

재생에서 문장을 확인하면 해당 기록 생성과 출력 재생을 확인한 것이다. 목록만 보거나 기본 재생을 했다고 모든 입력 스트림까지 검증한 것으로 기록하지 않는다. 문장이 두 번 보이더라도 터미널 에코와 cat 출력일 수 있으므로 입력 스트림의 독립적인 증거로 보지 않는다. `stdin`은 표준입력, `ttyin`은 터미널 입력으로 기록된 스트림이다. 같은 세션에 `-f stdin,ttyin`을 지정해 두 입력 종류를 함께 선택하고 테스트 문장을 대조한다. 문장이 보이면 선택한 입력 중에 기록된 증거지만 두 종류 각각의 성공을 뜻하지는 않는다. 재실습은 [치트시트](cheatsheet.md#9-로그-생성과-출력-재생), 실제 관찰은 [학습 기록](learning-log.md)에 둔다. 이후 인증 실패 횟수와 TTY 조건을 검증한다.

### TTY 실행 조건 확인

**읽을 범위:** VM의 `man sudoers`에서 `requiretty`·`use_pty`, [공식 sudo 매뉴얼](https://raw.githubusercontent.com/sudo-project/sudo/main/docs/sudo.man.in)과 `man sudo`에서 `-n`, [util-linux 공식 setsid 매뉴얼](https://raw.githubusercontent.com/util-linux/util-linux/master/sys-utils/setsid.1.adoc)과 `man setsid`에서 `DESCRIPTION`·`-w`·`-c`를 읽는다. [Linux setsid(2) 매뉴얼](https://man7.org/linux/man-pages/man2/setsid.2.html)의 `DESCRIPTION`에서 새 세션에는 처음에 제어 터미널이 없다는 점까지만 확인한다. 세션·프로세스 그룹의 내부 구현은 작업 제어를 학습할 때로 미룬다.

`requiretty`는 sudo 호출자에게 TTY를 요구한다. 명령 실행용 PTY를 만드는 `use_pty`와 목적이 다르다. 일반 사용자 셸에서 `setsid -w sudo -n true </dev/null`로 제어 터미널 없는 새 세션에서 sudo를 호출한다. `-c`를 넣으면 제어 터미널을 다시 설정하므로 사용하지 않는다. `-w`는 실행 종료를 기다리고 해당 종료 상태를 전달한다. 표준입력은 `/dev/null`로 연결하고 sudo의 `-n`은 비밀번호 등 입력 요청을 막는다. `true`는 실행되더라도 별도 작업을 하지 않는 검사 대상이다.

**판단 기준:** TTY가 필요하다는 명시적 거절로 requiretty 동작을 확인한다. 0이 아닌 종료 상태는 보조 증거이며, 명시적인 거절을 확인했다면 종료 상태만을 위해 재검사하지 않는다. 메시지는 설치 버전·언어에 따라 다를 수 있다. `a password is required`만 나오거나 다른 오류로 실패한 경우에는 TTY 조건 검증 성공으로 판단하지 않는다. 출력이 계속 화면에 보이는 것과 호출 세션에 제어 터미널이 있는 것은 별개다. 재실습 명령은 [치트시트](cheatsheet.md#10-tty-실행-조건-검증)에 둔다.

### 인증 경로와 실패 횟수 확인

**읽을 범위:** 위 공식 sudo 매뉴얼과 설치된 `man sudo`의 `-k`·`-v`, sudoers 매뉴얼의 `passwd_tries`·`badpass_message`를 읽는다. [Linux-PAM 공식 pam_faillock 매뉴얼](https://raw.githubusercontent.com/linux-pam/linux-pam/master/modules/pam_faillock/pam_faillock.8.xml)은 `DESCRIPTION`, `preauth`·`authfail`·`authsucc`, `auth`·`account` 연결과 설정 우선순위를 본다. 모듈이 연결되어 있으면 [faillock.conf 매뉴얼](https://raw.githubusercontent.com/linux-pam/linux-pam/master/modules/pam_faillock/faillock.conf.5.xml)과 설치된 `man faillock.conf`의 `deny`·`fail_interval`·`unlock_time`, `man faillock`의 실패 기록 조회도 읽는다. PAM 전체 설정이나 잠금 정책 변경은 이 검사의 범위가 아니다.

`passwd_tries=3`은 한 번의 sudo 인증에서 실패를 기록하고 종료하기 전까지 허용하는 시도 수다. `badpass_message`는 잘못된 비밀번호에 대한 문구이며 마지막 실패 횟수 요약과 구분한다. PAM의 계정 잠금은 여러 인증에 걸친 실패 기록을 다루므로 이 제한과 별개다.

`cat /etc/pam.d/sudo`로 시작해 `auth`·`account`의 `include`·`substack`이 가리키는 파일을 따라 읽는다. 참조가 `system-auth`일 때 해당 파일을 조회하고, 그 안의 추가 참조도 확인한다. 비밀번호 변경용 `password` 스택이나 authselect 프로필 이름만으로 sudo의 인증 경로를 판단하지 않는다. `pam_faillock.so` 등 잠금 처리가 있으면 모듈 인수와 `conf=` 지정 파일(기본 `/etc/security/faillock.conf`), 현재 실패 기록을 함께 확인한다. 모듈 인수가 설정 파일보다 우선한다. 임계값·집계 시간·해제 시간과 기존 실패 수를 확인하기 전에는 고의 실패를 시도하지 않는다. 검사 때문에 잠금 정책을 끄거나 기록을 초기화하지 않는다.

잠금 위험을 해소하고 비밀번호 인증이 필요한 일반 사용자 터미널에서 `sudo -k`로 현재 세션의 인증 캐시를 무효화한 뒤 `sudo -v`를 실행한다. `-v`는 명령을 실행하지 않고 필요한 인증을 수행한다. 한 번의 호출에서 의도적으로 틀린 값을 세 번 입력해 재시도 중 지정 오류 문구, 세 번째 실패 뒤 네 번째 요청 없이 종료되는지를 확인한다. 입력 프롬프트가 없거나 조기 종료·계정 잠금 메시지가 나오면 해당 결과로는 3회 제한을 검증할 수 없다. 반복 호출하지 않고 정책·PAM 경로를 확인한다. 실제 비밀번호나 입력한 값은 학습 기록에 남기지 않는다. [재실습 절차](cheatsheet.md#11-sudo-인증-경로와-실패-검사).

## 5. SSH·firewalld·SELinux

**반드시 읽을 범위 — 이 구간에 들어갈 때:** [RHEL 10 OpenSSH](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/securing_networks/using-secure-communications-between-two-systems-with-openssh) 1.1 `SSH and OpenSSH`와 1.5.3 `Setting a non-default port for SSH`를 읽는다. [Rocky firewalld for Beginners](https://docs.rockylinux.org/guides/security/firewalld-beginners/)는 `Basic Usage`, `Saving your changes`, `Managing Zones`, `Managing Ports`, `Managing Services`까지 읽는다. [RHEL 10 SELinux 입문](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/getting-started-with-selinux)은 1.1–1.3과 1.5 `SELinux states and modes`를 읽는다. 원문 근거는 7–8쪽이다.

**설명할 수 있어야 할 것:** sshd의 리스닝·로그인 허용, 방화벽의 zone·서비스·포트와 실행/영구 설정, SELinux의 라벨·포트 유형·허용 정책은 서로 다른 층이다. 4242 일반 사용자 접속과 root 거절을 각각 확인해야 하므로 어느 층에서 막혔는지 구분한다. 설정 시 VM의 `man sshd_config`에서 `Port`, `PermitRootLogin`, `Include`, `Match`, `man firewall-cmd`에서 `--permanent`, `--reload`, 조회 옵션, `man semanage-port`에서 포트 유형을 읽는다. OpenSSH의 [공식 참조](https://man.openbsd.org/sshd_config)는 개념 대조용이며 설치 버전의 옵션·최종 적용값을 우선한다.

**시작 조회의 목적:** sshd는 SSH 접속과 로그인을 처리하고, firewalld는 네트워크 트래픽의 허용 범위를 정하며, SELinux는 sshd 같은 프로세스가 접근할 자원·포트를 정책으로 제한한다. 먼저 서비스의 현재 실행 상태와 부팅 시 활성화 설정을 구분하고, SELinux 현재 모드와 VM 인터페이스·주소를 확인한다. 이 조회만으로 4242 리스닝·외부 접속·root 거절을 완료 처리하지 않는다. 결과를 바탕으로 설정과 실제 접속을 차례로 확인한다. [시작 조회와 결과 읽기](cheatsheet.md#12-sshfirewalldselinux-시작-조회).

**SSH 설정을 읽는 순서:** 위 공식 `sshd_config`의 `DESCRIPTION`, `Include`, `Port`, `Match`를 설치된 `man sshd_config`와 대조한다. 본체를 위에서부터 읽고 `Include`가 놓인 위치에서 지정 파일을 따라간다. 와일드카드에 맞는 파일은 사전식 순서로 처리되므로 파일 이름뿐 아니라 본체의 Include 앞뒤 설정과 `Match` 문맥을 함께 본다. 디렉터리에 파일이 있다는 것만으로 읽힌다고 판단하지 않는다.

별도 예외가 명시되지 않은 옵션은 일반적으로 먼저 얻은 값을 사용한다. 따라서 뒤쪽 파일의 값이 앞의 값을 덮어쓴다고 가정하지 않는다. `Match`의 조건부 설정은 접속 조건을 함께 보아야 한다. `Port`는 여러 번 지정할 수 있는 예외이므로 활성 `Port 22`가 있는 상태에 `Port 4242`를 추가해도 22를 대체했다고 볼 수 없다. 주석인 `#Port 22`와 활성 설정을 구분하고, 이후 유효 설정과 실제 리스닝을 대조한다. 이 단계에서는 기존 파일과 포함 경로를 먼저 읽고 편집 위치를 정한다. [조회 절차](cheatsheet.md#13-ssh-설정-파일-조회).

**별도 파일 선택과 검사:** 로컬 과제 설정은 `/etc/ssh/sshd_config.d/00-b2br.conf`에 `Port 4242`와 `PermitRootLogin no`만 둔다. `00`은 `40-`·`50-`으로 시작하는 배포판 파일보다 먼저 읽히도록 정한 이름이다. 일반적인 최초 값 우선 규칙을 따르되 본체의 Include 위치와 앞선 설정도 확인한다. 기존 암호 정책·PAM 설정 파일을 수정하지 않고 과제의 두 설정을 분리하려는 선택이며, `Port`의 복수 지정 예외는 유효 설정 검사로 확인한다.

[공식 sshd 매뉴얼](https://man.openbsd.org/sshd)과 설치된 `man sshd`에서 `-t`·`-T`를 읽는다. `-t`는 설정 유효성과 호스트 키의 건전성을 검사하고, `-T`는 검사 후 유효 설정을 출력하고 종료한다. `Match` 조건별 판단이 필요하면 `-C`의 접속 조건 지정 부분을 읽는다. 출력에서 `port 4242`·`permitrootlogin no`를 대조하되, 이 결과는 실행 중 서비스의 리스닝이나 실제 접속 성공을 증명하지 않는다. 설정 검사 후 SELinux에서 4242의 SSH 사용을 허용할 준비를 마치고 방화벽·접속 경로를 점검한 다음 서비스를 반영한다. 그 전에는 재시작하지 않는다. [작성·검사 절차](cheatsheet.md#14-별도-ssh-설정-작성과-검사).

**SELinux에 SSH 포트 등록:** 위 RHEL 10 OpenSSH 1.5.3 `Setting a non-default port for SSH`의 패키지 전제조건과 SELinux 등록 → firewalld 구성 → 서비스 재시작 순서를 읽는다. `policycoreutils-python-utils`는 `semanage` 관리 도구를 제공한다. [semanage-port 매뉴얼](https://man7.org/linux/man-pages/man8/semanage-port.8.html)과 설치된 `man semanage-port`의 `DESCRIPTION`, `-a`·`-t`·`-p`·`-l`, SSH 예시를 대조한다.

TCP 4242를 `ssh_port_t`에 등록하면 SELinux 정책상 sshd가 사용할 수 있는 포트 유형에 연결된다. `-a`는 항목 추가, `-t`는 SELinux 유형, `-p`는 프로토콜, `-l`은 목록 조회다. 이는 방화벽의 트래픽 허용이나 sshd의 포트 변경을 대신하지 않는다. 목록의 `ssh_port_t` TCP 항목에 4242가 포함되는지 확인한다. 이미 정의된 포트라는 오류가 나면 기존 유형·범위를 먼저 확인하고 임의로 `-m`으로 덮어쓰지 않는다. 이미 같은 유형이면 재등록할 필요가 없고, 다른 유형이면 사용 목적을 확인한 뒤 판단한다. 이후 firewalld 현황을 확인한다. [등록·확인 절차](cheatsheet.md#15-selinux에-ssh-포트-등록).

**firewalld 현황 읽기:** zone은 인터페이스나 출처에 연결하는 규칙 묶음이다. 위 Rocky 입문서의 zone·서비스·포트 부분과 [공식 firewall-cmd 매뉴얼](https://firewalld.org/documentation/man-pages/firewall-cmd.html)의 `Zone Options`, `Options to Adapt and Query Zones and Policies`에서 `--get-active-zones`, `--list-all`, `--zone`을 읽는다. 설치된 `man firewall-cmd`에서도 같은 옵션을 확인한다.

`--get-active-zones`는 현재 활성 zone과 연결된 인터페이스·출처를 보여 준다. `--list-all`은 zone을 지정하지 않으면 기본 zone을 조회하므로 두 출력의 zone 이름과 대상 인터페이스를 대조한다. 기본 zone이 대상 인터페이스의 활성 zone과 다르면 실제 zone을 지정해 읽는다. `--permanent` 없는 조회는 현재 실행 구성이며, 서비스 이름으로 허용된 항목도 있으므로 `ports`만으로 개방 범위를 판단하지 않는다. `services`·`ports`·출처·추가 규칙을 읽고 변경 대상을 정한다. [정상 조회 절차](cheatsheet.md#16-firewalld-활성-zone과-규칙-조회).

**방화벽 변경과 서비스 반영:** 위 공식 firewall-cmd 매뉴얼의 `--permanent`, `--add-port`, `--remove-service`, `--reload`와 Rocky 입문서의 `Saving your changes`를 읽는다. permanent 변경은 저장 구성이며 reload로 실행 구성에 반영된다. 이 실습은 IPv4 접속을 기준으로 대상 zone에 TCP 4242를 허용하고 불필요한 `ssh`·`cockpit`·`dhcpv6-client` 서비스 허용을 제거한다. DHCPv6를 사용하는 환경에서는 `dhcpv6-client` 제거를 그대로 적용하지 않는다. 서비스 허용 제거는 해당 프로그램 삭제·중지를 뜻하지 않는다.

대상 zone의 runtime·permanent 모두 `services`가 비고 `ports`가 `4242/tcp`인지 대조하되 추가 규칙도 함께 확인한다. runtime의 인터페이스 연결과 permanent의 명시적 인터페이스 항목은 구분한다. permanent 칸이 비었다는 이유만으로 연결 실패라고 단정하거나 인터페이스를 추가하지 않고, 부팅 후 활성 zone도 확인한다.

SSH 설정 검사·SELinux 등록·방화벽 준비 후 sshd를 재시작하고 실행 상태와 실제 리스닝을 확인한다. 설치된 `man systemctl`의 `restart`·`is-active`, `man ss`의 `-l`·`-n`·`-t`·`-p`를 읽는다. 목표는 `active`, sshd의 TCP 4242 리스닝, TCP 22 리스닝 없음이다. IPv4와 IPv6 리스닝이 각각 한 줄씩 표시될 수 있다. 이 결과와 호스트에서의 일반 사용자 접속·root 거절은 별도 검증이며, VirtualBox 네트워크 방식과 필요한 포트 전달도 실제 설정으로 확인한다. [반복 절차](cheatsheet.md#17-firewalld-규칙-변경과-대조).

**호스트에서 VM으로 접속:** [VirtualBox 7.2 공식 네트워크 문서](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/networkingdetails.html)의 `Network Address Translation (NAT)`와 `Configuring Port Forwarding with NAT`를 읽는다. 먼저 실제 어댑터 연결 방식을 확인한다. 일반 NAT라면 호스트 `127.0.0.1:4242`를 게스트 TCP 4242로 전달하는 로컬 접속 규칙을 선택한다. Host IP는 호스트에서 받을 주소이고 Guest Port는 전달 목적지 포트이므로 반드시 지정한다. 내장 DHCP로 주소를 받는 일반 NAT에서는 Guest IP를 생략할 수 있어 빈칸을 기본으로 삼는다. 내장 DHCP가 아닌 수동 고정 IP라면 실제 Guest IP를 지정한다. VM의 주소나 포트 전달 창만으로 NAT·DHCP 사용을 확정하지 않는다.

처음 접속할 때 서버 호스트 키 지문을 VM 콘솔의 공개키 지문과 대조한다. [공식 ssh-keygen 매뉴얼](https://man.openbsd.org/ssh-keygen)의 `-l`·`-f`와 설치된 매뉴얼을 읽는다. ED25519를 제시하면 `/etc/ssh/ssh_host_ed25519_key.pub`를 비교하고, 다른 알고리즘이면 해당 호스트 공개키를 비교한다. 같은 알고리즘의 지문이 일치한 뒤 신뢰를 승인한다. 호스트 키 확인은 서버 식별이고 이후 비밀번호 입력은 사용자 인증이다. 로그인 뒤 `hostname`·`whoami`로 목적지와 사용자를 대조한다. [전달 값과 접속 절차](cheatsheet.md#19-virtualbox-포트-전달과-호스트-접속).

**과제 포트와 호스트 전달 포트:** 원문 v5.2 인쇄 8쪽(PDF 9쪽)은 “An SSH service must be running on port 4242 in your virtual machine.”라고 명시하고 방화벽 조건도 VM 내부 포트 4242를 대상으로 한다. NAT의 호스트 IP·포트는 지정하지 않는다. 따라서 호스트 `127.0.0.1:14242` 또는 `127.0.0.2:4242`에서 VM의 4242로 전달하는 구성은 모두 이 포트 요구에 부합한다. 선택 기준은 호스트의 주소·포트 사용 가능 여부와 설명 가능한 접속 경로이며, 호스트 포트까지 4242라는 이유로 과제 적합성이 더 높아지는 것은 아니다.

**호스트 입구의 식별:** 같은 TCP 포트 번호라도 로컬 IP 주소와 함께 바인딩 대상을 구분한다. [Linux ip(7)](https://man7.org/linux/man-pages/man7/ip.7.html)의 `DESCRIPTION`·`Address format`·`Special and reserved addresses`에서 주소·포트 조합, 모든 주소를 뜻하는 `0.0.0.0`, Linux의 `127/8` 루프백 범위를 읽는다. 위 Oracle NAT 문서는 Host IP로 수신 주소를 지정하는 근거다. 기존 리스너가 `127.0.0.1:4242`에만 바인딩했다면 `127.0.0.2:4242`와 공존할 가능성이 있으나 다른 리스너·환경과 실제 전달 성공을 확인해야 한다. 같은 `127.0.0.1:4242`를 사용하려면 기존 점유를 조정해야 하며 학교 관리 서비스라면 관리자와 조율한다. 포트 재사용 옵션·소켓 내부 구현은 이번 범위에서 미룬다.

**root 접속 거절 확인:** 일반 사용자로 목적지·로그인 계정을 확인한 뒤 호스트로 돌아와 같은 전달 포트로 root 접속을 검사한다. [공식 ssh_config 매뉴얼](https://man.openbsd.org/ssh_config)의 `NumberOfPasswordPrompts`를 읽는다. 값을 1로 지정하는 것은 클라이언트 질문을 제한하기 위한 선택이며 `PermitRootLogin no`의 서버 정책과 구분한다. 올바른 root 비밀번호로도 인증이 거절되고 root 셸이 열리지 않는지 확인한다. 단순 연결 장애를 인증 거절의 증거로 삼지 않는다. [검사 절차](cheatsheet.md#20-root-ssh-접속-거절-확인).

**존재와 용도만:** SSH 공개키 인증은 비밀번호 외의 인증 방식이고 SELinux 거절 로그는 정책 문제를 추적하는 단서다. 공개키 방식이 필요하거나 정상 설정 후에도 거절될 때 해당 부분을 더 읽는다. Enforcing·Permissive·Disabled의 차이 자체는 필수 이해 범위다.

**이번에 미룰 것과 이유:** SSH 암호 알고리즘 수학·스마트카드·점프 호스트, nftables 내부 규칙 생성, 자체 SELinux 정책 모듈 작성은 이번 접속 경로의 요구가 아니다. 인증 인프라 확장이나 기존 정책으로 해결할 수 없는 서비스 운영 때 학습한다. firewalld·SELinux를 끄는 것을 범위 축소로 삼지 않는다.

| 과제 필수 목표 | 확인 명령·행동 |
|---|---|
| SSH가 VM의 4242번 포트에서 실행, root SSH 로그인 불가 | `sudo ss -lntp`, `sudo sshd -T`. 조건별 설정도 확인하고 외부에서 일반 사용자 접속 성공·root 거절을 직접 비교한다. 평가 때 만든 새 계정으로도 접속할 수 있어야 한다. |
| Rocky는 firewalld 사용, VM 부팅 시 활성, 필수 구성에서 4242만 개방 | `systemctl is-active firewalld`, `systemctl is-enabled firewalld`, `sudo firewall-cmd --get-active-zones`, `sudo firewall-cmd --list-all-zones`, `sudo firewall-cmd --permanent --list-all-zones` |
| SELinux가 부팅 시 실행되며 과제에 맞게 구성 | `sestatus`, 재부팅 후 상태와 SSH 등 필요한 기능의 동작 확인 |

방화벽은 포트 목록뿐 아니라 활성 zone의 서비스·규칙·허용 범위도 본다. 기본 `ssh` 서비스가 22번을 열고 있지 않은지 확인한다. 현재 실행 설정과 영구 설정을 구분한다. SSH 설정·리스닝·실제 접속은 각각 확인하며 호스트의 포트 전달과 VM 내부 4242도 구분한다. 원문은 SELinux의 부팅 시 실행과 과제에 맞는 구성을 요구한다. 이번 실습은 `getenforce`·`sestatus`로 부팅 후 Enforcing 상태와 로드된 정책을 확인하고, 필요한 SSH 동작이 허용되는지 검증하는 것을 목표로 한다. 포트 유형·라벨·거절 로그를 읽고, 비활성화나 Permissive 상태로 문제를 덮은 것을 완료로 기록하지 않는다.

## 6. monitoring.sh와 cron·wall

**전체 흐름과 첫 단위:** 시스템 상태 수집 → 요구 형식으로 출력·wall 방송 → 부팅 시점과 10분 주기의 cron 예약 순서로 구성한다. 첫 단위는 원문 10쪽의 OS 아키텍처·커널 버전·물리 및 가상 프로세서 수다. VM에서 원자료를 읽고 각 값의 뜻을 설명한 뒤 해당 부분만 코드로 옮긴다.

[GNU uname 매뉴얼](https://man7.org/linux/man-pages/man1/uname.1.html)과 설치된 `man uname`의 `-a`, `-s`, `-r`, `-v`, `-m`, `-o`를 읽어 커널 이름·릴리스·빌드 버전·머신 아키텍처·OS 표기를 구분한다. `uname -a`는 배포판의 제품명·릴리스를 확인하는 명령과 다르다. [util-linux lscpu 매뉴얼](https://man7.org/linux/man-pages/man1/lscpu.1.html)의 `DESCRIPTION` 중 정보 출처와 가상 환경 설명, `NOTES`의 출력 형식 주의를 읽는다. VM 출력의 `CPU(s)`는 논리 CPU 수, `Socket(s)`는 게스트에 보이는 소켓 수로 읽고 물리 프로세서·가상 CPU 지표의 근거를 구분한다. 필요하면 cores/threads와 온라인 CPU 목록을 함께 보되 VM 토폴로지를 호스트의 실제 하드웨어 수로 해석하지 않는다. 지금은 출력 이해가 목표이며 NUMA·캐시·명령어 플래그와 전체 스크립트 구현은 뒤로 미룬다. [첫 조회 절차](cheatsheet.md#21-monitoring-첫-조회-os와-cpu).

**계산용 CPU 출력:** 위 lscpu 매뉴얼의 `-p`·`-b`·`-a`를 읽는다. 기본 요약의 화면 배치는 바뀔 수 있으므로 계산할 때는 `lscpu -p=SOCKET,CPU`로 필요한 열과 순서를 지정한다. `#` 설명 줄을 제외한 행에서 첫 열은 소켓 ID, 둘째 열은 논리 CPU ID다. 소켓 ID의 서로 다른 값 개수와 CPU 데이터 행 수를 각각 세는 원리를 먼저 이해한다. `-p`는 기본적으로 온라인 CPU를 보여 주므로 전체 CPU 수와 대조할 때 오프라인 CPU 유무도 확인한다. 소켓 ID 자체를 개수로 쓰거나 1부터 시작한다고 가정하지 않는다. 과제의 물리 프로세서 지표는 여기서 게스트가 보고하는 소켓 수로 해석하며 `nproc` 명령의 CPU 수와 혼동하지 않는다.

**반드시 읽을 범위 — 이 구간에 들어갈 때:** 원문 10–11쪽에서 먼저 지표와 방송 조건을 읽는다. [Rocky Task Management](https://docs.rockylinux.org/books/admin_guide/11-tasks/)의 `Generalities`, `How the service works`, `Scheduling tasks` 아래 `The crontab command`·`The crontab file`·`Task execution process`로 예약 구조를 잡는다. 실제 작성 시 VM의 `man 5 crontab`에서 시간 필드·환경·`@reboot`, `man wall`의 `DESCRIPTION`을 읽고, 아래 표의 지표를 하나 선택할 때마다 해당 명령의 출력 필드 설명만 읽는다.

**설명할 수 있어야 할 것:** CPU 사용률의 측정 구간과 load average 차이, 사용/가용/전체 메모리·디스크의 분모, 사용자와 세션, VM CPU와 호스트 CPU, 누적 sudo 기록의 집계 범위다. 원문 출력 예시와 문구의 차이를 인식하고 계산 근거를 설명해야 한다. cron의 실행 사용자·환경·부팅 예약과 주기 예약, wall의 수신 대상도 방송 조건을 직접 검증하기 위해 필요하다. 스크립트는 지표 하나씩 원자료와 대조하며 작성한다.

**존재와 용도만:** `/proc`는 커널 정보를 노출하는 경로이고 systemd timer는 다른 예약 방식이다. 명령 출력만으로 지표 정의가 불명확하거나 운영 예약을 확장할 때 찾아본다. 지금 예약 방식을 추가할 필요는 없다.

**이번에 미룰 것과 이유:** 커널 스케줄러 구현, 성능 튜닝, 외부 모니터링 서버와 장기 시계열 저장은 과제의 순간 지표·터미널 방송 요구 밖이다. 병목 분석이나 지속적인 서비스 관측이 목표가 될 때 학습한다. Bash 기본 문법을 일괄 번역하는 대신 실제 필요한 계산·출력만 다룬다.

**필수 동작:** Bash로 작성한 `monitoring.sh`가 서버 시작 시와 이후 10분마다 모든 터미널에 아래 정보를 표시한다. 오류는 표시하지 않으며 배너는 선택이다. 코드를 설명하고 스크립트를 수정하지 않고 실행을 중단할 수 있어야 한다. 원문은 `wall`과 `cron`을 살펴보도록 안내한다.

| 표시할 정보 | 대조할 대상·도구 예시 |
|---|---|
| OS 아키텍처·커널 버전 | `uname -a` |
| 물리 프로세서 수·가상 프로세서 수 | `lscpu`; VM의 토폴로지와 호스트 물리 CPU 수를 혼동하지 않는다. |
| RAM의 사용 가능한 양·사용률 | `free -m`과 계산 기준 |
| 저장 공간의 사용 가능한 양·사용률 | `df -h`와 집계 대상 파일시스템 |
| 현재 CPU 사용률 | 일정 구간의 CPU 통계. load average와 구분 |
| 마지막 재부팅 날짜·시간 | `uptime -s` |
| LVM 활성 여부 | `lsblk`, `sudo lvs` |
| 활성 연결 수 | `ss -tn state established` 등 실제 연결 상태 |
| 서버 사용 사용자 수 | `who`; 세션 수와 사용자 수의 해석 확인 |
| IPv4·MAC 주소 | `ip -4 addr`, `ip link` |
| sudo로 실행한 명령 수 | 실제 sudo 로그와 직접 실행한 작업 대조 |

도구는 **대조용 예시이며 완성 구현이나 계산식이 아니다.** 원문은 RAM·디스크의 “사용 가능한 양”을 쓰지만 출력 예시는 사용량/전체 형식이다. 연결 예시도 TCP ESTABLISHED다. 이 차이는 평가 기준과 함께 확인한다. 임의 수치를 실측 출력처럼 채우지 않는다.

cron의 사용자·환경·경로와 wall의 수신 대상을 구분한다. 사용자 crontab과 시스템 예약 위치를 확인하고 예약 중단을 실제로 시연한다. 부팅 직후 방송, 10분 간격, 여러 터미널 수신, 오류 부재, 값의 근거, 스크립트 수정 없는 중단을 직접 관찰한다. 예약 설정 조회만으로 방송 성공을 판정하지 않는다.

### monitoring 메모리·디스크의 읽기 범위

[procps-ng free 공식 매뉴얼](https://gitlab.com/procps-ng/procps/-/raw/master/man/free.1)의 `DESCRIPTION`에서 total·used·free·shared·buff/cache·available, `OPTIONS`에서 `-m`만 먼저 읽는다. 설치된 VM의 `man free`와 대조한다. free는 완전히 미사용인 메모리이며 available은 새 프로그램에 사용할 수 있는 양의 추정치다. 현재 upstream의 used 정의는 total − available이지만 설치 버전의 정의를 확인하고, 표시 단위로 반올림된 숫자로 정확한 등식을 기대하지 않는다. Swap은 RAM과 따로 읽는다.

[df 매뉴얼](https://man7.org/linux/man-pages/man1/df.1.html)의 `DESCRIPTION`, `-B`·`--total`·`-x`와 단위 설명을 설치된 `man df`와 대조한다. `df -BM --total -x tmpfs -x devtmpfs`는 지정 유형을 제외한 마운트 파일시스템을 MiB 단위로 보여 준다. 어떤 마운트가 합계에 들어가는지 먼저 확인한다. 파일시스템의 Size·Used·Avail·Use%를 구분하고 예약 공간·표시 반올림 때문에 Avail을 단순히 Size − Used로 단정하지 않는다. 파일시스템 합계를 가상 디스크 용량·LVM의 미할당 공간·swap과 혼동하지 않는다.

원문 v5.2 인쇄 10쪽의 요구 문장은 다음과 같다.

> The currently available RAM on your server and its utilization rate as a percentage.
>
> The currently available storage on your server and its utilization rate as a percentage.

같은 쪽의 예시는 다음 두 줄이다.

```text
#Memory Usage: 74/987MB (7.50%)
#Disk Usage: 1009/2Gb (49%)
```

요구 문장은 가용량을 언급하지만 예시는 사용량/전체처럼 표현한다. 원문은 디스크 퍼센트가 used/size인지 `df`의 Use%인지, 파일시스템 집계 범위가 무엇인지 명시하지 않는다. 예시의 `MB`·`Gb` 표기도 그대로 인용한 것이며, 이를 근거로 바이트 환산 기준·분자 단위·반올림 규칙을 임의로 확정하지 않는다. 조회 명령의 MiB 단위와 원문 예시 표기는 별개다. 이 구간에서는 원자료와 정의를 읽고, 계산·출력 선택은 근거를 정리한 뒤 진행한다.

### monitoring 메모리·디스크의 계산과 awk

원문에 계산식이 없으므로 이 부분 구현은 두 지표 모두 **used/total × 100**을 선택한다. 이는 구현의 선택이며 원문의 명시 조건이 아니다. `free -m`과 `df -m`의 정수 출력값을 계산 입력으로 쓴다. 원문 예시의 `MB`·`Gb` 라벨은 유지하지만 실제 메모리와 디스크 used는 MiB, 디스크 total의 `$2 / 1024`는 GiB다. `%.0f`로 전체 용량을 반올림해 표시해도 비율의 분모는 변환 전 `$2`다. 퍼센트는 메모리 소수 둘째 자리, 디스크 정수로 표시한다. 도구 자체의 MiB 표시 정밀도와 printf의 표시 반올림을 구분한다.

[GNU df 매뉴얼](https://www.gnu.org/software/coreutils/manual/html_node/df-invocation.html)의 `-m`·`--total`·`-x`와 `--output` 필드만 읽는다. 웹 페이지 접근이 안 되면 [공식 매뉴얼 원본](https://github.com/coreutils/coreutils/blob/master/doc/coreutils.texi)의 `df invocation`을 찾는다. 매뉴얼은 `pcent`를 used/size로 요약하지만, [실제 src/df.c](https://github.com/coreutils/coreutils/blob/master/src/df.c)의 `PCENT_FIELD` 계산은 일반적인 비음수 값에서 `used / (used + available) × 100`을 올림한다. reserved 공간 등의 영향으로 이 분모는 전체 size와 다를 수 있다. 소스에서는 이 계산 블록만 대조하며 전체 C 구현을 공부할 필요는 없다. 설치된 버전의 매뉴얼·출력도 함께 확인한다.

awk는 이번 두 행을 읽는 데 필요한 범위만 익힌다. 기본적으로 공백으로 나눈 첫 필드가 `$1`이며 `$2`·`$3`는 total·used다. `$1 == "Mem:"` 또는 `$1 == "total"`이라는 조건이 맞는 행에만 `{...}` 동작을 적용한다. `printf`의 `%s`는 문자열, `%.2f`·`%.0f`는 표시 소수 자릿수, `%%`는 문자 `%`다. 작은따옴표는 셸이 `$1` 등을 먼저 확장하지 않도록 awk 프로그램을 감싼다. [GNU awk 매뉴얼](https://www.gnu.org/software/gawk/manual/gawk.html)의 `Fields`·`Patterns`·`Printf` 해당 항목이나 설치된 `man awk`에서 이 요소만 찾는다.

`LC_ALL=C`는 그 명령의 출력 언어·로케일을 고정해 `Mem:`·`total` 같은 행 이름을 비교하기 위한 선택이다. 파이프 앞의 환경 지정은 앞 명령에만 적용되며 뒤의 awk까지 자동으로 전파되지는 않는다. 현재 부분 예시의 소수점 출력은 실행 환경과 대조하고 전체 스크립트 검증 때 로케일 범위도 점검한다.

### monitoring CPU와 부팅 시각의 읽기 범위

[procps-ng vmstat 공식 매뉴얼](https://gitlab.com/procps-ng/procps/-/raw/master/man/vmstat.8)의 `DESCRIPTION`, `delay`·`count`, `FIELD DESCRIPTION FOR VM MODE`의 `CPU`만 읽는다. `vmstat 1 2`의 첫 CPU 표본은 부팅 이후 평균, 둘째는 직전 1초 구간이다. 프로세스·메모리 열은 이 평균 설명과 다르게 순간 상태이므로 구분한다. 기본 출력에서 `id`가 15열인지 헤더와 실제 열을 대조하고 옵션으로 열 구성을 바꾸면 다시 확인한다.

`awk 'END {...}'`는 입력을 다 읽은 뒤 한 번 실행하므로 이 출력에서는 마지막 수치 행의 `$15`를 사용한다. `100 - id`는 idle을 제외한 비율로 `wa`(I/O 대기)와 `st`(VM이 빼앗긴 시간)도 포함한다. 따라서 load average나 순수 연산 실행시간 비율과 같지 않다. `#CPU load`는 출력 라벨이며 측정량의 정의는 이 계산으로 설명한다. `%.1f`는 출력 자릿수일 뿐 원자료의 측정 정밀도를 높이지 않는다.

[procps-ng uptime 공식 매뉴얼](https://gitlab.com/procps-ng/procps/-/raw/master/man/uptime.1)의 `-s`·`--since`는 부팅 시각을 `yyyy-mm-dd HH:MM:SS` 형식으로 출력한다. 설치된 `man uptime`의 해당 옵션과 원자료를 대조한다. `cut -c 1-16`은 앞 16자를 선택해 원문 예시처럼 분까지만 표시한다. 초를 시간 반올림하는 것이 아니라 초 부분을 생략한다. 날짜 형식과 부분 코드의 실행 확인은 서로 다른 도구가 보고하는 시각의 일치 확인과 구분한다. 명령 치환의 결과를 큰따옴표로 감싸 날짜와 시각을 하나의 printf 인자로 전달한다. 설치된 `man cut`에서 `-c`와 위치 범위만 읽는다. 두 부분 예시 모두 명령 실패·빈 출력·CPU 열 변경에 대한 처리는 전체 스크립트 검증에서 다시 다룬다.

### 부팅 시각의 출처와 읽기 깊이

[procps-ng v4.0.4 uptime.c](https://gitlab.com/procps-ng/procps/-/raw/v4.0.4/src/uptime.c)의 `print_uptime_since()`는 현재 시각에서 가동시간을 빼고 현지 시각으로 출력한다. `who -b`는 BOOT_TIME 기록을 읽는 경로다. [systemd v257 update-utmp.c](https://raw.githubusercontent.com/systemd/systemd/v257/src/update-utmp/update-utmp.c)의 `get_startup_monotonic_time()`·`on_reboot()`는 userspace 시작 타임스탬프를 가져와 실시간 시계로 변환해 부팅 기록을 쓰는 흐름을 보여 준다. 이 자료는 두 도구의 출처 차이를 설명하며 특정 VM에서 관측한 차이의 원인을 입증하지 않는다. 참조 소스 버전을 실제 설치 버전으로 간주하지 않는다.

지금은 계산 시각과 저장된 기록의 차이만 설명하고 `uptime -s`의 분 단위 출력을 유지한다. 시계·부팅 타임라인의 상세 추적은 현재 과제 진행에 필요하지 않아 미룬다. 시간 정확성 자체가 목표가 되거나 다른 동작에 영향을 줄 때 설치 버전과 실제 기록을 함께 조사한다.

### monitoring LVM·TCP 연결의 읽기 범위

[util-linux lsblk 매뉴얼](https://man7.org/linux/man-pages/man8/lsblk.8.html)의 `DESCRIPTION`과 `-o`·`--output`만 읽고 설치된 `man lsblk`와 대조한다. `lsblk -o NAME,TYPE`은 장치 이름과 유형을 지정해 트리 관계를 읽기 위한 조회다. TYPE의 `lvm` 존재 여부를 보고, `disk`·`part`·`crypt`와 구분한다. 이것은 현재 보이는 블록 장치의 관측이며 LVM 패키지 설치 여부와는 다르다. 우선 원자료에서 유형을 찾고 yes/no 출력 코드는 이후에 작성한다.

[iproute2 ss 매뉴얼](https://man7.org/linux/man-pages/man8/ss.8.html)의 `DESCRIPTION`, `-H`·`-t`·`-n`, `STATE-FILTER`의 `established`를 읽는다. `ss -Htn state established`는 헤더를 생략하고 TCP의 ESTABLISHED 소켓을 숫자 주소·포트로 조회한다. 출력 행에서 로컬·상대 주소와 포트를 읽고 행 수를 확인한다. 이 수는 현재 관측되는 소켓 수이며 사용자 수·고유 상대 호스트 수·LISTEN 포트 수와 다르다. 조회하는 네트워크 네임스페이스의 범위이며 SSH 등 연결 변화에 따라 달라질 수 있다. 빈 출력은 정상 종료했을 때만 해당 상태의 소켓이 없다는 뜻으로 읽는다. 필터·행 의미를 확인한 뒤 집계 코드를 작성한다.

### monitoring LVM·TCP 부분 구현의 읽기 범위

lsblk 매뉴얼의 `-r`·`--raw`와 `-n`·`--noheadings`를 이어 읽는다. `lsblk -rn -o TYPE`은 트리 장식과 제목 없이 TYPE만 출력해 행 전체를 검사하기 위한 입력을 만든다. [GNU grep 매뉴얼](https://www.gnu.org/software/grep/manual/grep.html)의 `-q`·`-x` 및 종료 상태 설명을 읽는다. `-q`는 일치한 내용을 출력하지 않고 일치 여부를 종료 상태로 알리며 `-x`는 행 전체가 `lvm`인 경우만 일치시킨다.

[GNU Bash 매뉴얼](https://www.gnu.org/software/bash/manual/html_node/Conditional-Constructs.html)의 `if`와 설치된 `man bash`의 `Pipelines`만 대조한다. if는 출력 문자열이 아니라 명령의 종료 상태가 0인지로 분기한다. 기본 파이프라인의 종료 상태는 마지막 명령인 grep의 상태다. 따라서 이 부분 예시는 정상 조회 결과의 존재/부재를 구분하는 단계이며, 조회 실패를 안전하게 no로 판정하는 완성 구현은 아니다. 명령 오류·pipefail 등은 전체 스크립트 검증에서 따로 점검한다.

TCP 쪽은 ss의 `-H`가 제목을 없애고 `-t`가 TCP를 선택하며 `-n`이 숫자 표기를 유지하고 `state established`가 상태를 한정한다. 설치된 `man wc`에서 `-l`의 줄바꿈 개수 의미만 읽는다. 정상적인 ss 출력의 소켓 행을 세는 것으로, 시스템 전체 연결의 양 끝을 하나로 중복 제거하는 계산은 아니다. 두 끝이 같은 관측 범위에 보이면 각각의 소켓 행이 될 수 있다. 사용자 수나 고유 연결 상대 수로 해석하지 않는다.

### monitoring 로그인 세션·IP·MAC의 읽기 범위

[GNU who 매뉴얼](https://man7.org/linux/man-pages/man1/who.1.html)의 `DESCRIPTION`과 설치된 `man who`에서 기본 출력의 사용자·터미널·로그인 시각을 읽는다. 지금은 `who` 원자료에서 같은 사용자명이 여러 터미널에 나타나는지를 확인한다. 로그인 세션 수와 중복을 제외한 사용자 수는 다른 지표이며, 등록된 전체 계정 수나 TCP 소켓 수와도 다르다. 원문이 요구하는 서버 사용 사용자 수를 어떻게 집계할지 원자료를 읽은 뒤 정한다.

[iproute2 ip 매뉴얼](https://man7.org/linux/man-pages/man8/ip.8.html)의 `-br`·`--brief`, `OBJECT`의 `address`·`link`만 먼저 읽고 설치된 `man ip`와 대조한다. `ip -br address`는 인터페이스의 IP 주소를, `ip -br link`는 링크 정보를 간단한 표로 보여 준다. 같은 인터페이스의 IPv4와 MAC을 짝지어 읽고 IPv4의 `/접두사 길이`는 주소 자체와 구분한다. loopback과 외부 접속에 쓰는 인터페이스, IPv4와 IPv6를 구분한다. 여기서는 `enp0s3`를 확인 대상으로 삼지만 이름과 주소를 다른 VM에도 고정값으로 적용하지 않는다. 여러 주소가 있으면 무작정 첫 값을 택하지 않고 대상 인터페이스와 표시 범위를 먼저 정한다. 네트워크 설정 변경이나 전체 라우팅 학습은 이 조회 단위에 필요하지 않다.

### monitoring 사용자수·IPv4·MAC 부분 구현

원문 v5.2 인쇄 10쪽은 “The number of users using the server.”를 요구하며 예시는 `#User log: 1`이다. 세션 행 수를 쓰라고 명시하지 않는다. 이 구현은 who에 나타난 로그인 사용자명을 중복 제거해 세는 해석을 선택한다. 같은 계정의 여러 세션은 한 사용자로 계산하며, 이것은 계정을 공유하는 실제 사람 수를 판별하는 기능이 아니다. 모든 등록 계정을 세는 것도 아니다.

`awk '{print $1}'`은 사용자명 열을 고르고 `sort -u`는 중복을 제거하며 `wc -l`은 남은 행을 센다. 기존 awk·sort·wc 읽기 범위에서 이 동작만 대조한다. [ip 매뉴얼](https://man7.org/linux/man-pages/man8/ip.8.html)의 `-4`(IPv4)·`-o`(한 레코드를 한 줄로), [ip-address 매뉴얼](https://man7.org/linux/man-pages/man8/ip-address.8.html)의 `show dev IFNAME`을 읽는다. 지정 NIC의 출력에서 `$4`가 IPv4/접두사 길이임을 확인한 뒤 사용한다. 설치된 `man cut`의 `-d`(구분자)·`-f`(필드 선택)를 읽고 `/`로 나눈 첫 필드만 고르는 이유를 설명한다.

[Linux 커널 sysfs 네트워크 ABI](https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-class-net)의 `/sys/class/net/<iface>/address` 항목은 현재 인터페이스 링크 주소를 설명한다. Ethernet NIC의 MAC을 이 파일에서 읽어 같은 NIC의 IP와 짝짓는다. 이는 영구 하드웨어 주소임을 보장하는 값은 아니다. 현재 예시는 단일 대상 NIC·단일 IPv4를 전제로 하며, NIC 이름 변경·여러 IPv4·명령 실패와 빈 결과는 전체 검증에서 다시 다룬다.

### sudo 횟수 원자료의 읽기 범위

[systemd journalctl 매뉴얼](https://www.freedesktop.org/software/systemd/man/latest/journalctl.html)의 필드 일치 조회, `-n`·`--lines`, `--no-pager`, `-o cat`과 설치된 `man journalctl`의 같은 항목만 읽는다. `_COMM=sudo`는 해당 명령 이름의 로그 필터이며, `-n 10`은 최근 메시지 10개 제한이다. cat은 메시지 본문 위주로 출력한다. 세션 메시지와 명령 실행 기록이 섞일 수 있어 로그 행 수를 곧바로 sudo 실행 횟수로 삼지 않는다. 기록 보존 범위·실행 기록의 실제 형식을 확인하고 집계 기준을 정한 뒤 구현한다.

### sudo I/O 세션 목록의 읽기 범위

[공식 sudoers 매뉴얼](https://github.com/sudo-project/sudo/blob/main/docs/sudoers.man.in)의 `Accepted command log entries`·`Denied command log entries`와 `log_input`·`log_output`·`iolog_dir`를 읽는다. 명령 기록과 PAM 세션 메시지는 다른 사건이며, `COMMAND`가 있다는 것만으로 허용된 실행이라고 단정할 수 없다. 따라서 journal의 전체 행 수나 COMMAND 단순 검색 결과를 바로 실행 횟수로 삼지 않는다.

이미 설정한 `/var/log/sudo`의 I/O 로그를 재사용하는 방향으로 [공식 sudoreplay 매뉴얼](https://github.com/sudo-project/sudo/blob/main/docs/sudoreplay.man.in)의 `DESCRIPTION`, `-d`·`-l`만 읽고 설치된 `man sudoreplay`와 대조한다. 목록 모드는 보관된 세션을 파일명 또는 시퀀스 번호 순으로 나열하고 TSID로 실행 기록과 연결할 수 있다. 설치된 `man tail`의 `-n`은 끝에서 지정한 줄 수를 읽는 옵션이다. 마지막 다섯 줄은 형식 확인 표본이지 전체 개수나 시간순 정렬을 보장하는 선택이 아니다.

이 방향에서 세려는 대상은 **현재 디렉터리에 보관된 sudo I/O 실행 세션 수**다. 영구 누적 명령 수나 sudo로 연 셸 안에서 실행한 개별 명령 수가 아니다. 로그 삭제·시퀀스 재사용·로깅 예외가 있으면 범위가 달라지며, 세션 존재가 명령의 성공 종료를 뜻하지도 않는다. 조회를 sudo로 수행하면 그 조회도 기록될 수 있다. 실제 목록 형식과 기록 범위를 확인한 뒤 집계 방식을 정하며 아직 집계 코드는 작성하지 않는다.

### monitoring 첫 구현의 흐름

[첫 구현 예시](cheatsheet.md#23-monitoring-첫-구현-예시)는 OS·CPU 세 항목만 다룬다. `#!/bin/bash`는 파일을 직접 실행할 때 사용할 Bash 경로를 지정한다. `architecture=$(uname -a)`의 `$(...)`는 명령의 표준출력을 받아 변수 값으로 쓰며 끝의 줄바꿈은 제거한다.

CPU 계산은 `lscpu`의 열 지정 → `grep -v '^#'`로 설명 줄 제외 → 필요한 집계의 순서다. `|`는 앞 명령의 출력을 다음 명령의 입력으로 전달한다. 소켓 열은 `sort -u`로 같은 ID를 하나로 합친 뒤 `wc -l`로 종류 수를 센다. CPU 열은 논리 CPU마다 한 행이므로 바로 행 수를 센다. 기본 `-p`의 온라인 CPU 범위가 이 계산에도 적용된다. `printf`의 `%s`는 변수 값을 문자열로 넣고 `\n`은 줄바꿈을 출력한다. 변수 확장을 큰따옴표로 감싸 Architecture의 공백 등을 한 인자로 유지한다.

설치된 `man bash`의 `Command Substitution`, `Pipelines`, `printf`, `man grep`의 `-v`, `man sort`의 `-u`, `man wc`의 `-l`만 이 예시와 대조한다. 출력 형식을 먼저 재현하고 다른 지표를 추가한다. 명령 실패 처리와 CPU가 오프라인인 경우의 집계 범위는 전체 스크립트 검증에서 다시 점검한다.

## 7. 평가 재현과 README·서명

**반드시 읽을 범위 — 제출 준비 때:** [원문](../en.subject.pdf)의 인쇄 6쪽, 12–13쪽, 16–17쪽을 함께 읽는다. 일반 지침의 서명 파일 언급만 읽으면 README 요구를 놓친다. Linux에서는 설치된 `man sha1sum`의 파일 해시 출력 설명을 읽고 실제 VM 디스크를 대상으로 한다. 사용 중인 VirtualBox/UTM 도움말에서는 `Snapshots`를 찾아 생성·복원·삭제와 현재 디스크 상태의 관계를 확인한다.

**설명할 수 있어야 할 것:** 디스크 내용이 바뀌면 서명이 바뀐다는 점, VM 파일 Git 제출 금지, 평가 시작 시 스냅샷 없음과 평가용 스냅샷 종료 시 삭제, README의 선택 이유·비교·AI 사용 범위다. 근거는 원문 제출 조건이며, SHA-1 계산 성공만으로 디스크와 제출물이 일치한다고 단정하지 않는다. Debian/Rocky·AppArmor/SELinux·UFW/firewalld·VirtualBox/UTM 비교는 README 필수이므로 생략하지 않는다.

**존재와 용도만:** 전체 복제와 연결 복제는 디스크 보존 방식이 다르다는 정도를 알아둔다. 평가용 보존 방법을 실제로 선택할 때 사용 중인 가상화 도구의 해당 도움말을 더 읽는다.

**이번에 미룰 것과 이유:** SHA-1 알고리즘 구현·충돌 공격 수학, 다른 하이퍼바이저의 내부 구조, 보너스 웹 서비스 구축은 현재 필수 제출·설명 목표가 아니다. 암호학 분석·가상화 개발·보너스를 새 목표로 삼을 때 돌아온다. 비교를 위해 다른 OS 전체를 재설치할 필요는 없다는 것은 학습 범위 판단이며 비교 자체를 면제한다는 뜻이 아니다.

재부팅 후 각 목표를 다시 대조하고 선택 이유를 설명한다. README와 실제 디스크 SHA-1을 담은 `signature.txt`가 최종 제출 대상이다. VM은 Git에 넣지 않는다. 각 평가는 스냅샷 없이 시작하고 평가용 스냅샷은 이후 생성해 종료 시 삭제한다. VM 재실행으로 서명이 달라질 수 있으므로 실제 디스크와 제출 서명을 대조한다. README 등 제출 세부 조건은 기존 가이드에서 확인한다.

README의 첫 줄은 `*This project has been created as part of the 42 curriculum by jungblee.*`를 유지하고 Description·Instructions·Resources, 참고 자료와 AI 사용 작업·범위를 채운다. OS 선택 이유와 장단점, 파티션·보안·사용자·서비스 설계, Debian/Rocky·AppArmor/SELinux·UFW/firewalld·VirtualBox/UTM 비교를 실제 학습에 근거해 정리한다. Debian은 이 비교에 필요한 범위에서 읽는다. 선택한 OS와 도구를 설명하고 원문의 apt/aptitude 질문 예시도 비교 맥락에서 살펴본다. 평가 지침에 포함되면 작은 스크립트·동작 변경도 직접 수행할 준비를 한다.

서명 대상은 VM 설정이나 ISO가 아니라 실제 `.vdi`(UTM은 `.qcow2`)다. 원문은 서명 불일치와 그래픽 서버 설치를 0점 조건으로 명시한다. VirtualBox 사용이 필수이며 사용할 수 없을 때 UTM을 사용한다. README의 설치·실행 설명과 설계 근거는 실제 재현 가능한 구성에 맞춘다.

## 구간별 공식 자료 읽기

전체를 한꺼번에 따라 하지 않고 해당 구간의 개요와 필요한 항목만 읽는다. Rocky 입문서는 버전이 섞일 수 있어 Rocky 10의 설치된 도구·매뉴얼과 RHEL 10 자료를 함께 대조한다. `man`은 실제 VM에 설치된 패키지 버전의 매뉴얼을 읽는 방법이다.

| 구간 | 입문 범위 |
|---|---|
| 1 | [Rocky 10 설치](https://docs.rockylinux.org/guides/installation/), [DNF](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/)의 패키지 조회·저장소·업데이트 개념. `man lsblk`, `man vgs`, `man systemctl` |
| 2 | [Rocky User Management](https://docs.rockylinux.org/books/admin_guide/06-users/)의 그룹·사용자·소유권. `man id`, `man useradd`, `man groupadd`, `man chmod` |
| 3 | [Rocky PAM](https://docs.rockylinux.org/guides/security/pam/)의 모듈 유형과 제어 흐름. `man pam_pwquality`, `man pwquality.conf`, `man chage`, `man authselect` |
| 4 | 설치된 `man sudo`, `man sudoers`의 인증, Defaults, `requiretty`, `secure_path`, 입출력 로깅 항목과 `man sudoreplay` |
| 5 | [OpenSSH sshd](https://man.openbsd.org/sshd.8), `man sshd_config`, [firewalld](https://firewalld.org/documentation/man-pages/firewall-cmd.html)의 조회·runtime/permanent, [RHEL 10 SELinux](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/index)의 시작하기·모드·거절 원인 확인 |
| 6 | [Rocky cron](https://docs.rockylinux.org/guides/automation/cron_jobs_howto/)의 예약 구조, `man 5 crontab`, `man wall`, 각 지표 도구의 매뉴얼 |
| 7 | 원문 인쇄 12–13쪽 README, 16–17쪽 제출·평가. 기존 번역에서 원문 예시와 해설을 구분 |

명령 참고: [firewall-cmd 공식 문서](https://firewalld.org/documentation/man-pages/firewall-cmd.html), [OpenSSH sshd 공식 매뉴얼](https://man.openbsd.org/sshd.8). 설치된 버전의 도움말도 함께 확인한다.
