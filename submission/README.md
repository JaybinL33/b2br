*This project has been created as part of the 42 curriculum by jungblee.*

# Born2beRoot

## Description

Born2beRoot는 가상 머신에 최소 구성의 Linux 서버를 구축하고 직접 관리하는 과제다. 디스크 암호화, 저장 공간 분리, 사용자 권한, 원격 접속과 모니터링을 구성해 데이터와 관리 경로를 보호하고 서버 상태를 파악한다.

VirtualBox에 Rocky Linux 10.2 Minimal을 설치함. VM 이름은 `b2br`, 호스트명은 `jungblee42`로 지정함. SSH 4242/tcp로 관리하며 firewalld와 SELinux Enforcing을 사용 중. 시스템 정보를 수집하는 Bash 스크립트를 작성하고 cron에 정기 방송을 등록함.

## Instructions

### 실행 환경

실행에는 Rocky Minimal과 아래의 저장소·보안·서비스 구성을 갖춘 VirtualBox VM을 사용한다. 제출 저장소에는 `README.md`와 가상 디스크 SHA-1을 담은 `signature.txt`를 두고, VM 디스크는 호스트에서 별도로 관리한다.

VirtualBox에서 `b2br`를 시작하고 콘솔에서 LUKS 암호를 입력한다. 현재 호스트의 `127.42.42.42:4242`를 게스트의 4242/tcp에 연결하도록 NAT 포워딩을 설정함. 호스트 터미널에서 접속한다.

```bash
ssh -p 4242 jungblee@127.42.42.42
```

다른 호스트에서는 해당 VirtualBox 포워딩 주소를 사용한다. 게스트에 접속한 뒤 서비스 상태를 조회하고 모니터링을 수동 실행할 수 있다.

```bash
systemctl is-active sshd firewalld crond
getenforce
sudo /bin/bash /usr/local/sbin/monitoring.sh
```

### 모니터링 예약

스크립트를 `/usr/local/sbin/monitoring.sh`에 root 소유·755 권한으로 설치함. root crontab에 다음 두 예약을 등록함.

