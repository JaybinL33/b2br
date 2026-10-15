# Linux 디렉터리 지도와 b2br 경로 참조

[개념 지도](concepts.md) · 선행 개념: [파일·경로·마운트](concepts/06-files.md) · 실제 명령 순서: [치트시트](cheatsheet.md)

Linux의 디렉터리는 파일을 분류하는 관례이면서, 프로그램과 관리자가 서로의 자료를 찾는 약속이다. 위치를 이해하면 “어느 파일을 외워야 하는가”에서 “이것은 프로그램인가, 설정인가, 상태인가”로 질문을 바꿀 수 있다. 여기서는 경로의 역할·보존 기간·관리 주체를 함께 읽는다. inode·링크·마운트의 원리는 기존 파일 장에 두고, 이 문서는 그 원리가 실제 디렉터리 배치로 어떻게 나타나는지 설명한다.

**근거의 범위:** FHS 3.0은 일반 배치 원칙, systemd의 `file-hierarchy(7)`는 현대 systemd 시스템의 관례, Rocky 문서는 배포판 사용 맥락이다. FHS의 전통적 `/bin`·`/sbin` 구분과 현대 usr-merge를 같은 시대의 강제 배치로 섞지 않는다. 아래 **b2br 사례**는 저장소의 Rocky 10.2 구성·학습 기록과 대조한 경로다. 이 문서 작성 중 VM을 조회한 것은 아니며, 모든 선택 경로의 존재·마운트 유형·링크 목적지를 확인했다고 주장하지 않는다.

