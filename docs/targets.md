# Born2beRoot — 직접 만들고 대조할 목표

**이 문서는 목표·읽을 범위·확인 방법을 정리한다. 화면으로 확인한 내용은 별도로 표시하며, 읽기 안내 자체는 완료 기록이 아니다.** 한 단위씩 직접 구성하고 관찰한 결과를 [학습 기록](learning-log.md)에 남긴다. 요구사항은 [원문 v5.2](../en.subject.pdf), 자세한 항목은 [기존 가이드](ko/Born2beRoot-v5.2-ko-guide.md)를 기준으로 한다. 사용자는 보너스를 목표로 하지 않기로 했다. 아래는 필수 서버 관리·모니터링 실습에 맞춘 선택이며 보너스 그림을 복제하지 않는다.

## 남은 학습의 전체 지도

설치 후 사용자 화면에서 셸 진입과 디스크 배치·VG 수치를 확인했다. 확인 범위와 남은 검증은 [학습 기록](learning-log.md)에 구분한다. 아래는 계속 대조할 목표다. 아래 순서로 **전체 역할 파악 → 필요한 개념 읽기 → 한 부분 직접 적용 → 결과 확인 → 배운 내용 기록**을 반복한다. 필수 조건의 숫자와 확인 방법은 이어지는 각 구간에 둔다.

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

2026-10-13에 아래 Rocky 공식 가이드와 RHEL **10** 문서의 제목·읽을 절을 확인했다. Rocky 관리 입문서는 릴리스별로 고정된 책이 아니므로 개념에 사용하고, 설치 안내는 제목이 *Installing Rocky Linux 10*인 문서를 사용한다. RHEL 10의 설명도 Rocky의 패키지 빌드·기본값·현재 설정을 대신 증명하지 않는다. 설정 시 VM의 설치 버전과 해당 `man`을 대조한다. 최신 upstream 매뉴얼의 새 옵션을 Rocky 10에 그대로 있다고 가정하지 않는다.

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
- root 10 GiB는 설치 후 사용량을 확인할 초기 배정이다. 용량 선택 이유와 ext4 축소 조건은 학습 기록에 둔다.

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
| 우리 선택 | Rocky Minimal | 설치기 화면의 10.2 표시와 실제 설치·부팅 검증을 구분하고 공식 릴리스 정보와 대조 |
| 과제 필수 + 로그인 대입 | root 외에 `jungblee`, 소속 그룹에 `user42`와 `sudo` 포함 | `id jungblee`, `getent group user42`, `getent group sudo`. Rocky의 `wheel`만으로 대신하지 않는다. |
| 과제 필수 + 로그인 대입 | hostname `jungblee42` | `hostnamectl --static`; 재부팅 뒤에도 확인 |

평가에서는 새 사용자를 만들고 그룹에 넣으며 호스트명을 변경할 수 있어야 한다. 이를 준비하며 사용자·그룹 생성, 기본·보조 그룹, 파일 소유권과 권한을 작은 실습으로 확인한다. hostname 설정은 사용자 보고와 셸 프롬프트 표시를 확인했으며 별도 조회·재부팅 지속성 검증은 남아 있다. Rocky의 Kdump 설정은 과제에서 요구하지 않는다.

## 3. 비밀번호·PAM

