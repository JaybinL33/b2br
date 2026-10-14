*This project has been created as part of the 42 curriculum by jungblee.*

# Born2beRoot

## Description

Born2beRoot는 가상 머신에 최소 구성의 Linux 서버를 만들고 디스크, 사용자, 권한, 원격 접속과 보안 정책을 직접 관리하는 42 시스템 관리 과제다. 이 저장소는 **과제 v5.2**의 원문과 참고 자료, 반복 실습에서 실제로 배운 내용을 함께 보관한다.

현재는 학습을 이어 가기 위한 진행 중 문서다. 아래 설치·실행 안내와 설계 설명은 기존 실습을 정리한 것이며, 최종 제출 전 실제 구성과 다시 대조한다.

이 GitHub 저장소(`JaybinL33/b2br`)는 원문·번역·학습 기록을 보관하는 학습 저장소다. 실제 42 제출 저장소의 주소와 이 저장소와의 관계는 아직 확인하지 않았다. 원문 인쇄 16쪽의 제출 대상은 루트 `README.md`와 `signature.txt`이므로, 제출 README는 이곳의 `docs/` 링크가 없어도 설치·실행 안내와 필수 설계 설명을 읽을 수 있도록 준비한다. 현재 학습 자료를 삭제하거나 제출 저장소를 새로 만들었다는 뜻은 아니다.

## 자료와 현재 상태

- [과제 원문 PDF](en.subject.pdf)
- [한국어 전체 번역](docs/ko/Born2beRoot-v5.2-ko-full.md) · [PDF](docs/ko/Born2beRoot-v5.2-ko-full.pdf)
- [요구사항 정리와 평가 체크리스트](docs/ko/Born2beRoot-v5.2-ko-guide.md) · [PDF](docs/ko/Born2beRoot-v5.2-ko-guide.pdf)
- [학습 기록](docs/learning-log.md)
- [직접 만들고 대조할 목표](docs/targets.md): 디스크 구조 예시와 단위별 필수 상태·확인 방법
- [최소 치트시트](docs/cheatsheet.md): 처음부터 재실습하는 절차·명령·설정·기대 결과

이 실습은 Rocky Minimal·VirtualBox, LUKS 위 LVM, SSH 4242와 firewalld·SELinux를 사용한다. 호스트 접속 기준은 `127.42.42.42:4242`다. 구체적인 파티션 용량과 보안 선택은 [목표 문서](docs/targets.md)에 둔다.

**확인한 상태:** 호스트명 check42 변경·SSH 재접속 및 jungblee42로 현재·저장 이름 복원(이번 연습의 재부팅 검증은 미실시), 모니터링 12항목 출력, root 소유 설치본, 첫 cron 방송의 해당 터미널 단일 수신, 재부팅 후 서비스·SELinux·활성 방화벽 규칙 유지 및 @reboot 작업 시작·종료 기록, 두 예약 줄 주석 처리 후 30분 무방송을 통한 중단 검증, 예약 설정 복구와 복구 후 방송 수신(사용자 확인). **남은 확인:** 여러 터미널 동시 수신, 부팅 방송 수신, 아직 시험하지 않은 비밀번호 품질 조건·root 자신의 품질 적용과 평가 재현, 실제 연결 디스크 확인·제출 서명 파일 작성·최종 대조. 부팅 작업 로그만으로 내부 오류 부재나 방송 수신까지 확정하지 않는다. 세부 근거와 이전 이력은 [학습 기록](docs/learning-log.md)에 둔다.

원문·번역·가이드는 요구사항을 찾는 자료이고, 학습 기록은 직접 적용한 결과를 남기는 문서다. 문서를 읽거나 예시를 확인한 것만으로 체크리스트를 완료 처리하지 않는다.

## Instructions

### 학습 흐름

공식 입문 자료로 전체 구조를 살핀 뒤, 지금 필요한 개념과 선택 근거를 좁혀 읽는다. 작은 단위로 직접 실습하고 결과를 확인한 다음, 실제로 배운 내용만 간결한 한국어로 기록한다. 이후 전체 구조에서 이번 작업의 위치를 다시 확인한다. 기본 문법 풀이와 코드의 단순 번역, 미리 완성된 설정을 따라 적는 방식은 피한다.

| 순서 | 학습 단위 | 확인할 질문 |
|---|---|---|
| 1 | 설치 결과 확인 | 부팅한 OS·디스크·마운트가 계획과 일치하는가? |
| 2 | 계정·그룹·권한 | 사용자와 파일 권한을 구분하고 호스트명을 바꿀 수 있는가? |
| 3 | 비밀번호·PAM | 품질·만료 규칙이 기존 계정에도 적용되는가? |
| 4 | sudo 정책·감사 로그 | 권한 위임과 제한, 입출력 기록이 실제 동작하는가? |
| 5 | SSH·firewalld·SELinux | 접속 경로의 각 계층이 허용·거절을 어떻게 결정하는가? |
| 6 | 모니터링·cron·wall | 지표의 근거와 방송·예약·중단을 확인할 수 있는가? |
| 7 | 평가 재현·README·서명 | 재부팅 후 재현하고 선택 이유와 제출물을 설명할 수 있는가? |