```cron
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

`*/10`은 매시 0·10·20·30·40·50분, `@reboot`는 부팅 시 실행을 위한 예약이다. 스크립트 내부에서 경로를 생략한 명령은 cron 환경의 PATH에 나열된 디렉터리에서 찾는다. `/bin/bash`와 스크립트 경로는 절대경로로 지정함.

예약은 `sudo crontab -l`로 조회한다. 방송을 중단하려면 `sudo crontab -e`에서 두 실행 줄을 모두 주석 처리하고, 복구할 때는 주석을 제거한다.

## Project description

### 운영체제와 가상화 도구

RHEL 계열의 SELinux·firewalld·사용자 관리 방식을 익히기 위해 Rocky Minimal을 선택함. 필요한 서비스 위주로 설치하면 업데이트하고 관리할 대상이 줄어든다. SSH 포트를 변경할 때는 sshd 설정, 방화벽, SELinux 포트 정책을 함께 맞춰야 한다.

| 비교 | 선택 기준과 제약 |
|---|---|
| Debian / Rocky Linux | Debian은 APT와 폭넓은 패키지 저장소를 이용할 수 있어 범용 서버 구성에 적합하다. stable의 패키지 버전이 필요한 기능보다 오래되면 backports 등 별도 공급 경로를 검토해야 한다. Rocky는 RHEL 계열의 운영 방식과 호환성이 필요한 환경에 적합하다. 이 구성에서는 서비스 포트나 데이터 경로를 변경할 때 SELinux 정책과 방화벽 설정까지 함께 관리해야 한다. |
| VirtualBox / UTM | 현재 호스트에서 게스트와 NAT 포워딩을 관리하기 위해 VirtualBox를 선택함. 사용할 호스트·게스트 아키텍처는 지원 범위에 맞춰야 한다. UTM은 Mac에서 QEMU와 Apple Virtualization을 활용하며, 다른 CPU 아키텍처도 에뮬레이션할 수 있지만 네이티브 가상화보다 성능 비용이 크다. 과제에서는 VirtualBox를 사용할 수 없을 때 UTM을 허용한다. |

### 저장소

약 30 GiB 디스크에서 `/boot` 2 GiB는 암호화 영역 밖에 배치함. 나머지 주요 볼륨은 `LUKS → LVM PV → VG LVM → LV` 구조로 구성함.

| 볼륨 | 용량 | 파일시스템·용도 |
|---|---|---|
| root | 10 GiB | ext4, `/` — OS·패키지·시스템 데이터 |
| home | 4 GiB | ext4, `/home` — 사용자 파일 |
| var_log | 2 GiB | ext4, `/var/log` — 시스템·sudo 로그 |
| swap | 4 GiB | 스왑 — 메모리 압박 시 사용할 디스크 공간 |

LUKS는 디스크에 저장되는 데이터를 암호화한다. 디스크 파일이 유출됐을 때 내부 데이터를 보호하기 위해 적용함. 디스크 잠금을 해제한 뒤에는 사용자 권한과 SELinux 정책에 따라 프로그램의 파일 접근이 허용되거나 차단된다.

LVM으로 디스크 공간을 볼륨 그룹(VG)에 묶고, 용도별 논리 볼륨(LV)에 나눠 줄 수 있다. 사용량에 따라 LV를 확장하기 위해 VG에 약 8 GiB를 남겨 둠. ext4는 오프라인 축소를 지원한다. 사용 중인 LV의 공간을 다른 LV에 재배정하려면 파일시스템과 LV의 크기를 순서에 맞춰 조정해야 한다.

사용자 파일과 로그에 별도 용량을 배정하고 root의 공간을 확보하기 위해 `/home`과 `/var/log`를 분리함. 로그 볼륨 자체가 가득 차면 기록과 sudo 실행에 영향을 줄 수 있으므로 용량 점검과 로그 순환은 여전히 필요하다.

### 사용자와 관리 권한

일반 계정 `jungblee`를 `jungblee`, `user42`, `sudo` 그룹에 두고 사용 중. 개인 계정으로 SSH에 접속한 뒤 sudo로 관리 작업을 수행함. 이 계정의 sudo 허용 범위는 `(ALL : ALL) ALL`로, 모든 대상 사용자·그룹 권한으로 명령을 실행할 수 있다.

여러 사용자를 그룹에 넣으면 같은 sudoers 규칙으로 명령과 실행 조건을 지정할 수 있다. 개인 계정에서 sudo를 사용하면 관리 작업을 실행한 사용자를 기록으로 추적할 수 있다.

sudo에 인증 시도 최대 3회, 사용자 정의 실패 메시지, TTY 요구와 `secure_path`를 설정함. `secure_path`는 sudo가 실행 파일을 검색할 디렉터리를 지정한다. sudo로 실행한 작업의 입력·출력을 `/var/log/sudo/`에 기록하도록 설정함.

비밀번호에 최소 길이 10, 대문자·소문자·숫자 포함, 동일 문자 연속 최대 3, 사용자명 검사와 `difok=7`을 설정함. 변경 주기는 최소 2일·최대 30일, 만료 경고는 7일 전으로 지정함. root가 비밀번호를 변경할 때도 품질 기준을 적용하도록 `enforce_for_root`를 설정함. root는 자신이나 다른 계정의 비밀번호를 변경할 때 기존 비밀번호를 입력하지 않으므로 이전 비밀번호와의 유사성 검사는 생략된다.

### 접근 제어

firewalld는 네트워크에서 서버로 들어오는 연결을 제한한다. 활성 zone에는 SSH용 4242/tcp만 개방함. sshd에서 root의 SSH 로그인을 금지함. SSH 서버가 4242 포트를 사용할 수 있도록 SELinux에 등록함.

서비스 취약점으로 공격자가 프로세스를 제어하게 되면, 실행 계정이 접근할 수 있는 다른 자료까지 피해가 번질 수 있다. SELinux 정책에는 프로그램이 접근할 수 있는 파일과 사용할 수 있는 포트 등의 규칙이 담겨 있다. Enforcing 모드에서는 일반 파일 권한이 허용하는 접근도 SELinux 정책에 어긋나면 차단된다. 서비스가 침해되었을 때 다른 자료로 피해가 번지는 위험을 줄이기 위해 Enforcing 모드를 사용 중.

| 비교 | 선택 기준과 제약 |
|---|---|
| AppArmor / SELinux | AppArmor 프로파일에는 프로그램이 접근할 파일 경로와 허용할 작업을 적는다. 파일 위치가 바뀌면 관련 경로 규칙을 확인해야 한다. SELinux는 프로그램과 파일에 부여된 보안 라벨을 보고 정책에 허용된 접근인지 판단한다. 파일 위치나 서비스 포트를 바꿀 때는 라벨과 정책도 함께 확인해야 한다. Rocky의 기본 보안 체계에 맞춰 SELinux를 사용함. |
| UFW / firewalld | UFW는 allow/deny 규칙으로 소수 포트를 관리할 때 간결하다. 여러 인터페이스를 신뢰 수준별로 나누려면 그에 맞는 규칙 설계가 필요하다. firewalld는 네트워크 인터페이스나 접속 출발지에 zone을 지정해 서로 다른 허용 규칙을 적용할 수 있다. 현재 적용된 규칙과 재시작 후 사용할 영구 설정을 함께 관리해야 한다. Rocky에 대한 과제 조건에 따라 firewalld를 사용함. |

### 서비스와 모니터링

| 서비스 | 역할과 선택 이유 |
|---|---|
| sshd | 호스트에서 게스트를 원격 관리하기 위한 SSH 접속 제공 |
| firewalld | 필요한 관리 포트만 허용하고 재부팅 뒤에도 방화벽 정책 적용 |
| crond | 주기·부팅 작업으로 모니터링 실행 |

세 서비스가 부팅 시 실행되도록 설정함. 모니터링 스크립트에서 값을 수집한 뒤 보고서를 `wall`에 한 번 전달하도록 작성함. `wall`은 전달받은 내용을 로그인한 터미널에 방송한다. 출력 항목은 OS·커널, 물리 CPU 수, vCPU 수, 메모리, 디스크, CPU 사용률, 마지막 부팅 시각, LVM 여부, TCP 연결, 로그인 사용자, IP·MAC, sudo 기록이다.

메모리·디스크의 사용량과 전체 용량, 사용률, 가용량(available)을 표시하도록 작성함. 용량 단위는 MiB를 사용 중. 디스크 집계 대상은 tmpfs·devtmpfs를 제외한 마운트된 파일시스템이다. CPU 값은 `vmstat`의 유휴 비율을 100에서 뺀 값이다. 연결 수는 ESTABLISHED TCP 소켓을 세고, 사용자 수는 로그인명 중복을 제거해 계산한다. sudo 값은 현재 보관된 I/O 세션 수다.

## Resources

- [Born2beRoot v5.2 과제](https://cdn.intra.42.fr/pdf/pdf/197915/en.subject.pdf): 구성과 제출 요구사항.
- [Rocky Linux](https://rockylinux.org/) · [Debian Releases](https://www.debian.org/releases/): 운영체제와 릴리스 정책.
- [Rocky User Management](https://docs.rockylinux.org/books/admin_guide/06-users/): 계정·그룹 관리.
- [RHEL LVM](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_logical_volumes/overview-of-logical-volume-management) · [LUKS](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/encrypting-block-devices-by-using-luks): 공간 관리와 디스크 암호화.
- [Rocky SELinux](https://docs.rockylinux.org/guides/security/learning_selinux/) · [AppArmor](https://ubuntu.com/server/docs/how-to/security/apparmor/): 프로세스 접근 제어.
- [firewalld](https://firewalld.org/documentation/concepts.html) · [UFW](https://ubuntu.com/server/docs/how-to/security/firewalls/): 네트워크 접근 제어.
- [VirtualBox 매뉴얼](https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/Introduction.html) · [UTM](https://mac.getutm.app/): 가상화 도구.
- VM의 `man sudoers`, `man pwquality.conf`, `man crontab`, `man wall`: 설치된 도구의 정책과 실행 방식.

### AI 활용

AI를 과제 번역·요구사항 정리, 공식 문서 탐색, 학습 자료와 이 README의 작성·수정에 사용함. 모니터링 스크립트의 단계별 코드 예시, 지표 해석과 예시 코드 검사에도 활용함. 구성 설명을 실제 작업 기록과 대조하고, VM 설치·설정 적용·실행은 직접 수행함.
