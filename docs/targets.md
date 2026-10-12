# Born2beRoot — 직접 만들고 대조할 목표

**아래는 목표 상태와 확인 방법이다. 실제 실행 결과나 완료 기록이 아니다.** 한 단위씩 직접 구성하고 관찰한 결과를 [학습 기록](learning-log.md)에 남긴다. 요구사항은 [원문 v5.2](../en.subject.pdf), 자세한 항목은 [기존 가이드](ko/Born2beRoot-v5.2-ko-guide.md)를 기준으로 한다. 사용자는 보너스를 목표로 하지 않기로 했다. 아래는 필수 서버 관리·모니터링 실습에 맞춘 선택이며 보너스 그림을 복제하지 않는다.

## 남은 학습의 전체 지도

설치 시작은 사용자 보고로 확인했다. 설치 완료·실제 배치·부팅은 아직 확인하지 않았다. 아래 순서로 **전체 역할 파악 → 필요한 개념 읽기 → 한 부분 직접 적용 → 결과 확인 → 배운 내용 기록**을 반복한다. 필수 조건의 숫자와 확인 방법은 이어지는 각 구간에 둔다.

| 구간 | 목적·핵심 개념 | 끝났다고 판단할 상태 | 확인의 중심 | 이번에 미룰 심화 |
|---|---|---|---|---|
| 1. 설치 결과 | 부팅, 마운트, LUKS·LVM, 패키지·서비스 | 최소 OS가 부팅되고 설계와 실제 디스크가 일치 | OS 정보, lsblk·vgs·df, 재부팅 | RAID, thin provisioning, 디스크 축소 실행 |
| 2. 계정·그룹·권한 | UID/GID, 소유권, 기본·보조 그룹, root와 일반 사용자 | 로그인·그룹·호스트명 조건 충족, 새 계정·그룹과 호스트명 변경을 설명하며 수행 | id·getent·hostnamectl, 소유권·접근 결과 | LDAP/AD, 고급 ACL |
| 3. 비밀번호·PAM | 인증 경로, 품질과 만료의 차이, 기존 계정과 기본값 | 모든 암호 규칙과 root 예외 적용, 기존 계정 비밀번호 갱신 | chage와 실제 허용·거절 동작 | MFA, 사용자 PAM 모듈 작성 |
| 4. sudo·감사 로그 | 권한 위임, TTY, 실행 경로, 명령 로그와 입출력 로그 | 제한·오류 메시지·입출력 기록이 실제 동작 | 문법·적용 정책·로그·실패 동작 | 중앙 로그 서버, 복잡한 역할 분리 |
| 5. 원격 접속·보안 | sshd, 포트, 방화벽 zone, SELinux 정책 | 4242 일반 사용자 접속, root 거절, 부팅 후 정책 유지 | 서비스·리스닝·접속·거절 로그 | VPN, 고급 라우팅, 자체 SELinux 정책 모듈 |
| 6. 모니터링·예약 | 지표의 뜻, 표본과 계산, cron 실행 환경, wall 방송 | 모든 원문 지표, 시작 시·10분마다 방송, 수정 없는 중단 | 원자료와 출력 대조, 여러 터미널·시간 관찰 | 외부 대시보드, 장기 통계 수집 |
| 7. 재현·제출 | 재부팅 지속성, 설명·작은 변경, 디스크 서명 | 필수 조건 재현, README 완성, 서명 일치 | 직접 시연과 파일 대조 | 보너스 서비스, 설치 자동화 |

## 1. 설치 결과와 디스크 확인

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

| 구분 | 도달할 상태 | 확인 방법 |
|---|---|---|
| 과제 필수 | 최신 안정 Debian/Rocky, 최소 서비스, X.org·Wayland 등 그래픽 서버 없음 | `cat /etc/os-release`, 설치 패키지·실행 서비스 확인. 버전 출력만으로 최신 안정 여부나 그래픽 서버 부재를 증명하지는 못한다. |
| 우리 선택 | Rocky Minimal | 설치기 화면의 10.2 표시와 실제 설치·부팅 검증을 구분하고 공식 릴리스 정보와 대조 |
| 과제 필수 + 로그인 대입 | root 외에 `jungblee`, 소속 그룹에 `user42`와 `sudo` 포함 | `id jungblee`, `getent group user42`, `getent group sudo`. Rocky의 `wheel`만으로 대신하지 않는다. |
| 과제 필수 + 로그인 대입 | hostname `jungblee42` | `hostnamectl --static`; 재부팅 뒤에도 확인 |