각 구간의 [전체 학습 지도와 목표](docs/targets.md)에 공식 자료의 읽을 절, 반드시 설명할 개념, 존재만 알아둘 기능, 심화를 미룰 이유와 다시 읽을 시점을 정리했다. 먼저 [자료 선택과 읽는 깊이](docs/targets.md#자료-선택과-읽는-깊이)를 보고 해당 구간으로 이동한다.

### 첫 단위: 디스크 구조 읽기

원문 인쇄 7–8쪽의 요구사항과 그림에서 시작한다. [설치 결과와 디스크 확인](docs/targets.md#1-설치-결과와-디스크-확인)에 정한 순서로 공식 입문 자료를 읽고, 파티션·암호화·LVM·파일시스템·마운트의 관계를 실제 출력과 연결한다. 자세한 읽기 범위는 목표 문서 한곳에서 관리한다.

### 설치와 실행

별도 컴파일은 없다. VirtualBox에서 Rocky Minimal을 설치하고 Bash 스크립트를 실행한다. 과제는 VirtualBox를 요구하며 사용할 수 없을 때 UTM을 허용한다. 설치된 OS는 Rocky 10.2로 확인했지만, 제출 시점의 최신 안정 버전 조건은 공식 릴리스와 별도로 대조한다.

1. Rocky 설치 ISO로 부팅해 그래픽 서버 없는 최소 설치를 선택한다. 약 30 GiB 디스크에 BIOS Boot 영역과 비암호화 `/boot`를 두고, 나머지는 LUKS 안의 LVM으로 구성한다. 아래 설계 용량을 배정하고 root와 관리자 jungblee를 만든다.
2. 호스트명을 `jungblee42`로 설정하고 jungblee를 `user42`·`sudo`에 추가한다. Rocky의 관리자 그룹 `wheel`과 과제 그룹 이름은 구별한다. 아래 비밀번호·sudo 정책을 적용하고 기존 계정의 비밀번호도 갱신한다. 세부 설정 순서는 [치트시트](docs/cheatsheet.md)에 있다.
3. OpenSSH 서버를 4242/tcp에서 실행하고 root SSH 로그인을 금지한다. SELinux는 Enforcing으로 유지하며 SSH 포트 정책을 맞춘다. firewalld의 활성 인터페이스 zone에는 4242/tcp만 열고 runtime·permanent 설정을 대조한다. `sshd`, `firewalld`, `crond`는 부팅 때 실행되도록 설정한다.
4. 직접 작성한 `monitoring.sh`를 `/usr/local/sbin/monitoring.sh`에 root 소유·755로 설치한다. root crontab은 아래와 같다. 스크립트는 12개 지표를 모아 `wall` 한 번으로 전달한다. 기존 실습 소스는 [치트시트의 전체 코드](docs/cheatsheet.md#34-전체-출력을-wall-한-번으로-전달)에 보관한다.

```cron
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

실행할 때는 VirtualBox에서 **b2br**를 시작하고 콘솔에서 암호화 디스크를 해제한다. VM 이름 b2br와 게스트 호스트명 jungblee42는 다르며, VirtualBox 목록의 별도 VM jungblee42를 혼동하지 않는다. 현재 호스트의 NAT 전달 주소에서 다음과 같이 접속한다. 다른 호스트에서 재현하면 포워딩 주소를 그 환경에 맞춰 설정해야 한다.

```bash
ssh -p 4242 jungblee@127.42.42.42
hostname
sudo systemctl is-active sshd firewalld crond
getenforce
sudo /bin/bash /usr/local/sbin/monitoring.sh
```

호스트명은 `jungblee42`, 세 서비스는 `active`, SELinux는 `Enforcing`을 기대한다. 수동 방송과 별도로 부팅·10분 주기 방송 수신을 확인한다. 중단 시험은 `sudo crontab -e`에서 두 실행 줄만 주석 처리하며 스크립트를 수정하지 않는다. 복원은 같은 두 줄의 주석을 제거한다. 이 명령 목록이 모든 평가 항목의 검증 완료를 뜻하지는 않는다.

### 제출

v5.2 인쇄 12–13쪽·16–17쪽에 따라 최종 제출은 루트 `README.md`와 실제 가상 디스크 SHA-1을 담은 `signature.txt`다. VM은 Git에 넣지 않는다. 평가는 스냅샷 없이 시작하고, 평가용 스냅샷은 이후 생성해 종료 시 삭제한다. VM 재실행으로 서명이 바뀔 수 있으므로 최종 디스크와 제출 서명을 대조한다.

전달받은 화면에서 선택된 **b2br**의 Snapshots 목록에는 `Current State`만 있으며 상태는 `Running`이다. 파일 관리자에는 `~/goinfre/VirtualBox VMs/b2br/b2br.vdi`가 보인다. Storage 연결 화면을 확인한 것은 아니므로 이 파일을 실제 연결 디스크로 확정하지 않는다. 이후 사용자가 해당 파일의 SHA-1 `1ac763cd9e7e5526d602a6dc928d3563fb186b00`을 계산했다. 사용자가 계산 당시 VM이 Powered Off였음을 확인했다. 실제 연결 디스크와 제출 서명 파일의 대조는 남아 있다. 실제 42 제출 저장소의 주소·로컬 경로와 `signature.txt` 작성·제출 여부도 미확인이다.

평가 사이의 디스크 보존은 [서명 보존과 복원 절차](docs/targets.md#평가-사이의-디스크-보존과-복원)를 따른다. 원문은 평가 시작 전 스냅샷 없음과 평가 중 전용 스냅샷 생성·종료 시 삭제를 구분한다. Full Clone을 동일 SHA-1 백업으로 취급하지 않는다.

## Project description

### OS 선택과 비교

이 실습은 Rocky Minimal에서 SELinux·firewalld·PAM과 사용자 관리를 함께 다루는 구성을 선택했다. 기존 실습을 기준으로 정리한 선택 근거는 RHEL 계열의 서비스·보안 관리 방식을 직접 익힐 수 있다는 점이다. 과제 자체는 입문자에게 Debian을 권장하며 Rocky가 더 복잡하다고 설명한다. Rocky 선택이 Debian보다 쉽거나 더 안전하다는 뜻은 아니다.

| 비교 | 장점과 대가, 이번 구성과의 관계 |
|---|---|
| Debian / Rocky Linux | Debian stable은 운영용 안정 릴리스와 정해진 지원 주기를 제공하며 과제에서 입문자에게 권장한다. 안정 버전을 쓰는 만큼 최신 기능이 필요하면 패키지 버전을 따로 확인해야 한다. Rocky는 엔터프라이즈 Linux·RHEL 호환성을 지향해 관련 관리 방식을 학습하기 좋다. 이번에는 SELinux 정책과 firewalld zone, PAM 관리 경로까지 함께 이해해야 하는 학습 부담이 있다. 두 배포판 모두 최신 기능보다 안정적 운영 조건을 확인해야 한다. [Debian 릴리스](https://www.debian.org/releases/) · [Rocky 소개](https://rockylinux.org/) |
| AppArmor / SELinux | 둘 다 일반 파일 권한에 더해 프로그램의 접근을 제한하는 강제 접근 제어다. AppArmor는 프로그램별 프로파일과 파일 경로 규칙으로 허용 범위를 읽기 좋지만 실제 보호는 적용된 프로파일 범위에 달린다. SELinux는 보안 컨텍스트·타입과 정책으로 접근을 판단하며 경로의 권한만으로 설명되지 않는 거부를 추적해야 한다. Rocky에서는 SELinux를 유지하고 SSH 포트 정책을 맞췄다. [AppArmor](https://ubuntu.com/server/docs/how-to/security/apparmor/) · [Rocky SELinux](https://docs.rockylinux.org/guides/security/learning_selinux/) |
| UFW / firewalld | UFW는 allow/deny 중심의 간단한 호스트 방화벽 인터페이스라 소수 규칙을 관리하기 쉽다. firewalld는 zone과 서비스 정의로 인터페이스별 신뢰 범위를 나누기 좋지만 활성 zone과 runtime/permanent 차이를 함께 관리해야 한다. 과제의 Rocky 조건에 따라 firewalld를 사용하고 4242/tcp만 허용했다. 둘 중 하나가 무조건 더 안전한 것이 아니라 실제 규칙·적용 인터페이스가 중요하다. [UFW](https://ubuntu.com/server/docs/how-to/security/firewalls/) · [firewalld 개념](https://firewalld.org/documentation/concepts.html) |
| VirtualBox / UTM | VirtualBox는 이번 환경에서 VM·NAT 포워딩·스냅샷을 관리하는 도구이며 과제의 기본 선택이다. 지원되는 호스트·게스트 조합은 설치 버전에 맞춰 확인해야 한다. UTM은 Mac에서 QEMU 기반 에뮬레이션·가상화와 Apple Virtualization을 제공한다. 다른 아키텍처 에뮬레이션은 활용 범위가 넓지만 네이티브 가상화보다 느릴 수 있다. 과제에서는 VirtualBox를 사용할 수 없을 때의 대안이며 이번에 UTM을 실행 비교한 것은 아니다. [VirtualBox](https://www.virtualbox.org/) · [UTM](https://mac.getutm.app/) |

### 주요 설계와 서비스

- **디스크:** 비암호화 `/boot` 2 GiB 외에 LUKS 안에서 VG `LVM`의 root 10 GiB, home 4 GiB, `/var/log` 2 GiB, swap 4 GiB를 나눈다. 개인 파일과 로그 증가를 root 공간에서 분리하고 약 8 GiB의 VG 여유를 확장용으로 남긴다. 용량은 초기 예산이며 실제 사용량을 확인해야 한다. 로그 분리는 로그 순환·용량 관리를 대신하지 않는다.
- **사용자·비밀번호:** 일반 작업은 jungblee, 관리 작업은 sudo로 수행한다. jungblee는 user42·sudo와 Rocky 관리자 그룹 wheel에 속한다. 기간은 최소 2일·최대 30일·경고 7일, 품질은 길이 10 이상·대문자/소문자/숫자 포함·동일 문자 연속 최대 3·사용자명 제외·이전 비밀번호 차이 조건을 적용한다. root에도 품질을 적용하되 이전 비밀번호 비교 예외를 구분한다. 설정과 개별 거절 시험의 완료 범위는 학습 기록으로 구분한다.
- **sudo:** 인증 최대 3회와 사용자 정의 실패 메시지, TTY 요구, 제한된 실행 경로, `/var/log/sudo/` 입출력 기록으로 관리 명령의 권한과 추적성을 다룬다. 입력 로그에는 민감한 값이 남을 수 있어 로그 접근 권한을 관리한다.
- **서비스:** `sshd`는 일반 계정의 원격 관리, `firewalld`는 접근 포트 제한, `crond`는 root monitoring 예약 실행을 담당한다. SELinux는 별도 원격 접속 서비스가 아니라 접근 제어 계층이다. Bash·wall·sudo는 실행 도구이며 추가 웹 서비스를 설치하는 보너스는 현재 범위에 포함하지 않는다.
- **모니터링:** 구조·CPU·메모리·디스크·부팅 시각·LVM·연결·사용자·네트워크·sudo 기록을 12항목으로 출력한다. sudo 값은 현재 보관된 I/O 세션 수이므로 삭제된 과거 기록까지 포함한 영구 누적값으로 해석하지 않는다.

## Resources

과제 조건은 원문을 기준으로 하고 공식 문서는 개념과 도구를 이해하는 데 사용한다. 구간별 읽기 범위와 자료 선정 원칙은 [목표 문서](docs/targets.md#자료-선택과-읽는-깊이)에 모았다. Rocky 공식 입문서와 RHEL 10 문서의 제목·절은 2026-10-13에 대조했으며, 설치 패키지의 동작은 VM에서 별도로 확인한다.

- [42 Born2beRoot v5.2 공식 과제](https://cdn.intra.42.fr/pdf/pdf/197915/en.subject.pdf): 요구사항과 평가·제출 조건.
- [Rocky Linux — File System](https://docs.rockylinux.org/books/admin_guide/07-file-systems/): 디스크에서 파일시스템과 마운트까지의 입문 흐름.
- [RHEL 10 — Disk partitions](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/disk-partitions): 파티션 개요, GPT·MBR 구분, 마운트 지점.
- [RHEL 10 — Overview of logical volume management](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_logical_volumes/overview-of-logical-volume-management): PV·VG·LV의 관계와 LVM의 목적.
- [RHEL 10 — Encrypting block devices by using LUKS](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/encrypting-block-devices-by-using-luks): 첫 단위에서는 17.1절의 암호화 개념부터 읽는다.

### AI 활용 범위

AI는 한국어 번역·요구사항 정리 자료와 README·학습 기록·목표 문서의 문서화, 공식 자료 탐색 및 링크 확인에 활용했다. README의 OS·도구 비교와 설치·실행·설계 설명도 기존 실습 기록과 공식 자료를 바탕으로 AI가 정리했다. AI는 monitoring의 단계별 부분 예시와 설명을 준비하고 작업 환경에서 예시를 검사했다. VM 적용·실행은 사용자가 수행했으며, 전달받은 결과와 AI 작업 환경의 검사를 구분해 기록했다. AI가 사용자 VM을 설치하거나 보안 설정을 변경하지 않았다. 이후에도 스스로 판단하고 직접 적용한 결과를 바탕으로 설명을 점검하며, 실제 AI 사용 범위를 이곳에 갱신한다.
