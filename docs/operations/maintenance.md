# 프로세스·서비스·로그·업데이트·복구

개념부터 읽기: [패키지](../concepts/13-packages.md) · [서비스와 시간](../concepts/14-services.md) · [로그와 지표](../concepts/15-observability.md) · [백업](../concepts/16-backup.md)

[전체 안내서](../targets.md) · [부팅·저장장치](storage.md) · [계정·접속](access.md)

설정값과 monitoring 전체 소스는 [치트시트](../cheatsheet.md)에 유지한다. 여기서는 실행 상태·시간·로그·패키지 변경을 연결하고, 바뀐 상태를 되돌리는 범위를 판단한다. 예제는 운영 방법이며 실제 VM에서 새 검증을 완료했다는 의미가 아니다.

## 프로세스와 서비스는 같은 단위가 아니다

프로세스는 PID·부모·사용자 자격·열린 파일·메모리를 가진 실행 단위다. 프로그램 파일이 있어도 실행 중인 것은 아니며 같은 프로그램의 프로세스가 여러 개일 수 있다. systemd는 unit의 의존성과 수명으로 서비스를 관리하며 한 서비스에 여러 프로세스가 속할 수 있다. `ps`에서 PID 하나를 찾는 것, unit이 active인 것, 실제 요청을 처리하는 것은 각각 다른 관측이다.

