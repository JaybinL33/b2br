*This project has been created as part of the 42 curriculum by jungblee.*

# Born2beRoot

## Description

Born2beRoot는 가상 머신에 최소 구성의 Linux 서버를 구축하는 시스템 관리 과제다. 암호화된 저장소, 사용자와 권한, 비밀번호 정책, 원격 접속, 방화벽과 시스템 모니터링을 구성한다.

이 프로젝트는 VirtualBox의 Rocky Linux 10.2 Minimal을 사용한다. VM 이름은 `b2br`, 게스트 호스트명은 `jungblee42`다. SSH는 4242/tcp에서 제공하며 SELinux와 firewalld를 사용한다.

## Instructions

### 설치 구성

별도 컴파일 과정은 없다. Rocky Minimal 설치 후 LUKS 안에 LVM 볼륨을 구성하고, 아래 Project description의 사용자·보안 정책과 서비스를 설정한다. Bash 모니터링 스크립트는 VM의 `/usr/local/sbin/monitoring.sh`에 root 소유·755 권한으로 설치되어 있다. VM 디스크는 Git에 포함하지 않는다.

### 실행

VirtualBox에서 `b2br`를 시작하고 콘솔에서 암호화 디스크의 암호를 입력한다. 현재 호스트의 NAT 포워딩은 `127.42.42.42:4242`에서 게스트의 4242/tcp로 연결된다. 호스트 터미널에서 접속한다.

```bash
ssh -p 4242 jungblee@127.42.42.42
```

이 주소는 해당 호스트의 포워딩 주소다. 다른 호스트에서는 VirtualBox의 NAT 포워딩에 설정한 주소를 사용한다. VM 안에서 서비스와 호스트명을 조회하고 모니터링을 수동 실행할 수 있다.

```bash
hostname
sudo systemctl is-active sshd firewalld crond
getenforce
sudo /bin/bash /usr/local/sbin/monitoring.sh
```

모니터링은 root crontab에 부팅 시와 10분 간격으로 등록되어 있다. 스크립트는 시스템 정보 12항목을 모아 `wall`로 방송한다. crontab에 PATH를 명시하지 않고 cron이 제공하는 실행 환경을 사용한다. `/bin/bash`와 스크립트는 절대경로로 호출하며 내부의 경로 없는 명령은 cron PATH로 검색한다.