**반드시 읽을 범위:** 원문 8–9쪽의 기간·품질·root 예외·기존 비밀번호 갱신 조건을 기준으로 [Rocky User Management](https://docs.rockylinux.org/books/admin_guide/06-users/)의 `passwd command`, `chage command`, `/etc/shadow file`, `/etc/login.defs file`을 읽는다. [Rocky PAM 입문](https://docs.rockylinux.org/guides/security/pam/)은 `Introduction`부터 `Control Indicators`까지 읽어 네 처리 유형과 제어 흐름을 잡는다. 뒤의 `pam_cracklib`·`pam_tally` 예시는 현재 Rocky 10 구성의 설치·설정 지침으로 사용하지 않는다. [RHEL 10 authselect](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_authentication_and_authorization_in_rhel/configuring-user-authentication-using-authselect)에서는 2.1과 2.1.1, 2.2의 **현재 구성 조회 부분만** 읽는다. 프로필 전환 예시는 지금 실행하지 않는다.

**설정할 때 반드시 찾을 항목:** VM의 `man chage`에서 `-l`, `-m`, `-M`, `-W`, `man login.defs`에서 `PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE`를 찾는다. 아래 연결한 libpwquality 공식 매뉴얼의 `DESCRIPTION`과 `minlen`, `dcredit`, `ucredit`, `lcredit`, `ocredit`, `maxrepeat`, `usercheck`, `difok`, `enforce_for_root`, `use_authtok`를 읽고 설치 버전과 대조한다. 값의 의미를 정한 뒤 작성하며, 아직 완성 설정을 붙여 넣는 단계는 아니다.

**설명할 수 있어야 할 것:** 새 계정 기본값과 기존 계정 기간 값, 품질 검사와 저장, `passwd` → `system-auth`의 호출 관계, `requisite` 실패와 `use_authtok` 전달이다. root의 이전 비밀번호 비교 예외와 다른 품질 조건 적용도 구분한다. 특히 **login.defs/libeconf는 기본 파일 뒤 drop-in, libpwquality는 drop-in 뒤 본체이며 PAM 인수가 우선**이라는 차이는 반드시 알아야 한다. 아래 공식 근거처럼 실제 적용 값이 달라지므로 파일을 만들었다는 사실만으로 정책 적용을 판단할 수 없기 때문이다. 2/30/7과 마지막 변경일·실제 만료일도 함께 대조한다.

**존재와 용도만:** `auth`, `account`, `session`은 각각 인증·계정 허용·세션 처리를 맡고, authselect에는 사용자 정의 프로필이 있다. 로그인 실패 원인을 분리하거나 기존 프로필로 표현 못 하는 요구가 생길 때 더 읽는다.

**이번에 미룰 것과 이유:** PAM 모듈 C 구현, yescrypt 계산 내부, MFA·원격 인증 설계는 현재 연결된 품질 모듈에 과제 규칙을 적용하는 데 필요하지 않다. 자체 인증 개발이나 해시·인증 방식 변경 때 학습한다. shadow 소스는 문서만으로 불명확했던 drop-in 지원 빌드를 확인한 보조 근거이며 필독 소스 분석 과제가 아니다.

PAM은 서비스가 인증·계정·비밀번호·세션 처리를 연결하는 틀이다. 비밀번호 품질 검사와 계정별 만료 정보는 따로 확인한다. Rocky의 authselect 관리 여부와 실제 PAM 경로를 파악하고, 기본값 변경만으로 기존 계정도 바뀌었다고 가정하지 않는다.

`passwd`로 비밀번호를 바꿀 때는 PAM의 `password` 스택을 거친다. `pam_pwquality.so`는 새 비밀번호의 품질을 검사하고, 통과한 비밀번호를 뒤의 모듈에 전달해 실제 갱신을 이어 간다. 품질 검사와 저장을 같은 역할로 보지 않는다. 근거: [libpwquality의 pam_pwquality 매뉴얼](https://github.com/libpwquality/libpwquality/blob/master/doc/man/pam_pwquality.8.pod).

authselect가 관리하는 PAM 파일은 곧바로 편집하기보다 현재 프로필과 구성 상태, 실제 `password` 연결을 먼저 읽는다. [RHEL 10 authselect 공식 문서](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_authentication_and_authorization_in_rhel/configuring-user-authentication-using-authselect)도 관리 파일의 수동 편집을 경고한다. `cat /etc/pam.d/passwd`로 진입점을 읽고, 그 파일의 `password` 항목이 참조하는 `system-auth`를 `cat /etc/pam.d/system-auth`로 따라간다. `authselect current`와 `authselect check`는 관리 상태를 확인하는 조회다. 구체적인 순서는 [치트시트](cheatsheet.md)에 둔다.

사용자 화면에서 authselect는 `local` 프로필, 추가 기능 없음, 구성 유효 상태였다. `passwd`가 `system-auth`의 비밀번호 스택을 호출하고, 그 안에는 `pam_pwquality.so` → `pam_unix.so` → `pam_deny.so`가 연결되어 있다. `requisite`인 품질 모듈이 실패하면 해당 스택은 즉시 실패로 돌아간다. `pam_unix.so`의 `use_authtok`는 앞에서 받은 새 비밀번호를 이용해 로컬 비밀번호 갱신을 이어 간다. 품질 모듈이 이미 연결되어 있으므로 이번 단계에서 PAM 파일을 직접 편집할 필요는 없다. 정확한 화면 내용은 [학습 기록](learning-log.md)에 둔다.

다음에는 `cat /etc/security/pwquality.conf`와 `ls /etc/security/pwquality.conf.d`로 기존 설정과 분리 파일 목록을 읽고 규칙을 정한다. libpwquality는 drop-in을 먼저 파일명 ASCII 순서로 읽고 본체를 나중에 읽으므로, 본체에 같은 활성 항목이 있으면 drop-in 값을 덮는다. PAM 모듈 인수는 설정 파일보다 우선한다. 따라서 본체의 중복 설정도 확인한다. 이는 앞의 login.defs/libeconf와 읽기 순서가 다르다. 근거: [pwquality.conf 공식 매뉴얼](https://github.com/libpwquality/libpwquality/blob/master/doc/man/pwquality.conf.5.pod), [pam_pwquality 공식 매뉴얼](https://github.com/libpwquality/libpwquality/blob/master/doc/man/pam_pwquality.8.pod). 품질 규칙 작성과 동작 검증은 다음 단계다.

`/etc/login.defs.d/*.defs` 지원은 shadow-utils의 libeconf 사용 빌드에 달려 있다. [shadow 4.15.0의 읽기 코드](https://github.com/shadow-maint/shadow/blob/4.15.0/lib/getdef.c)는 `USE_ECONF` 조건으로 libeconf를 호출하며, [libeconf 공식 설명](https://github.com/openSUSE/libeconf/blob/master/README.md)은 분리된 설정 파일의 읽기 구조를 설명한다. libeconf는 기본 파일 뒤에 drop-in을 읽으며 파일명 사전순으로 처리한 뒤의 값이 앞의 값을 덮는다. 따라서 `/etc/login.defs.d/90-local.defs`에 세 기간 값을 모을 수 있지만, 더 뒤의 파일에 같은 키가 있으면 그 값이 우선한다. VM에서는 패키지 의존성으로 지원 빌드인지 확인하고 별도 파일 방식을 사용한다. 미지원이면 원본 파일 편집을 대안으로 선택하며 두 절차를 연달아 수행하지 않는다. 원본의 기존 변경을 복구하거나 삭제할 필요는 없다. 어느 방식도 기존 계정의 기간 값을 바꾸는 `chage`를 대신하지 않는다.

**비밀번호 필수 목표:** 만료 30일, 변경 최소 간격 2일, 만료 7일 전 경고. 최소 10자이며 대문자·소문자·숫자를 포함하고 사용자명을 포함하지 않는다. 동일 문자 연속은 최대 3개다. 이전 비밀번호에 없던 문자 최소 7개가 필요하며 **이 조건만 root 예외**다. 정책 구성 후 root를 포함해 VM의 모든 계정 비밀번호를 변경해야 한다.

`sudo chage -l jungblee`, `sudo chage -l root`로 기간 정책을 확인한다. 문자 조건은 실제 적용되는 인증·품질 정책과 허용/거절 동작을 대조한다. `chage`만으로 길이·문자 조건은 검증되지 않는다. 비밀번호 자체는 문서나 증거에 남기지 않는다.

10월 13일 사용자 화면에서 root와 jungblee 모두 최소 2일·최대 30일·경고 7일을 확인했다. 두 계정의 `Last password change`와 `Password expires`는 `never`였다. 기간 숫자 확인과 실제 만료 검증을 구분한다. 다음 순서는 **품질 정책 구성 → root와 jungblee의 실제 비밀번호 변경 → 마지막 변경 날짜·30일 뒤 만료일·2/30/7 재확인**이다. 관리자가 일반 계정의 비밀번호를 변경하면 이전 비밀번호 비교가 생략될 수 있으므로, 최종 변경 명령은 해당 경로를 확인한 뒤 정한다.

## 4. sudo 정책과 감사 로그

**반드시 읽을 범위 — 이 구간에 들어갈 때:** 원문 9쪽과 [RHEL 10 Managing sudo access](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/security_hardening/managing-sudo-access) 8.1 `User authorizations in sudoers`, 8.2 `Granting sudo access to a user`로 권한 위임을 이해한다. 설정 시 VM의 `man sudoers`에서 `Defaults`, `passwd_tries`, `badpass_message`, `requiretty`, `secure_path`, `log_input`, `log_output`, `iolog_dir`를 찾고, `man visudo`의 검사 옵션 `-c`를 읽는다.

**설명할 수 있어야 할 것:** 실행 사용자·대상 사용자·허용 명령, TTY, 명령 검색 경로, 명령 기록과 입출력 기록의 차이다. 이들은 원문이 직접 요구하는 제한과 로그를 검증하는 데 필요하다. 문법 검사 성공과 실패 시도·메시지·로그 동작 성공은 따로 확인한다.

**존재와 용도만:** `sudoreplay`는 입출력 기록을 재생하는 도구다. 로그가 남았는지를 넘어 세션 내용을 검토할 때 해당 매뉴얼을 읽는다.

**이번에 미룰 것과 이유:** 중앙 로그 서버·LDAP sudo 규칙·RHEL system roles는 단일 VM의 로컬 정책에 필요하지 않다. 여러 서버에 같은 권한을 배포하거나 로그를 외부 보존할 때 돌아온다. 이 안내는 sudo 정책 구현 완료 기록이 아니다.

**sudo 필수 목표:** 인증 실패 시 시도 3회 제한, 직접 정한 오류 메시지, 작업의 입력·출력 모두 `/var/log/sudo/` 아래 기록, TTY 모드 활성화, sudo에서 사용 가능한 경로 제한(`secure_path` 관련 개념). 오류 메시지 내용과 제한 경로의 최종 값은 아직 미선택이다.

`sudo visudo -c`로 문법, `sudo -l`로 적용 정책을 살핀다. 별도로 실패 시도 제한·메시지·TTY 요구를 확인하고 직접 수행한 sudo 작업의 입출력 로그와 대조한다. 문법 통과만으로 동작 검증을 대신하지 않는다.

## 5. SSH·firewalld·SELinux

**반드시 읽을 범위 — 이 구간에 들어갈 때:** [RHEL 10 OpenSSH](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/securing_networks/using-secure-communications-between-two-systems-with-openssh) 1.1 `SSH and OpenSSH`와 1.5.3 `Setting a non-default port for SSH`를 읽는다. [Rocky firewalld for Beginners](https://docs.rockylinux.org/guides/security/firewalld-beginners/)는 `Basic Usage`, `Saving your changes`, `Managing Zones`, `Managing Ports`, `Managing Services`까지 읽는다. [RHEL 10 SELinux 입문](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/getting-started-with-selinux)은 1.1–1.3과 1.5 `SELinux states and modes`를 읽는다. 원문 근거는 7–8쪽이다.

**설명할 수 있어야 할 것:** sshd의 리스닝·로그인 허용, 방화벽의 zone·서비스·포트와 실행/영구 설정, SELinux의 라벨·포트 유형·허용 정책은 서로 다른 층이다. 4242 일반 사용자 접속과 root 거절을 각각 확인해야 하므로 어느 층에서 막혔는지 구분한다. 설정 시 VM의 `man sshd_config`에서 `Port`, `PermitRootLogin`, `Include`, `Match`, `man firewall-cmd`에서 `--permanent`, `--reload`, 조회 옵션, `man semanage-port`에서 포트 유형을 읽는다. OpenSSH의 [공식 참조](https://man.openbsd.org/sshd_config)는 개념 대조용이며 설치 버전의 옵션·최종 적용값을 우선한다.

**존재와 용도만:** SSH 공개키 인증은 비밀번호 외의 인증 방식이고 SELinux 거절 로그는 정책 문제를 추적하는 단서다. 공개키 방식이 필요하거나 정상 설정 후에도 거절될 때 해당 부분을 더 읽는다. Enforcing·Permissive·Disabled의 차이 자체는 필수 이해 범위다.

**이번에 미룰 것과 이유:** SSH 암호 알고리즘 수학·스마트카드·점프 호스트, nftables 내부 규칙 생성, 자체 SELinux 정책 모듈 작성은 이번 접속 경로의 요구가 아니다. 인증 인프라 확장이나 기존 정책으로 해결할 수 없는 서비스 운영 때 학습한다. firewalld·SELinux를 끄는 것을 범위 축소로 삼지 않는다.

| 과제 필수 목표 | 확인 명령·행동 |
|---|---|
| SSH가 VM의 4242번 포트에서 실행, root SSH 로그인 불가 | `sudo ss -lntp`, `sudo sshd -T`. 조건별 설정도 확인하고 외부에서 일반 사용자 접속 성공·root 거절을 직접 비교한다. 평가 때 만든 새 계정으로도 접속할 수 있어야 한다. |
| Rocky는 firewalld 사용, VM 부팅 시 활성, 필수 구성에서 4242만 개방 | `systemctl is-active firewalld`, `systemctl is-enabled firewalld`, `sudo firewall-cmd --get-active-zones`, `sudo firewall-cmd --list-all-zones`, `sudo firewall-cmd --permanent --list-all-zones` |
| SELinux가 부팅 시 실행되며 과제에 맞게 구성 | `sestatus`, 재부팅 후 상태와 SSH 등 필요한 기능의 동작 확인 |

방화벽은 포트 목록뿐 아니라 활성 zone의 서비스·규칙·허용 범위도 본다. 기본 `ssh` 서비스가 22번을 열고 있지 않은지 확인한다. 현재 실행 설정과 영구 설정을 구분한다. SSH 설정·리스닝·실제 접속은 각각 확인하며 호스트의 포트 전달과 VM 내부 4242도 구분한다. 원문은 SELinux의 부팅 시 실행과 과제에 맞는 구성을 요구한다. 이번 실습은 `getenforce`·`sestatus`로 부팅 후 Enforcing 상태와 로드된 정책을 확인하고, 필요한 SSH 동작이 허용되는지 검증하는 것을 목표로 한다. 포트 유형·라벨·거절 로그를 읽고, 비활성화나 Permissive 상태로 문제를 덮은 것을 완료로 기록하지 않는다.

## 6. monitoring.sh와 cron·wall

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

## 7. 평가 재현과 README·서명

**반드시 읽을 범위 — 제출 준비 때:** [원문](../en.subject.pdf)의 인쇄 6쪽, 12–13쪽, 16–17쪽을 함께 읽는다. 일반 지침의 서명 파일 언급만 읽으면 README 요구를 놓친다. Linux에서는 설치된 `man sha1sum`의 파일 해시 출력 설명을 읽고 실제 VM 디스크를 대상으로 한다. 사용 중인 VirtualBox/UTM 도움말에서는 `Snapshots`를 찾아 생성·복원·삭제와 현재 디스크 상태의 관계를 확인한다.

**설명할 수 있어야 할 것:** 디스크 내용이 바뀌면 서명이 바뀐다는 점, VM 파일 Git 제출 금지, 평가 시작 시 스냅샷 없음과 평가용 스냅샷 종료 시 삭제, README의 선택 이유·비교·AI 사용 범위다. 근거는 원문 제출 조건이며, SHA-1 계산 성공만으로 디스크와 제출물이 일치한다고 단정하지 않는다. Debian/Rocky·AppArmor/SELinux·UFW/firewalld·VirtualBox/UTM 비교는 README 필수이므로 생략하지 않는다.

**존재와 용도만:** 전체 복제와 연결 복제는 디스크 보존 방식이 다르다는 정도를 알아둔다. 평가용 보존 방법을 실제로 선택할 때 사용 중인 가상화 도구의 해당 도움말을 더 읽는다.

**이번에 미룰 것과 이유:** SHA-1 알고리즘 구현·충돌 공격 수학, 다른 하이퍼바이저의 내부 구조, 보너스 웹 서비스 구축은 현재 필수 제출·설명 목표가 아니다. 암호학 분석·가상화 개발·보너스를 새 목표로 삼을 때 돌아온다. 비교를 위해 다른 OS 전체를 재설치할 필요는 없다는 것은 학습 범위 판단이며 비교 자체를 면제한다는 뜻이 아니다.

재부팅 후 각 목표를 다시 대조하고 선택 이유를 설명한다. README와 실제 디스크 SHA-1을 담은 `signature.txt`가 최종 제출 대상이다. VM은 Git에 넣지 않는다. 각 평가는 스냅샷 없이 시작하고 평가용 스냅샷은 이후 생성해 종료 시 삭제한다. VM 재실행으로 서명이 달라질 수 있으므로 실제 디스크와 제출 서명을 대조한다. README 등 제출 세부 조건은 기존 가이드에서 확인한다. 현재 서명 생성이나 VM 검증을 완료했다는 뜻은 아니다.

README는 현재 진행 중 문서다. 최종 첫 줄은 `*This project has been created as part of the 42 curriculum by jungblee.*`를 유지하고 Description·Instructions·Resources, 참고 자료와 AI 사용 작업·범위를 채운다. OS 선택 이유와 장단점, 파티션·보안·사용자·서비스 설계, Debian/Rocky·AppArmor/SELinux·UFW/firewalld·VirtualBox/UTM 비교를 실제 학습에 근거해 정리한다. Debian은 이 비교에 필요한 범위에서 읽는다. 선택한 OS와 도구를 설명하고 원문의 apt/aptitude 질문 예시도 비교 맥락에서 살펴본다. 평가 지침에 포함되면 작은 스크립트·동작 변경도 직접 수행할 준비를 한다.

서명 대상은 VM 설정이나 ISO가 아니라 실제 `.vdi`(UTM은 `.qcow2`)다. 원문은 서명 불일치와 그래픽 서버 설치를 0점 조건으로 명시한다. VirtualBox 사용이 필수이며 사용할 수 없을 때 UTM을 사용한다. README의 설치·실행 설명과 각 정책은 아직 검증된 최종 결과로 채우지 않는다.

## 구간별 공식 자료 읽기

전체를 한꺼번에 따라 하지 않고 해당 구간의 개요와 필요한 항목만 읽는다. Rocky 입문서는 버전이 섞일 수 있어 Rocky 10의 설치된 도구·매뉴얼과 RHEL 10 자료를 함께 대조한다. `man`은 실제 VM에서 해당 패키지의 매뉴얼을 읽는 방법이며 이번에 실행했다는 뜻이 아니다.

| 구간 | 입문 범위 |
|---|---|
| 1 | [Rocky 10 설치](https://docs.rockylinux.org/guides/installation/), [DNF](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/)의 패키지 조회·저장소·업데이트 개념. `man lsblk`, `man vgs`, `man systemctl` |
| 2 | [Rocky User Management](https://docs.rockylinux.org/books/admin_guide/06-users/)의 그룹·사용자·소유권. `man id`, `man useradd`, `man groupadd`, `man chmod` |
| 3 | [Rocky PAM](https://docs.rockylinux.org/guides/security/pam/)의 모듈 유형과 제어 흐름. `man pam_pwquality`, `man pwquality.conf`, `man chage`, `man authselect` |
| 4 | 설치된 `man sudo`, `man sudoers`의 인증, Defaults, `requiretty`, `secure_path`, 입출력 로깅 항목과 `man sudoreplay` |
| 5 | [OpenSSH sshd](https://man.openbsd.org/sshd.8), `man sshd_config`, [firewalld](https://firewalld.org/documentation/man-pages/firewall-cmd.html)의 조회·runtime/permanent, [RHEL 10 SELinux](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/index)의 시작하기·모드·거절 원인 확인 |
| 6 | [Rocky cron](https://docs.rockylinux.org/guides/automation/cron_jobs_howto/)의 예약 구조, `man 5 crontab`, `man wall`, 각 지표 도구의 매뉴얼 |
| 7 | 원문 인쇄 12–13쪽 README, 16–17쪽 제출·평가. 기존 번역에서 원문 예시와 해설을 구분 |

명령 참고: [firewall-cmd 공식 문서](https://firewalld.org/documentation/man-pages/firewall-cmd.html), [OpenSSH sshd 공식 매뉴얼](https://man.openbsd.org/sshd.8). 설치된 버전의 도움말도 함께 확인한다. 여기에 적은 VM 확인 명령은 이번 문서 작업에서 실행하지 않았다.