예를 들어 sshd가 active여도 잘못된 포트만 열었을 수 있고 crond가 active여도 원하는 예약이 없을 수 있다. `enabled`는 부팅 등의 연결을 구성하는 상태이며 즉시 실행했다는 뜻이 아니다. `start`는 지금 시작, `enable`은 시작 경로 연결, `enable --now`는 둘을 함께 요청한다. unit 파일을 수정했다면 manager의 `daemon-reload`, 서비스 자체 설정을 수정했다면 지원되는 reload 또는 restart가 관련된다. `daemon-reload`는 서비스 프로세스를 재시작하거나 sshd_config를 그 프로세스에 다시 읽히는 명령이 아니다. [systemctl 공식 매뉴얼 소스](https://github.com/systemd/systemd/blob/main/man/systemctl.xml)의 해당 명령을 대조한다.

게스트에서 장애 난 서비스의 상태와 이번 부팅 로그부터 본다:

```bash
systemctl status sshd --no-pager
systemctl is-enabled sshd
systemctl cat sshd
sudo journalctl -b -u sshd --no-pager
ps -eo pid,ppid,user,stat,etime,args
```

오류 메시지의 종료 코드·실패한 경로·시각을 마지막 설정 변경과 연결한다. `active (exited)`도 unit 유형에 따라 정상일 수 있고, 실패 상태를 지우는 `reset-failed`는 원인 수리가 아니다. 프로세스 명령줄에 비밀이 들어갈 수도 있으므로 출력은 선별해서 공유한다.

**사례 — 설정 변경 뒤 서비스가 시작하지 못함:** 관리자 콘솔과 기존 설정 사본을 확보한다. `systemctl cat`으로 패키지 unit·drop-in 관계를 보고 서비스 전용 문법 검사와 로그를 읽는다. 잘못된 설정만 되돌리고, unit을 고쳤다면 daemon-reload 후 해당 서비스 재시작이 필요할 수 있다. 재시작은 세션·진행 작업에 영향을 줄 수 있으므로 유지보수 시간을 정한다. 복구 후에는 active뿐 아니라 새 SSH 접속 같은 실제 기능과 다음 부팅 연결을 각각 확인한다.

프로세스를 종료할 때는 서비스의 정상 stop 절차를 우선 검토한다. TERM은 프로그램이 정리할 기회를 주지만 KILL은 그 기회를 주지 않는다. 무조건 `kill -9`를 쓰면 미완료 쓰기나 파일 잠금 문제를 남기고 systemd가 다시 띄워 증상이 반복될 수 있다. cgroup은 서비스의 프로세스를 묶고 자원을 관리하는 개념이다. OOM·지속적인 CPU/메모리 경쟁이 생길 때 제한과 압력을 더 조사한다.

## cron의 시간과 실행 환경

cron은 로그인 셸을 그대로 복제하지 않는다. 실행 사용자, PATH, 작업 디렉터리, 환경변수, TTY 존재를 따로 생각해야 한다. 이 구성은 root 사용자 crontab에서 절대 경로의 `/bin/bash /usr/local/sbin/monitoring.sh`를 실행하며 PATH도 명시한다. 사용자 crontab에는 사용자명 열이 없고 `/etc/crontab` 형식과 섞으면 명령 해석이 달라진다.

crond는 crontab 변경을 감지하므로 `crontab -e` 저장마다 재시작하지 않는다. `*/10`은 시작 후 10분 대기가 아니라 매시 00·10·20·30·40·50분에 해당한다. 시간대·시계 변경은 예약 해석에 영향을 준다. `@reboot`는 일반 주기 항목과 다르며 이 Cronie 구성의 잠금 처리 때문에 단순 서비스 restart를 부팅 시험으로 대신하지 않는다. [Cronie crontab(5)](https://github.com/cronie-crond/cronie/blob/master/man/crontab.5)와 [기존 부팅 예약 설명](../targets.md#root-crontab의-읽기-범위)을 연결한다.

방송이 없을 때 확인 순서는 **crond 실행 → root의 예약 → 작업 시작·종료 → 스크립트 경로·권한·환경 → 출력 → 수신 터미널**이다.

```bash
systemctl status crond --no-pager
sudo crontab -l
sudo journalctl -b -u crond --no-pager
sudo journalctl -b -t CROND --no-pager
ls -l /usr/local/sbin/monitoring.sh
bash -n /usr/local/sbin/monitoring.sh
who
timedatectl
```

`bash -n`은 구문 검사일 뿐 경로·권한·도구의 실행 성공을 보장하지 않는다. CMD/CMDEND 기록은 작업 실행 흔적이지 모든 파이프 구성 명령의 성공 증명은 아니다. 현재 스크립트는 모든 실패를 중단시키는 오류 처리를 갖춘 프로그램으로 간주하지 않는다. 실패한 도구가 뒤의 awk·wc에 빈 입력을 주면 그럴듯한 값이 나올 수도 있어 원자료와 stderr를 함께 봐야 한다.

wall 수신은 실행과 별도 단계다. 부팅 순간 로그인한 터미널이 없으면 실행됐어도 사람이 화면에서 못 볼 수 있다. 수동 sudo I/O용 PTY 때문에 같은 화면에 중복이 보이는 경우도 있다. root cron의 실제 출력·여러 터미널 수신과 원자료 12항목을 대조한다. 수동 실행은 **실제로 모든 대상 터미널에 방송**하므로 조용한 조회가 아니다. 영향이 허용될 때만 실행한다.

중단은 monitoring의 두 예약 줄을 주석 처리하고 다른 root 예약·PATH를 보존한다. 복구는 같은 줄을 되살리고 다음 예정 시각의 실행과 수신을 확인한다. 스크립트를 지우거나 crond 전체를 중단하면 다른 작업까지 영향을 받는다. systemd timer는 의존성·누락 실행 처리 등이 필요할 때 알아볼 대안이지만 현재 cron 설계를 바꾸는 추가 과제가 아니다.

## 로그를 질문에 맞게 고른다

| 질문 | 먼저 볼 자료 | 그 자료만으로 알 수 없는 것 |
|---|---|---|
| 서비스가 왜 종료됐나 | unit 상태, `journalctl -b -u 서비스` | 실제 클라이언트가 원하는 동작을 받았는지 |
| 커널·디스크 I/O 문제가 있나 | `journalctl -b -k` | 특정 앱 설정의 정당성 |
| sudo로 무엇을 실행했나 | sudo 메시지·I/O 세션과 sudoreplay | 보관하지 않은 과거 전체 횟수 |
| SELinux가 거절했나 | audit AVC와 프로세스·대상 컨텍스트 | audit 기록 부재만으로 모든 거절 원인 배제 |
| cron이 방송했나 | CROND 작업 기록 + 실제 터미널 수신 | 작업 시작 기록만으로 모든 출력의 성공 |

journal은 구조화된 필드·부팅 ID·시간으로 범위를 좁힌다. 먼저 시각과 부팅을 맞추고 필요한 unit을 읽는다. 필터링한 오류 한 줄만 떼면 직전 원인을 놓칠 수 있다. root 권한의 읽기 명령도 sudo 로그를 추가할 수 있으며 운영 조회가 시스템에 아무 흔적도 안 남기는 것은 아니다.

journald의 `Storage` 설정에 따라 `/run/log/journal`의 휘발성 보관 또는 `/var/log/journal`의 디스크 보관 등이 달라진다. `auto`는 디렉터리 존재 조건도 관련되므로 이전 부팅 로그가 없다고 부팅하지 않았다고 결론 내리지 않는다. [journald.conf 공식 문서 소스](https://github.com/systemd/systemd/blob/main/man/journald.conf.xml)의 `Storage`, `SystemMaxUse`, `RuntimeMaxUse`를 읽는다. `/var/log`를 별도 LV로 둔 설계가 모든 로그를 영구 보관하도록 자동 설정하는 것은 아니다.

```bash
sudo journalctl --list-boots
sudo journalctl --disk-usage
df -h /var/log
sudo du -xhd1 /var/log
```

**사례 — 로그가 커져 관리 작업이 실패함:** [공간 계층](storage.md#저장-공간은-네-곳에서-따로-계산한다)에서 어떤 파일·서비스가 원인인지 찾고 증가 원인부터 고친다. 필요한 증거와 보존 기간을 정한 뒤 적합한 순환 방식을 선택한다. journald 보존 제한, 텍스트 로그의 logrotate, sudo I/O 세션은 같은 형식이 아니다. `journalctl --vacuum-*`는 과거 기록을 지우며 보존할 증거를 되돌릴 수 없다. sudo 세션 일부를 임의로 잘라 replay와 집계가 깨지게 하지 않는다.

logrotate는 설정·상태 파일을 읽어 지정한 파일의 회전·압축·보존을 수행한다. 예약 실행 주체는 설치 구성에 따라 timer나 cron이므로 따로 확인한다. 파일 이름만 바꿔도 서비스가 이전 파일을 계속 열고 있을 수 있어 로그 재열기 방식이 필요하다. `copytruncate`에는 복사와 자르기 사이 기록 손실 가능성이 있다. [logrotate 공식 매뉴얼](https://github.com/logrotate/logrotate/blob/main/logrotate.8.in)의 `DESCRIPTION`, `rotate`, `postrotate`, `copytruncate`를 읽고 대상 서비스에 맞춰 결정한다. 이 안내는 새 순환 정책이 이미 설치됐다는 뜻이 아니다.

모니터링은 특정 시점의 요약이고 journal은 사건의 흐름이다. CPU 한 번의 수치로 장기 부하를 단정하지 않으며, `free`의 available과 used/total, df의 여유와 VG 여유, who의 사용자와 세션 수를 구분한다. 현재 지표의 계산·한계는 [monitoring 설명](../targets.md#6-monitoringsh와-cronwall)에 모았다. 외부 시계열 수집·경보는 장기 추세가 필요할 때 확장한다.

## 패키지 업데이트도 구성 변경이다

DNF는 저장소 메타데이터와 의존성을 바탕으로 RPM 설치·갱신 트랜잭션을 계획한다. RPM 데이터베이스의 설치 버전과 지금 실행 중인 프로세스·커널 버전은 다를 수 있다. 새 커널 패키지가 설치됐다는 사실만으로 실행 커널이 바뀌지는 않는다. 기존 서비스도 이전 라이브러리를 사용 중일 수 있어 업데이트 후 재시작·재부팅 계획이 필요하다.

게스트에서 먼저 범위를 살핀다. 아래 DNF 조회는 필요에 따라 네트워크를 사용하고 메타데이터 캐시를 갱신할 수 있지만 패키지를 설치하지 않는다.

```bash
dnf repolist
dnf check-update
dnf history list
rpm -q openssh-server firewalld cronie kernel-core
uname -r
```

`dnf check-update` 종료 코드 100은 업데이트가 있다는 뜻이며 일반 실패와 다르다. 출처: [DNF 명령 참조](https://github.com/rpm-software-management/dnf/blob/master/doc/command_ref.rst)의 `Check-Update Command`, `History Command`. 설치된 DNF의 man과 대조하며 다른 메이저 버전 명령을 섞지 않는다.

**업데이트를 실행할 조건:** 복구 가능한 종료 상태 백업, 콘솔 접근, 호스트·게스트·`/boot` 공간, 중단 가능한 시간, 저장소 신뢰와 변경 범위를 확보한다. 서명 제출 후 업데이트는 VDI와 제출 기준을 바꾸므로 평가용 기준본과 개선 중인 상태를 혼동하지 않는다. 아래는 실제 패키지를 바꾸는 작업이며 자동 승인 `-y`를 붙이지 않는다.

```bash
sudo dnf upgrade
```

트랜잭션 목록에서 제거·새 의존성·커널 변경을 검토하고 진행 여부를 정한다. 저장소 오류를 해결하려고 서명 검사나 TLS 검증을 끄지 않는다. 업데이트 뒤 `.rpmnew`·`.rpmsave`가 생겼다면 새 배포판 기본값과 로컬 보안 정책을 비교해 필요한 부분만 병합한다. 파일을 이름만 바꿔 통째로 교체하지 않는다. 서비스 전용 문법 검사 → 필요한 서비스 반영 → 새 SSH 접속·방화벽·SELinux·예약 동작 → 계획된 재부팅 뒤 지속성 순서로 확인한다.

업데이트 뒤 부팅 실패라면 콘솔에서 이전 설치 커널로 부팅해 커널·initramfs 문제인지 좁힐 수 있다. 이전 커널이 실제로 남아 있는 경우에만 가능한 통로이며 보안 업데이트를 영구 회피하는 해법은 아니다. 패키지 history undo/rollback은 트랜잭션 역연산이고 필요한 패키지 가용성과 현재 RPMDB 상태에 의존한다. 데이터·모든 설정·VM 전체를 되돌리는 백업 대체물이 아니다. 원인이 좁혀지지 않거나 상태가 크게 변했다면 보존한 디스크 기준본으로 복구할 손실 범위를 판단한다.

[Rocky DNF](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/)의 `Update and upgrade packages`, `DNF list history`, `DNF repository configuration`을 읽는다. 일반 운영의 업데이트 권장과 과제의 최신 안정 OS·제출 해시 조건은 다른 판단이며 단순 `dnf upgrade` 성공만으로 두 조건을 모두 증명하지 않는다.

## 스냅샷·백업·서명의 역할

| 수단 | 보존하는 것과 사용 목적 | 복구의 한계 |
|---|---|---|
| 설정 파일 사본 | 작은 변경 이전의 값 | 패키지·사용자 데이터·디스크 전체는 되돌리지 못함 |
| VirtualBox 스냅샷 | VM 설정·디스크 상태, 실행 중 생성 시 메모리도 포함 가능 | 같은 호스트·기반 디스크에 의존. 독립 장애 대비 백업과 다름 |
| 종료된 VDI의 바이트 동일 백업 | 파일 내용·내부 UUID·해시 기준 | VM 설정을 별도로 보존해야 하며 이후 변경은 복원 시 잃음 |
| Full Clone | 독립 실행할 VM | 메타데이터·UUID 차이로 원본 파일 해시 보존 수단이 아님 |
| signature.txt | 제출 시 디스크 SHA-1 | 데이터 복구 기능 없음; 해시 일치와 VM 요구 충족도 별개 |

스냅샷 Restore는 되돌리기, Delete는 현재 상태를 유지하면서 필요한 차등 디스크를 병합할 수 있는 삭제다. 삭제가 취소 버튼은 아니다. 호스트 저장장치 자체가 고장 나면 같은 디스크의 백업도 잃을 수 있어 별도 저장장치 보관은 일반 운영에서 검토할 확장이다. 과제 원문 인쇄 16–17쪽은 평가 시작 시 스냅샷 없음, 이후 평가용 생성과 종료 시 삭제를 요구한다. 일반 운영에서 편한 스냅샷 보관을 평가 규칙에 그대로 적용하지 않는다.

실제 파일 복사·복원 명령과 주의 조건은 [기준 디스크 보존과 복원](../targets.md#평가-사이의-디스크-보존과-복원)에 한곳으로 둔다. 복구 전에는 **대상 VM·연결 파일·Powered Off·차등 디스크 없음·기준본과 되돌릴 변경**을 확인한다. 복구 후에는 디스크 해시 대조와 실제 기능 확인이 필요하지만 기능 확인을 위해 부팅하면 디스크가 다시 바뀔 수 있다. 반복 평가는 그 다음 평가 전 기준본 복원·재해시 단계까지 포함한다. 해시 불일치만 덮으려고 변경된 디스크의 값으로 제출 파일을 임의 교체하지 않는다.

## 읽는 깊이와 운영 질문

| 깊이 | 내용과 이유 | 읽을 절과 질문 |
|---|---|---|
| 직접 설명하고 다룬다 | PID·unit·enabled/active, 설정 적용 시점, 시간·사용자·PATH, 로그 범위, 백업과 복구 범위. 현재 세 서비스의 운영에 필요하다. | [Rocky 프로세스](https://docs.rockylinux.org/books/admin_guide/08-process/)의 프로세스·신호, [System Startup](https://docs.rockylinux.org/books/admin_guide/10-boot/)의 `Managing system services`, `The journald process`; “active인데 왜 접속이나 방송이 안 되는가?” |
| 직접 설명하고 다룬다 | 업데이트 전후 차이와 기능·부팅·서명 대조 | DNF의 위 절, [원문](../../en.subject.pdf) 7·16–17쪽; “무엇을 바꿨으며 어떤 사본으로 어디까지 되돌릴 수 있는가?” |
| 존재와 쓰임을 안다 | systemd timer, cgroup·OOM, journal 보존, logrotate, 시간 동기화, 별도 매체 백업 | 장기 운영·부하·디스크 증가·시계 오차가 생기면 해당 도구 man을 읽는다. “지금 수치의 시각과 기록의 보존 범위를 믿을 수 있는가?” |
| 지금 미룬다 | 중앙 로그·외부 대시보드·자동 패치 배포·컨테이너 오케스트레이션·HA | 현재 단일 VM보다 운영 범위가 커질 때 필요하다. 먼저 수동 변경·장애 분리·복원 경로를 설명할 수 있어야 한다. |