```cron
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

스크립트를 수정하지 않고 예약을 중단하려면 `sudo crontab -e`에서 위 두 실행 줄을 주석 처리한다. 다시 활성화하려면 주석을 제거한다.

## Project description

### 운영체제 선택과 비교

Rocky Minimal을 선택해 SELinux·firewalld와 RHEL 계열의 사용자·서비스 관리 도구로 서버를 구성했다. 이 구성의 장점은 배포판의 보안 정책과 관리 도구를 함께 사용할 수 있다는 점이다. 반면 SELinux 정책, firewalld zone, PAM 설정 경로를 각각 이해하고 관리해야 한다.

| 비교 | 차이와 선택의 대가 |
|---|---|
| Debian / Rocky Linux | Debian stable은 운영용 안정 릴리스와 정해진 지원 주기를 제공하며 과제에서 입문자에게 권장한다. 안정 릴리스의 패키지가 필요한 최신 기능을 제공하는지는 별도 확인이 필요하다. Rocky는 RHEL 호환성을 지향하는 엔터프라이즈 Linux로 관련 관리 도구와 문서를 활용하기 좋지만, 이 과제에서는 SELinux·firewalld 등 함께 다룰 설정이 많다. |
| AppArmor / SELinux | 둘 다 일반 파일 권한에 더해 접근을 제한한다. AppArmor는 프로그램별 프로파일에서 파일 경로와 권한을 표현해 허용 범위를 읽기 쉽지만 보호 범위는 적용된 프로파일에 달린다. SELinux는 보안 컨텍스트·타입과 정책으로 접근을 판단하므로 권한뿐 아니라 라벨과 정책도 관리해야 한다. 이 VM은 SELinux Enforcing을 유지한다. |
| UFW / firewalld | UFW는 allow/deny 중심의 간단한 인터페이스로 소수의 호스트 방화벽 규칙을 관리하기 편하다. firewalld는 zone·서비스 단위 구성이 가능하지만 활성 zone과 runtime/permanent 설정을 함께 관리해야 한다. Rocky에 대한 과제 조건에 따라 firewalld를 사용한다. |
| VirtualBox / UTM | VirtualBox는 이 환경에서 VM, NAT 포워딩, 스냅샷을 관리하며 과제의 기본 도구다. 호스트·게스트 지원은 설치 버전에 따른다. UTM은 Mac에서 QEMU 기반 가상화·에뮬레이션과 Apple Virtualization을 제공한다. 다른 아키텍처의 에뮬레이션은 활용 범위를 넓히지만 네이티브 가상화보다 느릴 수 있다. 과제는 VirtualBox를 사용할 수 없을 때 UTM을 허용한다. |

### 파티션

약 30 GiB 디스크에 BIOS Boot 영역과 비암호화 `/boot` 2 GiB를 두고, 나머지는 LUKS 암호화 영역 안의 VG `LVM`으로 구성했다.

| 논리 볼륨 | 용량 | 용도 |
|---|---|---|
| root | 10 GiB | OS·패키지와 별도 분리하지 않은 시스템 데이터 |
| home | 4 GiB | 사용자 파일을 root 공간과 분리 |
| var_log | 2 GiB | `/var/log`와 sudo 기록의 공간을 분리 |
| swap | 4 GiB | 스왑 영역 |

VG에는 약 8 GiB를 미할당으로 남겨 필요한 볼륨을 확장할 수 있게 했다. 로그 볼륨 분리는 root의 공간 고갈 위험을 줄이지만 로그 보관·순환 관리를 대신하지 않는다.

### 사용자와 보안 정책

일반 작업 계정은 `jungblee`이며 `user42`, `sudo` 그룹에 속하고 `wheel`에는 속하지 않는다. 이 계정은 sudo로 관리 작업을 수행하며 `(ALL : ALL) ALL` 허용과 root 권한 실행을 확인했다. SSH root 로그인은 금지하고 일반 계정으로 접속한다.

비밀번호 기간은 최소 2일·최대 30일·만료 경고 7일이다. libpwquality에 길이 10 이상, 대문자·소문자·숫자 포함, 동일 문자 연속 최대 3, 사용자명 제외, `difok=7`을 설정했다. `enforce_for_root`로 root에도 품질 검사를 적용하며 이전 비밀번호 비교의 root 예외를 구분한다.

sudo는 인증 실패 최대 3회, 사용자 정의 실패 메시지, TTY 요구, 제한된 실행 경로를 설정했다. 입출력은 `/var/log/sudo/`에 기록해 관리 작업을 추적한다. firewalld의 활성 zone에는 4242/tcp만 열고 SELinux의 SSH 포트 정책도 맞췄다.

### 서비스와 모니터링

- `sshd`: 4242/tcp에서 일반 계정의 원격 관리 접속을 제공한다.
- `firewalld`: 활성 인터페이스의 접근 포트를 제한한다.
- `crond`: root의 모니터링 예약을 실행한다.

세 서비스는 부팅 시 실행되도록 설정했다. SELinux는 접근 제어 계층이며 별도의 원격 서비스가 아니다. 모니터링은 OS·커널, 물리·논리 CPU, 메모리, 디스크, CPU 사용률, 부팅 시각, LVM, TCP 연결, 사용자, IP·MAC, sudo 기록을 출력한다. 메모리·디스크는 MiB 단위의 사용량/전체, 사용률과 available을 함께 표시한다. sudo 값은 현재 보관된 I/O 세션 수이므로 삭제된 과거 기록까지 포함하는 영구 누적값은 아니다.

## Resources

- [Born2beRoot v5.2 과제](https://cdn.intra.42.fr/pdf/pdf/197915/en.subject.pdf): 구성·README·평가 요구사항.
- [Rocky Linux](https://rockylinux.org/) · [Debian Releases](https://www.debian.org/releases/): 배포판의 성격과 릴리스 정책.
- [Rocky User Management](https://docs.rockylinux.org/books/admin_guide/06-users/): 계정·그룹·비밀번호 관리.
- [RHEL LVM](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_logical_volumes/overview-of-logical-volume-management) · [LUKS](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/encrypting-block-devices-by-using-luks): 저장소 계층과 암호화.
- [Rocky SELinux](https://docs.rockylinux.org/guides/security/learning_selinux/) · [AppArmor](https://ubuntu.com/server/docs/how-to/security/apparmor/): 접근 제어 방식.
- [firewalld](https://firewalld.org/documentation/concepts.html) · [UFW](https://ubuntu.com/server/docs/how-to/security/firewalls/): 방화벽 관리.
- [VirtualBox 매뉴얼](https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/Introduction.html) · [UTM](https://mac.getutm.app/): 가상 머신과 스냅샷·복제.
- VM의 `man sudoers`, `man crontab`, `man wall`, `man hostnamectl`: 설치된 도구의 설정과 동작.

### AI 활용

AI는 과제 번역·요구사항 정리, 공식 자료 탐색, 학습 문서와 이 README의 작성·수정에 사용했다. OS·도구 비교와 설계 설명은 공식 자료 및 실제 작업 기록을 바탕으로 정리했다. 모니터링 스크립트의 단계별 부분 예시·설명과 AI 작업 환경에서의 예시 검사에도 사용했다. VM 설치, 설정 적용과 실행은 직접 수행했으며 AI가 사용자 VM을 조작하지 않았다.
