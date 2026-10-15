# 사용자·권한·원격 접속 운영

[전체 안내서](../targets.md) · [부팅·저장장치](storage.md) · [서비스 운영](maintenance.md)

이 문서의 예시는 jungblee, SSH 4242, enp0s3/public, 호스트 포워딩 `127.42.42.42:4242` 구성에 맞춘다. 설정 원문은 [치트시트](../cheatsheet.md)에 두며, 다른 장치·zone에 그대로 적용하지 않는다.

## 접근은 여러 번 판단된다

계정은 이름뿐 아니라 UID와 GID로 식별된다. 프로세스는 실행 사용자의 자격으로 파일·소켓에 접근한다. 파일 소유권과 모드가 허용하더라도 SELinux 정책이나 읽기 전용 마운트가 쓰기를 막을 수 있다. sudo는 허용된 명령을 다른 사용자 자격으로 시작하는 경로이고, PAM은 프로그램이 인증·계정·비밀번호·세션 처리를 연결하는 틀이다. 이 네 역할을 구분하면 `Permission denied`를 모두 chmod로 해결하려 하지 않게 된다.

파일의 `r/w/x`는 읽기·쓰기·실행, 디렉터리의 `x`는 경로 통과, `r`은 항목 이름 열람, `w`는 항목 생성·삭제와 연결된다. 파일을 삭제할 수 있는지는 파일 자신의 쓰기 비트만으로 결정되지 않는다. 상위 디렉터리 권한·sticky bit도 영향을 준다. 소유자·그룹·기타 비트를 읽고, 파일에 도달하는 모든 부모 경로를 따라간다. [Rocky User Management](https://docs.rockylinux.org/books/admin_guide/06-users/)의 `File owners`와 [File System](https://docs.rockylinux.org/books/admin_guide/07-file-systems/)의 `Basic permissions of files and directories`를 연결한다.

**사례 — cron은 되는데 일반 사용자 실행이 실패한다:** 먼저 어느 명령·경로에서 거절됐는지 찾는다. monitoring의 sudo I/O 목록 조회는 root 권한을 전제로 하므로 스크립트 파일이 755라고 내부 데이터까지 일반 사용자에게 읽히는 것은 아니다. 로그를 공개하거나 스크립트에 sudo를 중첩하기보다 실행 주체를 맞춘다.

게스트에서 읽기 중심으로 조사한다:

```bash
id
id jungblee
namei -l /usr/local/sbin/monitoring.sh
ls -ld /usr/local /usr/local/sbin
ls -lZ /usr/local/sbin/monitoring.sh
findmnt /var/log
```

`id`는 현재 프로세스의 그룹, `id jungblee`는 계정 조회 결과다. 그룹 추가 후 기존 로그인 프로세스가 자동으로 새 보조 그룹을 받는다고 가정하지 않는다. 새 로그인에서 확인한다. 소유권·모드 오류가 확인되면 해당 파일만 의도한 값으로 복구한다. `chmod -R 777`·광범위한 `chown -R`은 다른 설정과 보안 경계를 훼손하므로 해결책으로 사용하지 않는다. ACL의 추가 권한, setgid 디렉터리의 그룹 상속, umask의 새 파일 기본 제한은 공동 디렉터리가 필요할 때 찾아볼 확장 개념이다.

## 비밀번호 변경과 로그인 실패를 분리한다

`passwd`는 PAM의 password 스택을 통해 품질 검사와 실제 저장을 이어 간다. 현재 구성은 authselect local이 생성한 system-auth에 pam_pwquality가 연결된 형태다. pwquality drop-in은 본체보다 먼저 읽히고 PAM 인수가 우선한다. 따라서 파일 이름이 90으로 시작한다는 이유만으로 최우선이라고 판단하지 않는다. 정확한 정책과 root 예외는 [비밀번호·PAM](../targets.md#3-비밀번호pam)에 한곳으로 모았다.

`login.defs`의 기간 기본값은 새 계정을 만드는 도구가 사용하는 값이다. 기존 계정의 기간은 shadow에 저장되어 `chage`로 다루며, 품질 정책을 추가해도 기존 해시가 새 비밀번호로 바뀌지 않는다. 새 구축에서는 품질 설정 → jungblee 자신의 `passwd` → root 비밀번호 갱신 → 기존 계정 2/30/7 적용 순서로 같은 날의 최소 간격 충돌을 피한다. 운영 중 정책을 시험하려고 실제 계정의 최소 간격을 무조건 0으로 낮추지는 않는다.

| 관측 | 조사할 경계 | 복구 후 확인 |
|---|---|---|
| `passwd`가 변경을 너무 이르다고 거절 | 계정별 최소 변경 간격과 마지막 변경일 | 승인된 변경 시점·대상을 정하고 정상 갱신 뒤 원래 기간 정책을 대조 |
| 품질 오류 후 비밀번호 미변경 | pwquality 설정·PAM 인수·이전 암호 비교 경로 | 비밀번호를 노출하지 않고 정상 변경과 필요한 거절 조건을 별도로 확인 |
| 암호는 맞지만 로그인 거절 | 계정 만료·잠금·셸·서비스 PAM account 처리 | 해당 계정 상태를 고치고 새 로그인 확인; 무작정 해시나 PAM 제거 금지 |
| authselect 구성 불일치 | 생성 파일 직접 편집·프로필과 파일의 차이 | 원본과 변경분을 보존하고 관리 방식에 맞게 복구; `select --force`로 일괄 덮어쓰지 않음 |

조회는 필요한 계정·서비스만 좁힌다. shadow 원문과 암호를 증거에 붙이지 않는다.

```bash
sudo chage -l jungblee
sudo passwd -S jungblee
getent passwd jungblee
authselect current
sudo authselect check
sudo journalctl -b -u sshd --no-pager
```

sudo 인증과 변경 대상의 이전 비밀번호 입력은 다르다. `sudo passwd jungblee`는 관리자 대리 변경이어서 이전 비밀번호 비교를 생략한다. `enforce_for_root`는 품질 거절을 root에도 적용하지만 이 비교 생략을 되돌리지 않는다. 복구 때문에 관리자 재설정을 했다면 원래 사용자 자신의 비교 경로를 검증한 것으로 기록하지 않는다.

## sudo 설정을 읽는 시점과 복구 통로

sudoers는 sudo가 정책을 판단할 때 사용한다. 별도 데몬 재시작 없이 다음 호출에 반영되므로 잘못된 설정은 바로 관리자 작업을 막을 수 있다. 이 구성은 wheel 권한과 기본 secure_path를 유지하고 별도 Defaults 파일에 TTY·3회 제한·오류 문구·입출력 기록을 둔다. 그룹 이름 `sudo` 가입 자체와 sudoers의 허용 규칙은 별개다. secure_path는 실행 환경의 PATH이며 절대 경로 명령 전체를 허용/금지하는 목록이 아니다.

변경은 기존 관리 세션과 VM 콘솔 접근을 확보한 상태에서 `visudo`로 수행한다. `sudo visudo -c`, `sudo -l`은 각각 파서와 해당 사용자 정책을 보는 검사이며 실제 거절·입출력 기록 검증을 모두 대신하지 않는다. 파일이 잘못되어 sudo 자체가 막혔다면 이미 확보한 root 콘솔에서 해당 변경을 되돌린 뒤 파싱과 새 일반 사용자 sudo를 확인한다. root 셸도 없다면 [별도 복구 환경](storage.md#전원을-켠-뒤-셸까지)을 준비하며 무작정 재부팅해 남은 관리 통로를 없애지 않는다.

`requiretty`는 TTY 없는 호출을 막고, sudo의 I/O용 PTY는 입출력 기록과 연결된다. TTY 요구와 SSH 서버의 root 로그인 금지는 서로 다른 조건이다. sudo 로그 재생에는 민감한 입력이 포함될 수 있으므로 필요한 세션만 조사한다. 횟수 제한을 시험하기 전에 PAM 잠금 모듈이 있는지 확인하고 실제 계정으로 고의 실패를 반복하지 않는다.

## SSH가 안 될 때 경계를 하나씩 통과하기

```text
호스트 ssh 클라이언트 → 호스트 127.42.42.42:4242의 NAT 전달
 → 게스트 enp0s3 / firewalld의 활성 zone → sshd의 4242 리스닝
 → 서버 호스트 키 신뢰 → 사용자 인증·계정 허용 → 셸·파일 접근
```

SELinux는 이 경로의 단일 네트워크 홉이 아니다. sshd의 포트 바인딩과 파일 접근 등에서 별도로 정책을 적용한다. NAT 포트 전달은 호스트와 게스트의 포트를 이어 주며 게스트 방화벽을 대신하지 않는다. loopback 호스트 주소는 그 호스트 안에서 접속하는 경로다. 외부 PC에서 이 loopback 주소를 사용해 같은 VM에 도달할 수는 없다.

DHCP는 게스트 주소·라우팅 정보를 제공하고 DNS는 이름을 주소로 해석한다. 숫자 loopback 주소로 SSH하는 데 DNS는 직접 필요 없지만 `dnf`의 저장소 이름 해석에는 필요하다. 호스트 포트와 게스트 포트를 다르게 매핑할 수도 있으나 현재는 둘 다 4242다. IP 연결과 호스트명 설정은 별개다.

| 증상 | 먼저 보는 것 | 다음 분기 |
|---|---|---|
| 시간 초과 | VM 부팅·대상 주소·NAT 전달·방화벽 경로 | 게스트 콘솔에서 sshd 리스닝을 확인하고 패킷 경로를 좁힘. timeout만으로 방화벽 단정 금지 |
| Connection refused | 목적지 리스닝과 전달 대상 | 서비스 실패·잘못된 포트·명시적 reject 가능성. 비밀번호 정책부터 바꾸지 않음 |
| 호스트 키 경고 | 서버 정체성 | 재설치·주소 재사용인지 독립 콘솔 지문과 대조. known_hosts 삭제로 무조건 통과하지 않음 |
| Permission denied | 사용자·인증 방법·계정 정책 | 도달한 sshd의 로그와 유효 설정 대조. root 거절은 의도된 정책일 수 있음 |
| 접속 후 특정 파일만 거절 | 파일 권한·SELinux·마운트 | 네트워크 대신 경로·사용자 자격·AVC 기록으로 조사 |

게스트 콘솔에서 조회·검사한다. `public`은 이 구성의 zone이며 활성 연결이 달라졌다면 해당 zone을 조회한다.

```bash
ip -br address
ip route
sudo ss -lntp
systemctl status sshd --no-pager
sudo sshd -t
sudo sshd -T
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --list-all
sudo firewall-cmd --permanent --zone=public --list-all
sudo journalctl -b -u sshd --no-pager
```

`sshd -t`는 문법·키 관련 유효성, `-T`는 유효 설정을 본다. Match 조건이 있으면 실제 사용자·접속 조건을 `-C`로 제공해야 그 조건의 값을 대조할 수 있다. 본체의 Include 위치와 먼저 얻은 값의 우선순위를 살피고, 여러 Port가 누적될 수 있다는 예외도 구분한다. 설정은 시작·재읽기 때 sshd가 읽는다. 문법·SELinux·방화벽을 준비한 후 반영하고 기존 세션을 닫기 전에 **새 클라이언트 접속**으로 확인한다. reload/restart는 실제 운영 변경이며 restart를 문법 검사처럼 사용하지 않는다. [OpenSSH sshd_config](https://man.openbsd.org/sshd_config)의 `Include`, `Match`, `Port`, `PermitRootLogin`과 [sshd](https://man.openbsd.org/sshd)의 `-t`, `-T`, `-C`를 읽는다.

## 방화벽과 SELinux를 끄지 않고 복구한다

firewalld runtime은 지금 커널에 적용한 규칙, permanent는 다음 시작·reload 때 사용할 구성이다. runtime만 바꾸면 reload 후 사라지고 permanent만 바꾸면 현재 규칙에는 아직 반영되지 않는다. 연결된 zone과 서비스로 열린 포트를 함께 봐야 하며 `ports`가 비었다고 모든 포트가 닫힌 것은 아니다. [공식 runtime/permanent 설명](https://firewalld.org/documentation/configuration/runtime-versus-permanent.html).

**사례 — reload 뒤 SSH가 끊김:** 콘솔에서 active zone과 runtime/permanent 차이를 확인한다. 의도한 4242 허용이 permanent에 빠졌는지, 다른 zone을 수정했는지 찾는다. 원인을 고친 뒤 reload의 영향을 감수할 통로를 확보하고 반영한다. `--runtime-to-permanent`는 현재 전체 구성을 저장하므로 임시 허용까지 굳히는 일이 없는지 확인해야 한다. 방화벽 전체 중단을 정상 복구 경로로 삼지 않는다.

SELinux에서 프로세스·파일의 컨텍스트와 타입, 포트의 타입 연결은 별도 정보다. Enforcing 상태에서 SSH의 4242가 ssh_port_t에 연결되지 않으면 포트 바인딩이 거절될 수 있다. 파일을 옮겨 생긴 라벨 문제는 포트 등록으로 고쳐지지 않는다. `ls -Z`·프로세스 컨텍스트·AVC에서 거절된 작업과 대상을 짝지어야 한다.

```bash
getenforce
sudo semanage port -l
ps -eZ
sudo ausearch -m AVC,USER_AVC -ts recent
ls -lZ /usr/local/sbin/monitoring.sh
sudo restorecon -n -v /usr/local/sbin/monitoring.sh
```

`ausearch`는 audit 도구·기록을 사용할 수 있을 때 조사하며 기록이 없다고 SELinux를 무조건 배제하지 않는다. `restorecon -n`은 변경 예상만 보여 준다. 기본 정책상 라벨이 맞는지 확인한 뒤 **그 파일의 라벨을 실제 복구할 때만** `-n`을 뺀다. 다른 포트 타입이 이미 4242를 사용한다면 용도를 조사하지 않고 `semanage -m`으로 덮지 않는다. AVC만 보고 자동 허용 모듈을 생성하거나 Enforcing을 끄는 대신 잘못된 경로·라벨·포트·서비스 설정을 먼저 고친다. [RHEL 10 SELinux 진단](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_selinux/troubleshooting-problems-related-to-selinux)의 `Identifying SELinux denials`와 거절 원인 수정 부분을 읽는다.

## 호스트명과 세션의 수명

`hostname`은 현재 커널 이름, `/etc/hostname`은 정적 저장값이다. 파일 직접 편집은 실행 중인 이름을 바로 바꾸는 것과 다르다. `hostnamectl set-hostname`은 현재·저장값을 함께 다루며, 부팅 때 systemd가 정적 이름을 적용하되 커널 인수 등 우선 설정도 있다. 이미 실행된 셸의 프롬프트에는 이전 이름이 남을 수 있다. 현재 조회 → 저장값 조회 → 새 로그인 → 재부팅 지속성은 서로 다른 확인이다. [기존 호스트명 설명과 공식 근거](../targets.md#저장된-호스트명과-현재-호스트명)를 따른다.

계정 로그아웃도 프로세스 전체 종료와 같지 않다. `who`는 로그인 목록, `ps -u 이름`은 그 UID의 프로세스, `loginctl`은 세션과 사용자 관리자 상태를 다룬다. lingering·사용자 서비스·비동기 종료 때문에 수명이 달라질 수 있다. 계정을 제거하려면 담당 작업·파일 소유권·열린 세션을 먼저 확인한다. `loginctl terminate-user`는 그 사용자의 작업을 종료할 수 있고 `userdel -r`는 홈·메일을 삭제할 수 있으므로, 남길 데이터와 영향 대상을 결정한 뒤 수행한다. 계정 삭제가 다른 경로에 남은 UID 소유 파일까지 정리하는 것은 아니다.

## 읽는 깊이와 질문

- **직접 다룬다:** UID/GID·부모 경로 권한, PAM 진입점과 품질/기간 차이, sudo 정책, SSH/NAT/firewalld/SELinux의 역할. 실제 접속·관리 통로를 잃지 않는 데 필요하다. [PAM·authselect 읽을 범위](../targets.md#3-비밀번호pam), [sudo 읽을 범위](../targets.md#4-sudo-정책과-감사-로그)를 따라 “누가 어떤 파일을 언제 읽으며, 어느 거절이 어느 계층에서 나왔는가?”를 설명한다.
- **존재와 쓰임을 안다:** ACL·umask·setgid/sticky, SSH 공개키 인증, DNS·라우팅·NetworkManager 연결 프로필, loginctl lingering. 공유 디렉터리·키 인증·네트워크 변경·사용자 서비스가 필요할 때 해당 man과 [Rocky 프로세스 관리](https://docs.rockylinux.org/books/admin_guide/08-process/)의 프로세스·신호 설명을 읽는다. “로그아웃 후 남은 작업을 종료해도 되는가?”부터 판단한다.
- **지금 미룬다:** LDAP/AD·MFA·VPN·고급 라우팅·자체 SELinux 모듈. 단일 로컬 계정 서버의 현재 요구를 설명하는 데 필수는 아니며 문제를 겪는 계층과 목표가 생겼을 때 확장한다. 보안 기능의 우열을 단순 비교하거나 다른 배포판 설정을 섞지 않는다.