평가에서는 새 사용자를 만들고 그룹에 넣으며 호스트명을 변경할 수 있어야 한다. 이를 준비하며 사용자·그룹 생성, 기본·보조 그룹, 파일 소유권과 권한을 작은 실습으로 확인한다. hostname은 현재 사용자 보고만 있으며 성공 결과를 가정하지 않는다. Rocky의 Kdump 설정은 과제에서 요구하지 않는다.

## 3. 비밀번호·PAM

PAM은 서비스가 인증·계정·비밀번호·세션 처리를 연결하는 틀이다. 비밀번호 품질 검사와 계정별 만료 정보는 따로 확인한다. Rocky의 authselect 관리 여부와 실제 PAM 경로를 파악하고, 기본값 변경만으로 기존 계정도 바뀌었다고 가정하지 않는다.

**비밀번호 필수 목표:** 만료 30일, 변경 최소 간격 2일, 만료 7일 전 경고. 최소 10자이며 대문자·소문자·숫자를 포함하고 사용자명을 포함하지 않는다. 동일 문자 연속은 최대 3개다. 이전 비밀번호에 없던 문자 최소 7개가 필요하며 **이 조건만 root 예외**다. 정책 구성 후 root를 포함해 VM의 모든 계정 비밀번호를 변경해야 한다.

`sudo chage -l jungblee`, `sudo chage -l root`로 기간 정책을 확인한다. 문자 조건은 실제 적용되는 인증·품질 정책과 허용/거절 동작을 대조한다. `chage`만으로 길이·문자 조건은 검증되지 않는다. 비밀번호 자체는 문서나 증거에 남기지 않는다.

## 4. sudo 정책과 감사 로그

**sudo 필수 목표:** 인증 실패 시 시도 3회 제한, 직접 정한 오류 메시지, 작업의 입력·출력 모두 `/var/log/sudo/` 아래 기록, TTY 모드 활성화, sudo에서 사용 가능한 경로 제한(`secure_path` 관련 개념). 오류 메시지 내용과 제한 경로의 최종 값은 아직 미선택이다.

`sudo visudo -c`로 문법, `sudo -l`로 적용 정책을 살핀다. 별도로 실패 시도 제한·메시지·TTY 요구를 확인하고 직접 수행한 sudo 작업의 입출력 로그와 대조한다. 문법 통과만으로 동작 검증을 대신하지 않는다.

## 5. SSH·firewalld·SELinux

| 과제 필수 목표 | 확인 명령·행동 |
|---|---|
| SSH가 VM의 4242번 포트에서 실행, root SSH 로그인 불가 | `sudo ss -lntp`, `sudo sshd -T`. 조건별 설정도 확인하고 외부에서 일반 사용자 접속 성공·root 거절을 직접 비교한다. 평가 때 만든 새 계정으로도 접속할 수 있어야 한다. |
| Rocky는 firewalld 사용, VM 부팅 시 활성, 필수 구성에서 4242만 개방 | `systemctl is-active firewalld`, `systemctl is-enabled firewalld`, `sudo firewall-cmd --get-active-zones`, `sudo firewall-cmd --list-all-zones`, `sudo firewall-cmd --permanent --list-all-zones` |
| SELinux가 부팅 시 실행되며 과제에 맞게 구성 | `sestatus`, 재부팅 후 상태와 SSH 등 필요한 기능의 동작 확인 |

방화벽은 포트 목록뿐 아니라 활성 zone의 서비스·규칙·허용 범위도 본다. 기본 `ssh` 서비스가 22번을 열고 있지 않은지 확인한다. 현재 실행 설정과 영구 설정을 구분한다. SSH 설정·리스닝·실제 접속은 각각 확인하며 호스트의 포트 전달과 VM 내부 4242도 구분한다. 원문은 SELinux의 부팅 시 실행과 과제에 맞는 구성을 요구한다. 이번 실습은 `getenforce`·`sestatus`로 부팅 후 Enforcing 상태와 로드된 정책을 확인하고, 필요한 SSH 동작이 허용되는지 검증하는 것을 목표로 한다. 포트 유형·라벨·거절 로그를 읽고, 비활성화나 Permissive 상태로 문제를 덮은 것을 완료로 기록하지 않는다.

## 6. monitoring.sh와 cron·wall

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