- [전체 구조와 읽는 축](#전체-구조와-읽는-축)
- [루트 아래의 경로 지도](#루트-아래의-경로-지도)
- [프로그램은 usr 설정은 etc](#프로그램은-usr-설정은-etc)
- [var는 모두 지워도 되는 임시 공간이 아니다](#var는-모두-지워도-되는-임시-공간이-아니다)
- [run tmp vartmp는 수명이 다르다](#run-tmp-vartmp는-수명이-다르다)
- [장치와 커널 정보를 파일 경로로 만난다](#장치와-커널-정보를-파일-경로로-만난다)
- [b2br에서 실제로 연결되는 경로](#b2br에서-실제로-연결되는-경로)
- [안전하게 구조를 관찰하기](#안전하게-구조를-관찰하기)
- [학습 우선순위와 이유](#학습-우선순위와-이유)
- [공식 근거와 더 읽기](#공식-근거와-더-읽기)

## 전체 구조와 읽는 축

`/`는 모든 절대 경로의 시작이다. 그 아래에 디렉터리가 있다고 모두 같은 디스크에 저장되는 것은 아니다. b2br은 `/`, `/home`, `/var/log`를 별도 LV의 ext4로 연결한다. `/proc`·`/sys`는 커널 정보를 제공하는 별도 파일시스템이다. 경로 트리는 하나지만 그 뒤의 저장 방식은 여러 개다.

```text
/                         하나의 경로 트리의 시작
├─ usr/                   배포된 프로그램·라이브러리·공유 자료
│  └─ local/              이 컴퓨터 관리자가 별도로 설치한 도구
├─ etc/                   이 컴퓨터의 설정·계정 정보
├─ var/                   운영하면서 바뀌고 보존할 데이터
│  ├─ lib/                서비스의 지속 상태
│  ├─ log/                사건·감사 기록
│  ├─ cache/              재생성 가능한 캐시
│  └─ spool/              처리할 작업·예약·큐
├─ run/                   이번 부팅·세션의 실행 상태
├─ tmp/                   짧게 사용하는 임시 작업 공간
├─ home/ · root/          일반 사용자 홈 · root 계정 홈
├─ boot/                  커널·초기 부팅 이미지·부트로더 자료
├─ dev/                   장치 인터페이스
├─ proc/ · sys/           프로세스·커널 정보 · 장치·커널 객체 정보
└─ opt/ · srv/ · mnt/ · media/   추가 소프트웨어·서비스 자료·마운트 위치
```

이 그림은 역할 지도이며 실제 `tree /` 출력이 아니다. `/bin`·`/sbin`·`/lib*` 같은 호환 경로는 아래에서 별도로 다룬다.

세 가지 축을 함께 보자. **역할**은 프로그램·설정·데이터·인터페이스 중 무엇인지다. **수명**은 재부팅 뒤 보존해야 하는지, 재생성하는지다. **관리 주체**는 패키지 관리자·시스템 관리자·서비스·사용자·커널 중 누가 정상적인 내용을 만드는지다. root 소유라는 사실만으로 관리자가 텍스트 편집기로 직접 수정해도 되는 파일이라는 뜻은 아니다.

“영속”도 영원한 보존을 보장하지 않는다. 디스크의 파일은 삭제·회전 정책으로 없어질 수 있다. 반대로 가상 파일시스템이라는 표현도 모두 같은 의미가 아니다. tmpfs에는 실제 파일 내용을 저장하지만 수명이 제한되고, procfs·sysfs는 커널 상태를 파일 인터페이스로 표현한다. VFS는 이들을 공통으로 다루는 커널 계층이므로 “VFS이기 때문에 메모리에만 저장된다”는 해석은 틀리다.

## 루트 아래의 경로 지도

표의 **필수**는 경로를 설명하고 실제 작업과 연결할 수준, **존재**는 역할을 알고 필요할 때 찾아볼 수준, **심화**는 내부 구조·운영 조건까지 이어 읽을 수준이다. 모든 경로를 당장 편집하거나 만들어야 한다는 뜻은 아니다. 일반 저장 경로라도 다른 파일시스템을 마운트할 수 있으며, 표의 유형은 보통의 역할을 나타낸다.

| 경로 | 역할과 구체적인 쓰임 | 보존·파일시스템 | 정상 관리 주체 | 우선순위 |
|---|---|---|---|---|
| `/` | 전체 경로의 시작. root 파일시스템 위에 다른 파일시스템을 연결 | b2br은 ext4, 영속 | 설치기·관리자 | 필수: 나머지 경로의 기준 |
| `/etc` | 이 호스트의 설정·계정 DB. sshd 설정과 fstab이 여기 있음 | 보통 일반 저장, 영속. 일부 파일은 생성 파일·링크 | 관리자와 전용 관리 도구 | 필수: 과제 설정 대부분 |
| `/usr` | 설치된 명령·라이브러리·공유 자료. 개인 사용자 홈이 아님 | 보통 일반 저장, 영속 | 주로 패키지 관리자 | 필수: 프로그램과 설정 구분 |
| `/bin`, `/sbin` | 전통적 기본 명령·관리 명령 경로. 현대 시스템에서는 `/usr` 쪽 호환 링크 | 링크 대상의 저장 방식 | 배포판·패키지 관리자 | 필수: usr-merge와 PATH |
| `/lib`, `/lib64` | 전통적 필수 라이브러리 경로. 아키텍처에 따라 변형, `/usr` 쪽 링크 관례 | 링크 대상의 저장 방식 | 배포판·패키지 관리자 | 필수 역할, ABI 내부는 심화 |
| `/usr/local` | 배포판 패키지와 구별해 현장 관리자가 설치한 도구 | 보통 일반 저장, 영속 | 시스템 관리자 | 필수: monitoring 배치 |
| `/var` | 실행 중 변하며 보존할 상태·로그·큐 등 | 보통 일반 저장, 영속 | 서비스·전용 도구, 관리자는 정책 | 필수: 데이터 성격에 따라 하위 구분 |
| `/run` | 부팅 이후의 PID 파일·소켓·런타임 상태 | 통상 tmpfs, 부팅마다 새로 준비 | systemd·서비스·세션 관리자 | 필수: 보존 상태와 구분 |
| `/tmp` | 프로그램의 단기 임시 파일 | 디스크 또는 tmpfs, 보존 보장 없음 | 사용자·앱, 정리 정책은 관리자 | 필수: 보관 장소로 착각하기 쉬움 |
| `/var/tmp` | 재부팅을 넘어 필요한 임시 작업 파일 | 보통 일반 저장, 재부팅 간 보존 원칙. 기간 정리 가능 | 사용자·앱·정리 정책 | 필수: `/tmp`와 비교 |
| `/home` | 일반 사용자의 홈을 두는 관례적 위치 | b2br은 별도 ext4 LV, 영속 | 계정 도구가 준비, 각 사용자 사용 | 필수: 개인 파일·숨김 설정 |
| `/root` | UID 0 계정의 홈. `/`와 다른 디렉터리 | 보통 root 파일시스템의 일반 저장, 영속 | root 계정 | 필수: 세 가지 root 구분 |
| `/boot` | 커널·initramfs·부트로더 자료 | b2br은 별도 비암호화 ext4, 영속 | 설치기·커널 패키지·부팅 도구 | 필수: 부팅과 디스크 설계 |
| `/boot/efi` | UEFI 시스템에서 ESP를 연결하는 일반 위치 | ESP는 보통 FAT 계열, 영속 | UEFI 부팅 도구·관리자 | 존재: 현재 BIOS 설계와 구분 |
| `/dev` | 디스크·터미널 같은 장치 노드와 연결 경로 | 통상 devtmpfs, 동적 생성. 하위에 devpts·tmpfs 등 | 커널·udev | 필수: 장치가 일반 파일이 아님 |
| `/proc` | 프로세스와 커널 정보를 파일처럼 조회 | procfs, 동적 인터페이스 | 커널 | 필수: monitoring의 정보 기반 |
| `/sys` | 장치·드라이버·커널 객체의 속성 | sysfs, 동적 인터페이스. 하위 별도 마운트 가능 | 커널·장치 관리 도구 | 필수 역할, 속성 조작은 심화 |
| `/opt` | 공급자·제품 단위로 묶은 추가 소프트웨어 | 보통 일반 저장, 영속 | 공급자 설치기·관리자 | 존재: `/usr/local`과 선택 방식 비교 |
| `/srv` | 이 시스템이 서비스로 제공하는 데이터 | 보통 일반 저장, 영속 | 관리자·해당 서비스 | 존재: 모든 서버 데이터의 강제 위치는 아님 |
| `/mnt` | 관리자가 임시로 파일시스템을 연결할 자리 | 연결한 파일시스템에 따라 다름 | 관리자 | 존재: 폴더 자체가 장치는 아님 |
| `/media` | 이동식 매체의 마운트 위치 관례 | 연결한 매체에 따라 다름 | 관리자·자동 마운트 도구 | 존재: 데스크톱은 `/run/media`도 사용 |
| `/lost+found` | ext 계열 파일시스템 복구 시 연결을 잃은 객체를 수습할 공간 | 해당 ext 파일시스템의 일반 저장 | 파일시스템 생성·복구 도구 | 존재: 임의 개인 보관함이 아님 |

FHS가 설명하는 선택 디렉터리가 모든 Minimal 설치에 존재하는 것은 아니다. 특히 GUI·개발 도구·별도 제품을 설치하지 않았다면 관련 하위 경로가 없어도 그 자체로 이상은 아니다. 일반 배치 근거는 [FHS 루트 계층](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03.html), 현대 배치 비교는 [systemd v257 file-hierarchy](https://github.com/systemd/systemd/blob/v257/man/file-hierarchy.xml)와 [Rocky 파일시스템 안내](https://docs.rockylinux.org/books/admin_guide/07-file-systems/)에서 읽는다.

## 프로그램은 usr 설정은 etc

### usr-merge와 bin·sbin·lib

전통적으로 `/bin`·`/sbin`·`/lib`에는 초기 부팅·복구에 필요한 파일을, `/usr` 아래에는 나머지 시스템 소프트웨어를 두었다. **usr-merge**는 실제 파일을 `/usr` 아래에 모으고 이전 경로를 심볼릭 링크로 호환시키는 배치다. `/bin/bash`와 `/usr/bin/bash`가 같은 대상을 가리킬 수 있으므로 경로 둘을 보았다고 Bash를 두 번 설치한 것은 아니다.

Rocky/RHEL 계열에서도 `/bin`·`/sbin`·`/lib*`의 `/usr` 연결 관례를 전제로 문서를 읽되, **이 VM의 정확한 링크 목적지는 아래 `readlink`로 확인할 항목**이다. `/usr/sbin`을 `/usr/bin`에 추가 통합하는 배포판 변화와 usr-merge 자체도 구분한다. 최신 systemd 문서의 모든 링크 예를 Rocky 10.2에서 확인한 사실로 바꾸지 않는다.

`bin`은 일반 명령, `sbin`은 시스템 관리 성격의 명령이라는 역사적 구분이다. sbin에 있다고 root만 실행할 수 있는 것은 아니다. 실행 파일 모드와 실제 작업 권한을 따로 판단한다. 라이브러리는 다른 프로그램이 사용하는 코드이며 `lib64` 같은 구분은 아키텍처·ABI와 연결된다. 직접 위치를 바꾸면 동적 로더가 필요한 파일을 못 찾을 수 있으므로 패키지의 배치를 따른다. [FHS bin](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s04.html) · [sbin](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s16.html) · [lib](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s09.html).

### usr의 하위 경로를 역할로 읽기

| 경로 | 내용과 관리 방식 |
|---|---|
| `/usr/bin`, `/usr/sbin` | 설치된 명령. DNF/RPM으로 설치한 파일은 패키지 갱신이 관리하므로 직접 덮어쓰지 않음 |
| `/usr/lib`, `/usr/lib64` | 라이브러리와 패키지 내부 자료. 모든 파일이 실행 가능한 라이브러리인 것은 아님 |
| `/usr/libexec` | 다른 프로그램이 호출하는 보조 실행 파일. 직접 실행하는 사용자 명령과 구분 |
| `/usr/share` | 아키텍처 독립 자료. `man/`, `doc/`, `licenses/`, `locale/`, `zoneinfo/` 등. “사용자끼리 공유 폴더”라는 뜻이 아님 |
| `/usr/include` | C 등에서 빌드할 때 사용하는 헤더. 실행 중 설정 파일이 아님 |
| `/usr/src` | 설치한 소스·개발 관련 자료. 모든 설치 프로그램의 소스가 자동으로 존재하지 않음 |
| `/usr/local/bin`, `/usr/local/sbin` | 관리자가 별도로 설치한 명령. b2br monitoring의 설치본은 뒤쪽에 위치 |
| `/usr/local/etc`, `/usr/local/share` | 로컬 설치 소프트웨어의 설정·공유 자료를 분리할 수 있는 위치. 모든 앱이 자동으로 읽는 것은 아님 |

`/usr/local`의 local은 “인터넷 연결이 없다”는 뜻이 아니라 **배포판 공급 파일과 현장 설치 파일의 책임을 나눈다**는 뜻이다. `/opt/제품명`은 제품 파일을 하나의 하위 트리에 모으는 다른 배치다. FHS는 그 제품의 호스트 설정과 변동 데이터를 `/etc/opt`·`/var/opt`와 연결한다. 어느 쪽이든 소프트웨어가 실제로 사용하는 경로를 따라야 한다. [FHS usr](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch04.html) · [local](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch04s09.html) · [libexec](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch04s07.html) · [opt](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s13.html).

### etc의 파일은 모두 직접 편집하는 설정이 아니다

`/etc`는 이 호스트가 어떤 방식으로 동작할지를 정하는 위치다. 패키지가 제공한 기본 설정, 관리자가 작성한 drop-in, 전용 도구가 생성한 계정·인증 파일이 함께 있다. `.d` 디렉터리는 설정을 여러 조각으로 나누는 흔한 관례일 뿐 **자동으로 읽히거나 항상 마지막 파일이 이긴다는 공통 규칙은 없다**. SSH의 Include·Match, PAM 인수, libpwquality의 읽기 순서는 각 도구의 규칙이다.

예를 들어 systemd의 공급자 unit은 `/usr/lib/systemd/system`, 지속적인 로컬 unit·override는 `/etc/systemd/system`, 임시 unit 설정은 `/run/systemd/system` 계열에 놓인다. 이 구분은 배포판 기본값과 현장 변경, 이번 부팅에만 필요한 변경을 분리한다. 그러나 generator 출력 등 다른 검색 경로도 있으므로 세 위치만이 전체 검색 순서라고 외우지 않는다. 실제 unit 경로는 `systemctl show`로 확인한다. [systemd unit 검색 경로](https://github.com/systemd/systemd/blob/v257/man/systemd.unit.xml) · [FHS etc](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s07.html).

## var는 모두 지워도 되는 임시 공간이 아니다

`/var`는 운영하면서 내용이 변한다는 분류다. 파일을 잃어도 되는지의 분류가 아니다. 서비스가 처리한 결과와 현재 상태, 아직 처리하지 않은 일, 다시 계산할 수 있는 사본은 복구 가치가 서로 다르다.

| 경로 | 무엇을 남기는가 | 잃었을 때의 의미·관리 주체 |
|---|---|---|
| `/var/log` | 무슨 일이 있었는지: 시스템·서비스·감사 기록 | 조사 근거 손실. 서비스가 기록하고 관리자가 보존·회전 정책을 설정 |
| `/var/lib` | 서비스가 다음 실행에도 이어 써야 할 지속 상태·DB | 서비스 데이터나 상태 손실 가능. 해당 서비스·DB 관리 도구로 관리 |
| `/var/cache` | 원본에서 다시 받을 수 있거나 계산해 만들 수 있는 캐시 | 재생성 비용·일시적 성능 영향. 임의 삭제 대신 해당 도구의 정리 기능과 정책 사용 |
| `/var/spool` | 나중에 처리할 작업·큐·예약 정보 | 미처리 메일·인쇄 작업·예약 등을 잃을 수 있음. 담당 스케줄러·서비스가 관리 |
| `/var/tmp` | 재부팅을 넘어 필요한 임시 작업 파일 | 영구 보관 목적은 아니지만 작업 중 삭제하면 손실. 정리 시점은 정책에 따름 |
| `/var/crash` | 설정된 경우 커널·프로그램 장애 덤프 | 원인 분석 자료. 설치·설정에 따라 없을 수 있고 내용은 민감할 수 있음 |
| `/var/mail` | 로컬 메일함 관례 | 사용자 메일 데이터. 모든 서버에 메일 서비스가 설치된 것은 아님 |
| `/var/run`, `/var/lock` | 현대 시스템에서 `/run` 계열로 연결되는 호환 경로 관례 | 이름이 var 아래라고 영속 저장으로 단정하지 않음 |

가령 DNF가 내려받은 패키지 캐시와 설치 상태 DB는 모두 패키지 관리와 관련되지만 대체 가능성이 다르다. 캐시 정리와 DB 삭제를 같은 작업으로 보면 안 된다. 패키지 DB의 실제 위치도 버전에 따라 달라질 수 있어 `/var/lib` 아래라고 고정하지 않고 `rpm --eval '%{_dbpath}'`로 조회한다.

b2br의 `/var/log/sudo`는 sudo I/O 세션 기록이다. 그 설정은 `/etc/sudoers.d/90-b2br`, 실행 프로그램은 시스템 명령 경로에 있다. 한 기능의 파일이 세 영역으로 나뉘는 이유는 설정·실행·기록의 수명이 다르기 때문이다. 또한 이 VM은 `/var/log`를 별도 LV로 두므로 로그가 많아질 때 `/var` 전체와 같은 저장 공간이라고 가정하면 안 된다.

journal은 `/var/log/journal`에 영속 저장하거나 `/run/log/journal`에 휘발 저장할 수 있다. 실제 경로·보존은 journald 설정과 디렉터리 준비 상태에 달린다. `/var/log/messages`, `/var/log/secure`, `/var/log/cron` 같은 텍스트 로그도 rsyslog 설치·설정에 따라 존재한다. 모든 Rocky 설치에서 모든 파일이 반드시 있다고 안내하지 않는다.

근거: FHS [cache](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch05s05.html), [lib](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch05s08.html), [log](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch05s10.html), [spool](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch05s14.html), [journald Storage](https://github.com/systemd/systemd/blob/v257/man/journald.conf.xml). 기록 원리와 복구는 [관측 장](concepts/15-observability.md)·[유지 관리](operations/maintenance.md)에서 이어진다.

## run tmp vartmp는 수명이 다르다

`/run`은 “현재 실행되고 있는 시스템의 상태”다. 서비스 소켓이나 PID 파일은 이번 부팅의 프로세스와 연결되므로 이전 부팅의 것을 그대로 남기면 오히려 잘못된 상태를 가리킬 수 있다. `/run/user/UID`는 사용자 세션의 런타임 공간이며 홈을 대체하지 않는다. `/run`이 휘발이라는 이유로 실행 중 마음대로 지워도 된다는 뜻은 아니다.

`/tmp`는 짧은 임시 작업용이다. **반드시 RAM에 있다**, **항상 부팅 때 전부 지워진다**는 식의 단정은 피한다. 디스크·tmpfs 마운트와 정리 정책을 확인해야 하며 앱은 재부팅 후 보존을 기대하지 않는다. `/var/tmp`는 재부팅을 넘어 필요한 임시 자료를 위한 위치지만 오래된 파일 정리 정책은 적용될 수 있다. 며칠 뒤에도 반드시 필요한 개인 문서는 홈이나 정한 영속 데이터 위치에 보관한다. 공유 임시 디렉터리에는 보통 sticky bit를 두어 남의 파일 삭제·이름 변경을 제한한다. 누구나 파일을 만들 수 있다는 것과 남의 파일 내용을 읽거나 삭제할 수 있다는 것은 서로 다른 권한이다.

서비스의 **PrivateTmp**는 해당 서비스가 보는 `/tmp`·`/var/tmp`를 다른 프로세스의 보기와 분리할 수 있는 systemd 실행 옵션이다. 서비스가 만든 임시 파일이 내 셸의 같은 경로에서 바로 보이지 않을 수 있는 이유다. 이는 `/tmp` 전체가 항상 RAM이라는 뜻도, 이 VM의 모든 서비스가 해당 옵션을 사용한다는 뜻도 아니다. [systemd 임시 디렉터리 설명](https://systemd.io/TEMPORARY_DIRECTORIES/).

systemd-tmpfiles는 설정에 따라 임시 디렉터리 생성·권한·기간 정리 등을 처리한다. 공급자 규칙은 `/usr/lib/tmpfiles.d`, 로컬 규칙은 `/etc/tmpfiles.d`, 런타임 규칙은 `/run/tmpfiles.d` 계열에서 만난다. “이름에 tmp가 있다”보다 **파일의 목적과 생성·삭제 정책**을 읽어야 한다. 여기서는 정리 명령을 실행하지 않는다. [FHS run](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s15.html) · [tmp](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s18.html) · [var/tmp](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch05s15.html) · [tmpfiles.d](https://github.com/systemd/systemd/blob/v257/man/tmpfiles.d.xml).

## 장치와 커널 정보를 파일 경로로 만난다

`/dev/sda` 같은 경로는 일반 문서가 아니라 디스크를 다루는 장치 노드다. `/dev/mapper`는 device-mapper의 이름 경로를 통해 LUKS 해제 장치나 LV를 만나게 한다. `/dev/disk/by-uuid`는 장치를 식별자로 찾는 링크 계열이다. UUID와 경로의 관계를 보더라도 장치를 열어 내용을 출력하거나 덮어쓰는 동작은 관찰 명령과 전혀 다르다.

터미널은 `/dev/tty*`·`/dev/pts/*`, 버리는 출력 통로는 `/dev/null`처럼 나타난다. `/dev/pts`는 PTY용 devpts, `/dev/shm`은 공유 메모리 객체를 위한 tmpfs가 일반적이다. `/dev` 아래에 있다고 모두 디스크 블록 장치인 것은 아니다.

`/proc/meminfo`·`/proc/stat`은 커널이 현재 정보를 제공하는 경로다. monitoring의 `free`·`vmstat` 등 도구는 이런 커널 통계를 사람이 읽기 쉬운 형태로 바꾼다. `/proc/PID`에는 개별 프로세스 정보가 있으나 환경·인자·열린 파일에는 민감한 정보가 들어갈 수 있다. 전체를 수집해 공유하는 방식으로 관찰하지 않는다.

`/sys/class`는 장치를 종류별로, `/sys/devices`는 장치 구조를 따라 살필 때 만나는 경로다. `/sys/fs/cgroup`에는 프로세스 자원 그룹을 다루는 별도 cgroup 파일시스템이 연결될 수 있다. `/proc/sys`와 `/sys`의 일부 파일은 **쓰면 커널 설정이 바뀌는 인터페이스**다. 텍스트처럼 보인다는 이유로 일반 설정 문서처럼 편집하지 않는다. 영속 sysctl 설정의 `/etc/sysctl.d`와 현재 값을 제공하는 `/proc/sys`도 다른 층이다. [Linux proc](https://www.kernel.org/doc/html/latest/filesystems/proc.html) · [sysfs](https://docs.kernel.org/filesystems/sysfs.html).

홈 안의 숨김 디렉터리도 역할을 나눈다. `~/.config`는 앱 설정, `~/.local/share`는 사용자 데이터, `~/.cache`는 캐시라는 XDG 관례다. `~/.ssh`는 SSH 키·접속 관련 자료이며, `~`는 현재 사용자의 홈이므로 root의 홈과 일반 사용자의 홈이 다르다. 모든 앱이 XDG 규칙을 따르는 것은 아니다. [XDG 기본 디렉터리 명세](https://specifications.freedesktop.org/basedir/latest/).

## b2br에서 실제로 연결되는 경로

아래 **사용 경로**는 기존 치트시트·스피드런·학습 기록의 경로를 대조한 것이다. **관련 경로**는 이해에 필요한 배포판 관례이며 이번에 그 존재나 내용을 VM에서 확인하지 않았다. 모두를 열어 복사할 필요는 없다.

| 기능 | 사용 경로와 관련 경로 | 역할·관리 경계 |
|---|---|---|
| 계정·그룹 | `/etc/passwd`, `/etc/group`; 제한된 `/etc/shadow`, `/etc/gshadow` | 계정·그룹 식별 정보와 비밀번호 해시·기간 등을 분리. passwd라는 이름이어도 현대 shadow 구성의 평문 비밀번호 목록이 아님. useradd·usermod·passwd·chage 같은 도구가 갱신 |
| 기본 기간 | `/etc/login.defs`; 관련 `/etc/default/useradd`, `/etc/skel` | 새 계정 기본값과 홈에 복사할 초기 파일. 기존 계정 상태와 다름. 실제 b2br 기간 기본값은 login.defs에서 설정 |
| PAM | `/etc/pam.d/passwd`, `/etc/pam.d/system-auth`; 관련 `password-auth`, `sudo`, `sshd` | 프로그램의 PAM 진입점·공통 스택. authselect 관리 파일은 일반 수동 설정처럼 수정하지 않음 |
| 인증 구성 관리 | `/etc/authselect`; 공급자 프로필 `/usr/share/authselect/default`, 로컬 프로필 `/etc/authselect/custom` | 프로필·생성 결과·사용 지점의 관계. 시스템의 현재 선택은 authselect 조회가 기준이며 이 설명은 프로필 변경을 요구하지 않음 |
| 비밀번호 품질 | `/etc/security/pwquality.conf.d/90-local.conf`; 관련 `/etc/security/pwquality.conf` | b2br의 로컬 품질 조건. PAM 인수·본체·drop-in의 우선순위를 함께 읽음. `/etc/security`에는 limits 등 다른 보안 설정도 있어 한 정책 파일 전체와 같지 않음 |
| sudo | `/etc/sudoers`, `/etc/sudoers.d/90-b2br` → `/var/log/sudo` | 허용·환경·기록 정책과 실제 I/O 세션 기록을 분리. 정책은 visudo 관리 대상, 로그는 sudo 생성·sudoreplay 조회 대상 |
| SSH | `/etc/ssh/sshd_config`, `/etc/ssh/sshd_config.d/00-b2br.conf` | 서버의 4242·root 로그인 정책. 클라이언트 `/etc/ssh/ssh_config`·`~/.ssh/config`와 다른 역할 |
| SSH 신원 | 관련 `/etc/ssh/ssh_host_*_key`, `*.pub`; 사용자 `~/.ssh/known_hosts`, `authorized_keys` | 서버 개인키·공개키, 기억한 서버 키, 허용한 사용자 공개키를 구분. 개인키 내용을 출력·공유하지 않음. authorized_keys가 현재 사용된다는 뜻도 아님 |
| 방화벽 | 관련 `/etc/firewalld/zones`, `/usr/lib/firewalld/zones`·`services` | 로컬 영구 설정과 공급자 정의. b2br은 firewall-cmd로 public zone을 관리. 현재 runtime 정책이 파일 내용과 항상 같은 것은 아님 |
| SELinux | 관련 `/etc/selinux/config`, `/etc/selinux/targeted`, `/var/lib/selinux` | 기본 모드·정책 자료·정책 저장소를 구분. 포트 정책은 semanage 같은 도구로 관리. 파일의 SELinux 레이블은 확장 속성에 있을 수 있어 이 폴더의 파일 하나가 모든 현재 레이블인 것은 아님 |
| 파일시스템·암호화 | `/etc/fstab`, 관련 `/etc/crypttab` | fstab은 파일시스템 마운트·swap 설정, crypttab은 암호화 장치 연결 정보. 암호화 해제→LV→마운트는 서로 다른 단계. 실제 초기 부팅은 initramfs와도 연결 |
| 부팅 | `/boot/vmlinuz-*`, `/boot/initramfs-*`, Rocky 계열의 `/boot/grub2`, 관련 `/boot/loader/entries` | 커널·초기 환경·부팅 항목을 패키지와 부팅 도구가 관리. BIOS Boot 파티션은 `/boot` 디렉터리나 UEFI ESP와 다름 |
| 서비스 | `/usr/lib/systemd/system/{sshd,firewalld,crond}.service`, 관련 `/etc/systemd/system` | 패키지 unit과 로컬 변경. 활성화 연결과 실행 중 상태도 구분하며 파일 존재만으로 실행 여부를 판단하지 않음 |
| cron | root의 사용자 crontab; Cronie 관례 `/var/spool/cron/root` | b2br 예약은 root crontab을 통해 관리. spool 파일을 직접 편집하지 않음. `/etc/crontab`·`/etc/cron.d`는 사용자 칸이 있는 시스템 형식이며 같은 파일이 아님 |
| 주기 작업 디렉터리 | 관련 `/etc/cron.hourly`, `/etc/cron.daily`, `/etc/cron.weekly`, `/etc/cron.monthly`, `/etc/anacrontab` | 설치·배포판 구성에 따라 run-parts·anacron 등이 실행. 이름만으로 정확한 실행 시각을 보장하지 않으며 현재 monitoring의 설치 위치도 아님 |
| monitoring | 작업 중 `monitoring.sh` → 설치본 `/usr/local/sbin/monitoring.sh` → root cron → wall | 상대 경로 작업본의 실제 위치는 현재 디렉터리에 달림. 운영 기준은 root 소유 755 설치본이며 홈의 작업본만 바꾸면 설치본이 자동 변경되지 않음 |
| 이름·네트워크 | `/etc/hostname`; 관련 `/etc/hosts`, `/etc/resolv.conf`, `/etc/NetworkManager/system-connections` | 로컬 이름, 이름 조회, 연결 프로필을 구분. NetworkManager가 생성·관리하는 파일을 무조건 직접 덮어쓰지 않음. 연결 프로필은 비밀을 포함할 수 있음 |

이 표는 설정값을 새로 제안하지 않는다. 정확한 본문·적용 순서는 [치트시트](cheatsheet.md), 원리와 정책 충돌은 [PAM 장](concepts/10-authentication.md)·[targets](targets.md), 현재 확인 이력은 [학습 기록](learning-log.md)을 따른다. 특히 `crypttab`, SSH 개인키, shadow 계열, 연결 프로필, sudo I/O 로그는 내용을 공유하기 위한 목록이 아니다.

## 안전하게 구조를 관찰하기

다음은 **사용자가 필요할 때 VM에서 실행할 수 있는 읽기 중심 예시**다. 이번 문서 작업에서는 실행하지 않았다. 파일 내용 대신 경로·종류·마운트·패키지 소유·권한부터 본다. 접근 거절은 자동으로 sudo를 붙여 전체를 출력하라는 뜻이 아니다. 조회도 접근 시각·감사 로그 등에 흔적을 남길 수 있으므로 제출 디스크의 바이트 불변을 보장하는 방법은 아니다.

```bash
# 디렉터리 자체와 링크를 본다. 내부 파일 전체를 재귀 출력하지 않는다.
ls -ld / /etc /usr /var /run /tmp /var/tmp /home /root /boot /dev /proc /sys
ls -ld /bin /sbin /lib /lib64 /usr/bin /usr/sbin
readlink -f /bin /sbin /lib /lib64

# 해당 경로를 실제로 담는 파일시스템: 이름과 저장 방식을 분리한다.
findmnt -T /etc
findmnt -T /var/log
findmnt -T /run
findmnt -T /tmp
findmnt -T /proc
findmnt -T /sys

# 실행 파일 위치, 패키지 소유, 로컬 설치본의 메타데이터
command -v bash ssh sudo
rpm -qf /usr/bin/bash
rpm --eval '%{_dbpath}'
stat -c '%F %U:%G %a %n' /usr/local/sbin/monitoring.sh
ls -ld /etc/pam.d /etc/security /etc/sudoers.d /etc/ssh

# unit 내용 대신 공급 파일·drop-in 경로만 확인한다.
systemctl show sshd.service -p FragmentPath -p DropInPaths
```

`findmnt -T /etc`가 `/`의 파일시스템을 보여 주면 `/etc`는 별도 마운트가 아니라 그 안의 경로라는 뜻이다. 반면 `/var/log`의 별도 LV가 보이면 기록 공간이 분리된 것이다. `readlink -f`는 최종 경로를, `ls -ld`는 링크 자체의 연결을 읽는 데 쓴다. `rpm -qf`가 로컬 스크립트를 패키지 소유로 찾지 못한다고 곧 악성 파일이라는 뜻은 아니다. monitoring은 관리자가 설치한 파일이라는 출처가 있기 때문이다.

결과를 공유할 때도 필요한 경로·파일시스템 유형·권한만 발췌한다. `cat /etc/shadow`, 개인키 출력, `/proc/*/environ` 수집, 홈·`/etc` 전체 압축, sudo 세션의 무차별 재생은 이 관찰 절차에 포함하지 않는다. 스크린샷에 사용자명·호스트명·장치 식별자가 보이는지도 필요한 범위로 확인한다.

## 학습 우선순위와 이유

| 깊이 | 지금 설명할 수 있어야 하는 범위 | 이유·다음 단계 |
|---|---|---|
| 필수 이해 | `/`·`/root`·`/home` 차이, usr·etc·var·run 분리, tmp·var/tmp의 수명, dev·proc·sys의 인터페이스 성격 | 잘못된 대상 수정·보존 착각을 막고 경로만 보고도 다음 질문을 정할 수 있음 |
| 필수 이해 | b2br의 PAM·sudo·SSH 설정, monitoring 설치본·root crontab·sudo 로그, fstab과 별도 `/home`·`/var/log` | 이미 적용한 과제 구성을 경로와 역할로 설명해야 함. 표를 외우는 것보다 한 기능의 실행→설정→상태를 따라가기 |
| 존재만 알기 | opt·srv·media·mnt·lost+found, usr/include·src·libexec, XDG 사용자 경로, UEFI의 boot/efi | 현재 Minimal 실습에 모두 쓰이지 않지만 나중에 설치·복구·개발 자료를 찾을 위치를 알 수 있음 |
| 심화 | unit 검색 순서·generator, tmpfiles 정책, initramfs 안의 설정 사본, 라이브러리 ABI, proc/sys 쓰기 인터페이스·cgroup | “파일을 바꿨는데 왜 동작이 다른가” 또는 “재부팅 후 왜 달라졌는가”를 설명할 때 필요. [부팅](concepts/08-boot.md)·[서비스](concepts/14-services.md)로 연결 |

예를 들어 SSH를 복습할 때 전체 디렉터리를 암기하지 않고 `/usr/sbin/sshd`라는 프로그램, `/etc/ssh`의 서버 설정, systemd unit, 서버 키와 사용자 키, 로그의 관계를 연결해 본다. monitoring은 `/usr/local/sbin`의 실행본과 `/var/spool`의 예약 정보, `/proc` 기반 통계, `/var/log/sudo`의 집계 자료를 따라간다. 직접 관찰한 사실이 생겼을 때만 학습 기록에 반영한다.

## 공식 근거와 더 읽기

- [FHS 3.0](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html): 역할·배치·보존 원칙. 위 절의 세부 링크로 필요한 경로를 읽는다.
- [systemd v257 file-hierarchy 원문](https://github.com/systemd/systemd/blob/v257/man/file-hierarchy.xml): 현대 배치와 전통적 경로의 호환 관계를 비교하기 위한 고정 버전 자료. 이 VM의 systemd 버전을 새로 확인했다는 뜻은 아니다. [최신 원문](https://github.com/systemd/systemd/blob/main/man/file-hierarchy.xml)은 현재 [UAPI 파일 계층 명세](https://uapi-group.org/specifications/specs/linux_file_system_hierarchy/)로 연결하므로 버전 차이를 구분한다. UAPI.9는 현재 0.1 작업 중 명세이며 `/opt`·`libexec` 등에서 FHS와 다른 권고도 있으므로 미래 방향을 Rocky 10.2의 현재 배치로 적용하지 않는다.
- [Rocky 파일시스템과 마운트](https://docs.rockylinux.org/books/admin_guide/07-file-systems/): 배포판 입문. 일반 설명을 해당 VM의 전체 파일 목록으로 취급하지 않는다.
- [RHEL 10 authselect](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_authentication_and_authorization_in_rhel/configuring-user-authentication-using-authselect): PAM 생성 파일의 관리 경계. [firewalld 영구 설정](https://firewalld.org/documentation/configuration/runtime-versus-permanent.html)·[Cronie crontab](https://github.com/cronie-crond/cronie/blob/master/man/crontab.5): 공급자·로컬 정의와 사용자·시스템 예약의 차이.
- [OpenSSH 서버 설정](https://man.openbsd.org/sshd_config)·[sshd 파일](https://man.openbsd.org/sshd): 키와 설정의 역할. 실제 Rocky 옵션·파일 배치는 설치된 매뉴얼과 패키지 조회가 최종 기준이다.

이 문서는 설명·경로 대조 결과다. VM 설치·설정 변경·로그 조회·비밀 파일 열람·복구 검증을 수행한 기록은 아니다.
