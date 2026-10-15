# Born2beRoot 평가 진행과 상세 근거

평가 자리에서는 [진행 순서](#평가-자리에서-따라갈-진행-순서)만 위에서 아래로 따라간다. 설명·조회·변경·원복을 한 흐름으로 묶었으며, 필요한 상세 근거는 각 단계에서 연결한다. 명령은 직접 적용한 결과가 아닌 시연안이고 현재 사실은 [학습 기록](learning-log.md)을 따른다.

출처는 첨부 `Projects Born2beroot En.pdf` p.1~5와 저장소 [과제 v5.2](../en.subject.pdf) 인쇄 p.7~10·12~13·16~17이다. 이번에는 첨부 5쪽의 추출 텍스트와 과제 본문을 다시 대조했다. 첨부 오른쪽에서 잘린 문장은 복원하지 않는다. 아래 명령은 요구사항을 보여주는 선택이며 평가표의 지정 명령이 아니다. 필수 범위를 다루며 보너스 구축은 포함하지 않는다.

## 평가 자리에서 따라갈 진행 순서

**이 구간을 위에서 아래로 진행한다.** 먼저 어떤 문제가 있고 이 구성으로 무엇이 달라지는지 설명한다. 이어 실제 출력에서 그 설명에 해당하는 부분을 보여주고 평가자가 요청한 변경을 한다. 자세한 근거는 각 단계 끝의 링크에서만 펼쳐 읽는다. 아래는 시연 계획이며 실제 VM 검증 기록이 아니다.

**준비:** 별도 표시가 없으면 VM의 `jungblee` 셸이다. 기존 관리자 콘솔은 유지한다. `defenseuser`는 새 계정 예시, `reviewer42`는 평가자 로그인+42로 바꾼다. 암호는 프롬프트에만 입력한다. 조회 결과가 예상과 다르면 다음 변경을 자동으로 이어 하지 않는다.

**순서:** 제출물 → VM 개요 → 부팅·로그인 → 새 사용자·정책 → hostname·파티션 → sudo → 방화벽 → SSH → monitoring → 정리.

### 진행 1 — 부팅 전에 제출물과 디스크

**말하기:** “지금 평가할 디스크가 제출 당시의 디스크와 같은 상태인지 먼저 확인하겠습니다. 디스크 파일 전체에서 계산한 SHA-1을 제출한 값과 비교합니다. 부팅하면 로그 등 디스크 내용이 바뀔 수 있어 켜기 전에 비교하고, 다음 평가를 위한 기준 사본도 보존합니다. README에는 이 서버를 어떻게 구성했고 왜 그렇게 골랐는지 설명했습니다.”

**보여주기 — 호스트:** 본인 제출 저장소를 빈 폴더에 clone한 상태에서 README의 설명·설계·네 비교·AI 활용을 열어 보여준다. VirtualBox Storage에서 연결 파일을 확인하고 **Powered Off·스냅샷 없음**을 확인한다. 아래 경로는 확인한 실제 경로로 바꾼다.

```bash
cat signature.txt
sha1sum "$HOME/goinfre/VirtualBox VMs/b2br/b2br.vdi"
```

**판단·다음:** 두 해시가 같아야 한다. 평가자와 평가용 cold snapshot 또는 디스크 사본 보존 절차를 정한 뒤 부팅한다. VM 파일은 Git에 넣지 않는다. 평가 뒤 디스크 복구는 마지막 단계에서 별도로 한다. [제출·보존 상세](#1-평가-시작-전-저장소디스크readme)

### 진행 2 — VM과 선택한 OS 설명

**말하기:** “서버 설치를 연습하려고 일상적으로 쓰는 호스트 OS를 다시 설치할 필요는 없습니다. VirtualBox가 가상 CPU·메모리·디스크를 제공하고, 그 안에서 Rocky가 자기 커널로 부팅합니다. 그래서 디스크를 나누고 재부팅하는 실습을 게스트 안에서 할 수 있습니다. 다만 실제 CPU·메모리와 호스트 장애의 영향은 공유합니다.”

**OS 선택:** “과제는 Debian과 Rocky 중 선택하도록 했고, 저는 RHEL 계열 서버의 패키지·보안 관리를 익히려고 Rocky를 골랐습니다. Minimal은 GUI 없이 필요한 서비스를 중심으로 구성하는 출발점입니다. 관리할 프로그램을 줄일 수 있지만, 그것만으로 안전해지는 것은 아닙니다. Debian에서는 주로 APT와 deb 패키지를, Rocky에서는 DNF와 RPM 패키지를 다루므로 설치 명령과 참고할 관리 문서가 달라집니다.”

**DNF:** “프로그램 하나도 여러 라이브러리에 의존합니다. 파일을 따로 구해 설치하면 필요한 버전이 빠지거나 서로 맞지 않을 수 있습니다. DNF는 설정한 저장소에서 패키지와 필요한 의존 패키지를 찾아 함께 설치·갱신할 변경 목록을 계산합니다. 이 VM의 sudo·SSH 같은 프로그램도 패키지로 관리하므로 무엇이 설치됐는지 조회할 수 있습니다.”

**SELinux:** “서비스의 취약점으로 공격자가 그 프로그램을 조종하게 되면, 실행 계정이 접근할 수 있는 다른 자료까지 피해가 번질 수 있습니다. SELinux는 그 프로그램이 맡은 일에 필요한 접근인지 추가로 검사해 피해 범위를 제한합니다. 예를 들어 웹서버의 웹 자료 읽기는 허용하고 개인 자료 읽기는 막는 정책을 둘 수 있습니다. 실제 허용 여부는 적용된 정책에 달려 있고, 이 VM에서 웹서버 공격을 시험했다는 뜻은 아닙니다.”

**우리 구성과 연결:** “우리 서버의 SSH 4242는 세 조건을 구분합니다. sshd 설정은 4242에서 접속을 기다리게 하고, 방화벽은 그 포트로 들어오는 통신을 허용합니다. SELinux의 `ssh_port_t` 지정은 SSH 프로세스가 사용할 수 있는 종류의 포트로 4242를 분류합니다. 방화벽을 열어도 SELinux가 포트 사용을 막으면 sshd는 그 포트에서 기다리지 못할 수 있습니다. 그래서 접속 문제를 해결하려고 SELinux 전체를 끄는 대신 어느 조건이 맞지 않는지 봅니다.”

**보여주기:** README의 선택 이유를 짚고, 다음 단계의 `getenforce`와 뒤의 SSH 시연으로 연결한다. `Enforcing`은 차단 모드라는 근거이며 모든 서비스가 안전하다는 증명은 아니다. 파일 접근은 일반 권한 검사(DAC)와 SELinux 정책 검사(MAC)를 모두 통과해야 한다. root 권한 프로세스도 정책으로 제한할 수 있지만, unconfined처럼 제한이 적은 실행 영역도 있어 모든 root 작업이 똑같이 제한된다고 말하지 않는다. 프로그램과 대상의 역할을 정책이 구별하려고 붙이는 정보가 레이블·보안 문맥이며, 용어는 이 필요성을 설명한 뒤 소개한다. [접근 제어 상세와 공식 근거](concepts/12-security.md#네트워크-통과와-파일-접근을-별도로-판단한다) · [질문별 설명](#2-프로젝트-개요-짧게-말로-답하기)

### 진행 3 — 부팅과 일반 사용자 로그인

**말하기:** “로그인 암호만 있다면 디스크 파일을 가져간 사람이 다른 OS에서 내용을 읽는 문제까지 막지는 못합니다. 여기서는 LUKS로 저장 내용을 암호화해 부팅 때 먼저 열어야 합니다. 그 뒤 로그인은 열린 시스템을 누가 사용할지 확인합니다. 평소에는 일반 계정으로 작업해 실수로 시스템 파일을 바꿀 범위를 줄이고, 관리가 필요한 명령에만 sudo를 붙입니다.”

**보여주기:** 콘솔에서 암호화 해제 후 `jungblee`로 로그인한다. GUI 없는 부팅 화면과 아래 결과를 보여준다.

```bash
whoami
cat /etc/os-release
systemctl is-active sshd firewalld crond
getenforce
```

**판단:** `jungblee`, Rocky, 서비스 `active`, `Enforcing`을 확인한다. 이 단계에서는 설정을 바꾸지 않는다. [부팅 확인 상세](#3-기본-부팅과-로그인)

### 진행 4 — 계정 정책을 설명하고 새 사용자 만들기

**말하기:** “추측하기 쉬운 새 암호를 거절하는 품질 검사와, 그 암호를 얼마나 오래 쓸 수 있는지 정하는 기간 설정은 별개입니다. 이 과제에서는 최소 길이·문자 구성 등을 검사하고 30일 만료, 변경 간격 2일, 경고 7일을 적용했습니다. 새 계정을 만들어 같은 기준이 적용되는지 보여주겠습니다. 제 계정의 user42·sudo 가입은 과제 조건이고, 기존 실행 기록의 권한은 wheel 규칙에서 왔습니다. 과제용 권장 구성은 sudo 그룹에도 허용 규칙을 연결하고 그 그룹의 새 사용자로 실행을 확인하는 것입니다.”

**보여주기:** `id jungblee`, `chage -l jungblee`, `sudo chage -l root`로 그룹과 2/30/7일을 확인한다. `/etc/login.defs`의 기간 기본값, `/etc/security/pwquality.conf.d/90-local.conf`의 품질 설정을 `less`로 열어 설명한다. 길이 10·대소문자·숫자·연속 동일 문자 최대 3·사용자명 제외·이전 암호와 차이 7을 짚고, root는 이전 암호 비교만 예외라고 설명한다. 품질 규칙은 단순한 암호 선택을 줄이지만 안전성을 보장하지 않는다. 정기 만료는 같은 암호의 장기 사용을 제한하려는 과제 정책이며, 잦은 변경이 예측 가능한 변형을 유도할 수도 있으므로 실무의 보편적 최선이라고 설명하지 않는다. 새 계정의 실제 기간 출력과 암호 설정 결과를 근거로 삼는다.

**요청 시 변경:** 먼저 `getent passwd defenseuser`로 이름 충돌, `getent group evaluating`으로 그룹 존재를 확인한다. 없는 새 계정에만 진행하고, 그룹이 이미 있으면 groupadd는 생략한다.

```bash
sudo useradd -m defenseuser
sudo passwd defenseuser
sudo groupadd evaluating
sudo usermod -aG evaluating defenseuser
id defenseuser
sudo chage -l defenseuser
```

**판단·유지:** evaluating 가입과 새 계정의 2/30/7을 확인한다. `-m`은 홈 생성, `-aG`는 기존 보조 그룹을 유지하며 추가한다. **이 계정을 뒤의 sudo·SSH·monitoring까지 유지한다.** [정책·PAM 상세](#4-사용자그룹비밀번호-정책)

### 진행 5 — hostname 변경·재부팅·복원, 파티션 설명

**말하기:** “호스트명은 여러 서버의 접속 화면과 로그에서 어느 시스템인지 구분하게 해줍니다. 과제에서는 로그인명+42를 쓰며, 현재 화면만 바뀐 것이 아니라 재부팅 뒤에도 설정이 남는지 보여주겠습니다.”

**저장소 설명:** “로그가 계속 늘어 root 공간을 다 쓰면 다른 시스템 작업도 영향을 받습니다. 이 구성은 `/var/log`를 별도 볼륨으로 나눠 로그가 차지할 공간을 분리했습니다. 로그 볼륨이 가득 차면 기록 자체는 실패할 수 있어 용량 관리도 필요합니다. LVM에는 아직 나누지 않은 공간을 남겨, 나중에 필요한 볼륨에 배정할 수 있게 했습니다. 볼륨을 늘린 뒤 파일시스템도 그 공간을 쓰도록 확장해야 합니다.”

```bash
hostname
sudo hostnamectl set-hostname reviewer42
sudo reboot
```

**다시 로그인한 뒤:** `hostname`으로 바뀐 이름을 보여주고 다음처럼 복원한다. 재부팅으로 끊긴 연결은 다시 연다.

```bash
sudo hostnamectl set-hostname jungblee42
hostname
lsblk -f
sudo vgs
sudo lvs
```

**판단:** LUKS → PV → VG → LV → 파일시스템·마운트를 출력에서 짚는다. `/`·`/home`·`/var/log`·swap과 VG 여유를 설명한다. 여기서 PV는 LVM에 제공한 저장 공간, VG는 그 공간을 모은 풀, LV는 용도별로 나눠 파일시스템을 얹는 볼륨이다. `vgs`의 남은 공간과 `lvs`의 배정 크기를 연결해 설명한다. LUKS는 저장 내용의 암호화, LVM은 공간 배정을 맡으며 어느 쪽도 백업을 대신하지 않는다. `-f`는 암호화·파일시스템을 같이 보기 위해 쓴다. 저장장치는 변경하지 않는다. [파티션·재부팅 상세](#5-hostname-변경재부팅과-파티션)

### 진행 6 — sudo 그룹 추가와 정책·로그 시연

**평가 전 준비:** 기존 VM 기록은 wheel을 통한 실행이다. 아래 시연은 [sudo 그룹 허용 규칙](targets.md#sudo-그룹의-권한-연결과-평가-해석)을 적용·검증한 뒤의 권장 흐름이며 아직 VM에서 확인하지 않았다. 준비할 때 기존 Defaults를 보존하고 `sudo visudo -f /etc/sudoers.d/90-b2br`로 `%sudo ALL=(ALL:ALL) ALL`을 추가한 뒤 `sudo visudo -c`로 검사한다. 실제 변경 후 최종 디스크 백업·SHA-1·제출 signature를 갱신한다.

**말하기:** “모두가 root 암호를 공유하고 root 셸에서 일하면 누가 어떤 관리 작업을 했는지 구분하기 어렵고 실수의 영향도 큽니다. sudo는 자기 계정으로 로그인한 상태에서 허용된 관리 명령을 실행하고 그 요청을 기록하게 해줍니다. sudo 그룹에 실행 허용 규칙을 연결하고, 그 그룹에 넣은 새 사용자로 실제 실행과 기록을 보여주겠습니다. 이는 넓은 관리자 권한이므로 sudo를 썼다는 이유만으로 명령이 안전해지는 것은 아닙니다.”

```bash
rpm -q sudo
sudo usermod -aG sudo defenseuser
id defenseuser
sudo -l
sudo cat /etc/sudoers.d/90-b2br
sudo ls /var/log/sudo
sudo sudoreplay -d /var/log/sudo -l
```

**보여주기:** 설정에서 인증 최대 3회는 한 번의 sudo 인증 시도 제한이며 영구 계정 잠금은 아니라는 점을 설명한다. 사용자 정의 실패 메시지는 실패를 알리고, TTY 요구는 터미널 없이 실행하는 경로를 제한한다. secure_path는 관리 명령을 찾을 디렉터리를 정하며 명령 허용 목록이 아니다. 입출력 기록은 실행 뒤 무엇을 입력하고 받았는지 추적할 근거가 된다. `sudo cat`에 비밀 없는 시험 문장을 입력하고 Ctrl+D로 끝낸다. 다시 목록을 조회해 새 ID를 찾고 `sudo sudoreplay -d /var/log/sudo ID`로 재생한다. 입력 확인은 [로그 상세](#6-sudo-정책과-로그)를 따른다. `-d`는 현재 로그 디렉터리, `-l`은 세션 목록이다.

**새 사용자의 실제 권한 시연:** 기존 관리자 셸을 유지하고 `su - defenseuser`로 새 로그인한다. 이 계정에는 관리자 권한을 주는 그룹으로 sudo만 추가하며 wheel로 우회하지 않는다. 기본 그룹·evaluating 소속은 유지한다.

```bash
# 새 사용자 로그인 셸
id
sudo -l
sudo whoami
exit
```

`id`에서 sudo 가입·wheel 미가입, 허용 목록에서 다른 직접 허용 경로가 없는지 확인한다. 새 사용자 암호로 인증한 `sudo whoami`의 root 출력과 관리자 셸에서 조회한 새 사용자 로그를 대조한다. 계정은 SSH·monitoring까지 유지하고, 로그는 삭제하지 않는다.

### 진행 7 — 방화벽의 8080 추가·삭제

**말하기:** “서버에서 어떤 프로그램이 접속을 기다리더라도 외부에서 전부 접근하도록 둘 필요는 없습니다. firewalld로 필요한 통신만 허용해 노출 범위를 줄입니다. 여기서는 SSH에 필요한 4242를 허용했습니다. 지금 8080 규칙을 추가하면 통신 허용 정책은 바뀌지만, 8080에서 응답할 서버가 저절로 생기지는 않습니다. 목록에서 변경을 확인한 뒤 원래대로 지우겠습니다.”

```bash
rpm -q firewalld
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --list-all
```

**요청 시 변경·원복:** 실제 NIC의 zone이 public인지 먼저 확인한다. 다르면 아래 public도 그 이름으로 바꾼다. 원래 8080이 없을 때만 추가·삭제한다.

```bash
sudo firewall-cmd --zone=public --add-port=8080/tcp
sudo firewall-cmd --zone=public --list-all
sudo firewall-cmd --zone=public --remove-port=8080/tcp
sudo firewall-cmd --zone=public --list-all
```

**판단:** 추가된 8080과 삭제 후 원래 4242 정책을 보여준다. `--zone`은 실제 인터페이스 정책을 고르는 데 필요하다. 이번 변경은 runtime이며 영구 설정·서버·NAT를 추가하지 않는다. [방화벽 상세](#7-firewalld-4242-확인--8080-추가--삭제)

### 진행 8 — 같은 새 사용자로 SSH, root 거절

**말하기:** “서버 앞에 직접 앉지 않고 관리하려면 원격 접속이 필요합니다. SSH는 접속 대상 서버를 확인하고 통신을 암호화해, 네트워크에서 암호나 명령을 그대로 엿보는 일을 막습니다. 로그인할 계정도 인증합니다. 우리는 일반 계정으로 들어온 뒤 sudo를 쓰도록 root 직접 로그인을 막았습니다. 4242는 과제에서 정한 포트이며 포트 번호를 바꾸는 것 자체가 인증이나 암호화를 대신하지는 않습니다.”

**보여주기 — 기존 관리자 셸:** `rpm -q openssh-server`, `systemctl is-active sshd`, `sudo ss -lntp`로 설치·동작·4242 리스닝을 확인한다. `ss`의 옵션은 수신 중인 TCP 포트를 숫자로, 담당 프로세스와 함께 보기 위한 것이다. `sudo sshd -T | less`에서 `/permitrootlogin`으로 적용값 `no`를 찾고 `q`로 나온다.

**호스트의 새 터미널:**

```bash
ssh -p 4242 defenseuser@127.42.42.42
```

접속한 셸에서 `whoami`, `id`로 같은 계정과 evaluating·sudo 가입을 확인한다. **이 세션은 다음 monitoring 수신용으로 유지한다.** 호스트의 다른 터미널에서 `ssh -p 4242 root@127.42.42.42`를 시도해 인증 거절을 보여준다. timeout은 root 정책 검증이 아니다. 키 확인이 나오면 콘솔의 공개키 지문과 대조한다. SSH 설정은 바꾸지 않는다. [SSH 상세](#8-ssh-새-사용자-접속과-root-거절)

### 진행 9 — monitoring 설명 → 매분 → 중단·재부팅 → 복구

**말하기:** “관리자가 매번 여러 조회 명령을 입력하지 않아도 자원 사용량과 접속 상태를 한 보고서로 볼 수 있게 했습니다. 스크립트는 실행할 때마다 값을 모아 변수에 담고, 마지막 printf 그룹에서 보고서를 만들어 wall로 로그인한 터미널에 보냅니다. memory·disk에는 MiB의 사용량/전체와 비율을 담습니다. `export LC_ALL=C`는 출력 언어가 달라져 `Mem:`·`total` 행을 못 찾는 문제를 피하려는 설정입니다.”

**예약과 연결:** “스크립트는 보고서를 만들고, cron은 언제 실행할지를 맡습니다. root crontab의 `*/10`은 매시 0·10·20…분이며 등록한 때부터 10분을 세는 방식은 아닙니다. `@reboot`는 서버 시작 시 실행을 위한 별도 예약입니다. 평가에서 실행 간격을 바꾸거나 중단할 때는 보고서 코드를 고치지 않고 예약만 바꾸므로, 다시 켤 때 같은 코드를 그대로 사용할 수 있습니다.”

**보여주기 — 관리자 셸:** 코드의 수집부와 출력부를 짚고 수동 방송을 콘솔·새 사용자 SSH에서 확인한다. 먼저 아래 조회 결과를 보관해 중단 후와 비교한다.

```bash
sudo cat /usr/local/sbin/monitoring.sh
sudo crontab -l
ls -l /usr/local/sbin/monitoring.sh
sha256sum /usr/local/sbin/monitoring.sh
sudo /bin/bash /usr/local/sbin/monitoring.sh
```

**요청 시 변경:** 편집은 `sudo crontab -e`, 저장 확인은 `sudo crontab -l`로 한다. PATH와 다른 예약은 유지한다.

| 순서 | 바꿀 것 → 보여줄 결과 |
|---|---|
| 매분 | 기존 `*/10` 주기 줄만 `* * * * * /bin/bash /usr/local/sbin/monitoring.sh`로 교체 → 두 분 경계에서 콘솔·SSH의 12항목 방송 확인 |
| 값 변화 | 새 사용자 SSH에서 `exit` 후 다음 방송을 확인 → 사용자·TCP 수를 `who`, `ss -tn`과 대조. 같은 계정으로 다시 접속해 두 터미널 수신 확인 |
| 중단 | monitoring 주기 줄과 `@reboot` 줄 **둘 다** 주석 처리 → 다음 분 경계와 예약 상태 확인. 스크립트·권한·crond 전체는 바꾸지 않음 |
| 재부팅 | `sudo reboot` 후 로그인 → 같은 경로의 `ls -l`·`sha256sum`이 유지되고 monitoring 예약이 주석 상태인지 확인 |
| 호출 확인 | `sudo journalctl -b` → `/monitoring.sh`로 검색하고 `n`으로 다음 항목, `q`로 종료. `-b`는 이번 부팅만 보기 위한 옵션. 주석 상태에서 해당 cron 호출이 없어야 함 |
| 복구 | 아래 원래 예약으로 복원 → 매분·중복 줄이 없는지 조회하고 다음 10분 경계의 콘솔·SSH 방송 확인. 재부팅으로 끊긴 새 사용자 SSH는 다시 접속 |

```cron
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

**정확히 말하기:** “부팅 때 수신자가 없으면 방송이 안 보일 수 있고, cron의 CMD·END 기록은 호출·종료 근거이지 12개 지표의 성공이나 터미널 수신 증거는 아닙니다.” `@reboot`는 과제 p.10의 서버 시작 시 요구를 위한 구현 선택이다. 수집 실패 처리는 추가하지 않은 코드다. [지표·cron 상세](#9-monitoring-코드--매분--중단-재부팅--복구) · [계산과 구현 설명](targets.md#6-monitoringsh와-cronwall)

### 진행 10 — 시연을 마친 뒤 정리

**말하기:** “시연용 계정과 열린 포트, 매분 예약이 남으면 다음 사용이나 평가 때 원래 구성과 다르게 동작합니다. 변경한 설정을 되돌리고 일회성 계정을 정리하겠습니다. 그래도 로그인·명령 실행으로 로그 등 디스크 내용은 달라졌으므로 제출 SHA-1까지 돌아오는 것은 아닙니다. 디스크는 따로 보존한 기준 상태로 복구하고 다시 비교해야 합니다.”

**확인:** `hostname`은 jungblee42, 방화벽은 원래 정책, cron은 10분+부팅 예약인지 본다. 새 사용자 세션을 모두 로그아웃하고 남은 프로세스·보존할 파일을 확인한다. **이번에 만든 일회성 계정만** `sudo userdel -r defenseuser`로 삭제한다. `-r`은 홈 데이터도 삭제한다. evaluating도 이번에 만들었고 다른 사용처가 없을 때만 `sudo groupdel evaluating`로 지운다.

**디스크:** VM 정상 종료 후 사전에 정한 [디스크 보존·복원 절차](targets.md#평가-사이의-디스크-보존과-복원)를 따르고 SHA-1을 다시 대조한다. 스냅샷 Delete만으로 원래 바이트가 돌아온다고 가정하거나 실행 중인 디스크를 덮어쓰지 않는다. [종료 상세](#10-종료복구와-중단-조건)

---

## 필요할 때 펼쳐 읽는 상세 근거

아래는 진행본에서 참조하는 요구사항·설명·진단 자료다. 평가 중 모든 명령을 순서대로 추가 실행하는 목록이 아니다. 옵션의 뜻은 [명령어 참조](commands.md), 문제 원인과 복구는 [targets](targets.md)를 찾는다.

## 먼저 확인할 준비 공백

[전체 요구사항 감사](requirements-audit.md)에 지표 정의·비밀번호 검사 한계와 조건별 근거를 모았다. 이 표의 과거 성공을 최신 문서 코드·새 sudo 정책의 적용 완료로 확대하지 않는다.

| 항목 | 기록으로 확인된 범위 | 아직 준비·확인할 범위 |
|---|---|---|
| 계정 그룹 | 10월 13일 `id jungblee`에 `wheel,user42,sudo`가 모두 있음 | 새 로그인 셸의 `id`; 평가용 새 계정의 `evaluating`·`sudo` 가입 시연 |
| 관리자 권한 | `%wheel ALL=(ALL) ALL`과 sudo 사용 확인 | sudo 그룹 허용 규칙 추가는 제안 단계; wheel 없는 새 사용자의 새 로그인·sudo 실행·로그 검증 |
| hostname | `check42`로 변경·SSH 재접속·`jungblee42` 복원 확인 | 변경한 이름으로 **재부팅 후** 유지 확인(평가지 p.3) |
| 방화벽 | 현재 4242 중심 구성 기록 | 8080 추가·목록·삭제 시연 결과는 미확인(p.4) |
| 새 사용자 SSH | 과거 `b2brtest` 접속 성공 후 계정 삭제 확인 | 평가 당일 새로 만든 계정으로 다시 시연(p.4) |
| monitoring | 30분 중단 및 복구 후 방송 수신 확인 | 매분 실행, 중단 상태의 재부팅, 모든 터미널 수신, 부팅 방송 수신(p.4 및 과제 p.10) |
| 제출·디스크 | 당시 스냅샷 없음 화면, Powered Off에서 SHA-1 계산 확인 | 실제 Storage 연결·기준 사본·복원 검증, 학교 저장소와 `signature.txt` 제출 상태 |
| 추가 권장 확인 | Rocky 10.2 조회, 일부 GUI 패키지 부재, 일부 품질 거절 확인 | 과제의 최신 안정 버전 조건 대조, GUI 전체 부재, 나머지 품질·root 적용 시험 |

과거 확인을 현재 VM의 실시간 상태로 단정하지 않는다. 아래 준비를 읽었다고 학습 로그의 미확인을 완료로 바꾸지 않는다.

## 1. 평가 시작 전: 저장소·디스크·README

**평가 요구 — p.1~2:** 본인 참석, 본인의 올바른 제출 저장소를 빈 폴더에 clone, 속이는 alias 없음, 보조 검사 스크립트 함께 검토. 루트 `signature.txt`와 실제 디스크의 서명 일치, 시작 시 스냅샷 없음, 미리 실행 중인 VM은 받지 않음. **부팅 전에** 대조한 뒤 평가용 VM을 시작한다.

**대비 방법 — 호스트:** VirtualBox Storage에서 실제 연결된 VDI를 먼저 식별한다. 알려진 후보는 `~/goinfre/VirtualBox VMs/b2br/b2br.vdi`지만 연결 확인을 대신하지 않는다. Powered Off 상태에서 실제 경로의 `sha1sum` 결과 중 **해시만** 제출 파일과 비교한다. 경로 문자열까지 비교하지 않는다. 과거 계산값을 현재 값 대신 쓰지 않는다.

```bash
# 호스트: 실제 제출 저장소의 루트, 확인한 디스크 경로에서만
sha1sum "$HOME/goinfre/VirtualBox VMs/b2br/b2br.vdi"
cat signature.txt
```

기준 디스크를 유지하는 평가용 cold snapshot 또는 다른 디렉터리의 디스크 복사본 사용이 평가지 p.2에 나온다. 과제 p.16~17도 시작 시 스냅샷 없음과 평가용 스냅샷 생성·종료 시 삭제를 구분한다. **‘모든 스냅샷은 언제나 금지’로 외우지 않는다.** 삭제 버튼만 누르면 원본 바이트가 복원된다는 뜻도 아니다. 실제 보존·대조·복원은 [디스크 보존 절차](targets.md#평가-사이의-디스크-보존과-복원)를 따른다. 복사한 VDI의 UUID를 임의 변경하면 바이트·해시도 달라질 수 있다.

**README 요구 — p.2, 과제 p.12~13:** 제출 저장소 루트에 README가 있어야 한다. 첫 줄은 아래 형식이며 이탤릭이다.

```markdown
*This project has been created as part of the 42 curriculum by jungblee.*
```

프로젝트·OS 선택과 장단점·주요 설계(파티션, 보안, 사용자, 서비스), 네 가지 비교(Debian/Rocky, AppArmor/SELinux, UFW/firewalld, VirtualBox/UTM)를 확인한다. 과제의 Description·Instructions·Resources와 AI 활용 설명도 함께 대조한다. [제출용 README](../submission/README.md)를 실제 제출 루트에 복사하는 구조이며 학습 저장소 루트 안내문으로 대신하지 않는다.

## 2. 프로젝트 개요: 짧게 말로 답하기

**평가 요구 — p.2:** VM 동작·용도, OS 선택, Rocky/Debian 차이, Rocky 선택 시 SELinux·DNF를 설명한다. 설명이 명확하지 않으면 중단한다고 명시한다. 평상시 monitoring은 10분마다 동작해야 한다.

| 예상 질문 | 말로 답할 핵심 |
|---|---|
| VM은 무엇이고 왜 쓰나? | 호스트 자원을 가상 CPU·메모리·디스크로 제공해 별도 게스트 커널과 OS를 실행한다. 격리된 실습·다른 OS 실행·환경 재현에 유용하지만 자원 비용과 호스트 장애 영향이 있다. VirtualBox가 그 실행을 중재한다. |
| Docker와 같은가? (권장 확장 질문) | VM은 독립 커널을 부팅하고, 일반적인 Linux 컨테이너는 실행 환경을 분리하되 커널을 공유한다. Docker는 이미지로 컨테이너를 배포·실행하는 도구다. [가상화 장](concepts/02-virtualization.md) |
| 왜 Rocky 10.2 Minimal인가? | RHEL 계열의 DNF·SELinux·firewalld를 함께 다루는 최소 서버 구성이다. GUI 없이 필요한 서비스만 두며 보안 정책과 서비스의 관계를 배운다. 패키지·정책을 함께 이해해야 하는 관리 비용이 있다. 버전 조회와 ‘최신 안정 버전’ 충족 검증은 별개다. |
| Debian과 Rocky 차이는? | Debian은 커뮤니티 범용 배포판과 deb/APT/dpkg 생태계, Rocky는 RHEL 호환 지향과 RPM/DNF 생태계다. 이 과제는 Debian에 UFW, Rocky에 firewalld를 요구한다. Debian 계열 AppArmor와 Rocky SELinux의 정책 모델도 비교한다. 어느 쪽이 무조건 안전하거나 빠르다는 비교는 하지 않는다. |
| DNF는 무엇인가? | 프로그램에 필요한 라이브러리와 버전을 따로 맞추는 부담을 줄인다. 설정된 저장소의 의존 관계를 읽고 함께 설치·갱신할 패키지를 계산한다. 변경 목록을 확인해야 의도하지 않은 추가·삭제를 알 수 있다. RPM은 개별 패키지 형식과 설치 정보 관리를 맡는다. [패키지 장](concepts/13-packages.md) |
| SELinux는 무엇인가? | 서비스가 공격당해도 맡은 일에 불필요한 자료·포트까지 사용하지 못하도록 추가 검사해 피해 범위를 제한한다. 실제 범위는 정책과 실행 영역에 달린다. [진행 2](#진행-2--vm과-선택한-os-설명)의 SSH 4242 예로 설명한다. Enforcing은 거절을 적용하고 Permissive는 기록하되 차단하지 않는다. |
| 다른 필수 비교는? | AppArmor는 경로 중심 프로파일, SELinux는 레이블·타입 중심 정책. UFW는 간단한 규칙 인터페이스, firewalld는 zone·서비스 및 runtime/permanent 관리. VirtualBox는 이 호스트의 가상화 도구, UTM은 Mac의 가상화·에뮬레이션 도구이며 과제의 사용 허용 조건이 있다. [제출용 비교표](../submission/README.md#운영체제-선택과-비교) |

## 3. 기본 부팅과 로그인

**평가 요구 — p.2~3:** GUI 없는 시작, 암호 입력과 일반 사용자 로그인, 허용 OS·방화벽·SSH 동작 확인. p.3의 Simple setup에는 UFW라고 쓰였지만 p.4와 **과제 p.8은 Rocky에 firewalld**를 명시하므로 UFW를 추가 설치하는 의미가 아니다.

**대비 방법 — VM 콘솔/일반 사용자:** LUKS 해제 암호와 계정 로그인 암호의 역할을 구분하고 root로 일상 로그인하지 않는다.

```bash
whoami
id
cat /etc/os-release
systemctl get-default
systemctl is-active sshd firewalld crond
getenforce
```

기대값은 `jungblee`, Rocky 10.2, 서버용 `multi-user.target`, 각 서비스 `active`, `Enforcing`이다. 화면·설치 패키지·실행 서비스를 함께 확인한다. target 하나나 패키지 몇 개의 부재만으로 GUI 전체 부재를 증명하지 않는다. LUKS는 꺼진 디스크의 내용을 보호하고 로그인 인증은 열린 시스템의 사용자를 확인한다.

## 4. 사용자·그룹·비밀번호 정책

**평가 요구 — p.3:** 본인 로그인 계정의 `sudo`·`user42` 가입, 새 계정 생성·정책에 맞는 암호 지정, `evaluating` 그룹 생성과 배정, 정책 설정과 장단점 설명.

```bash
id jungblee
getent group sudo user42 evaluating
sudo authselect current
sudo authselect check
cat /etc/pam.d/passwd
cat /etc/pam.d/system-auth
cat /etc/security/pwquality.conf.d/90-local.conf
chage -l jungblee
sudo chage -l root
```

**수치 근거 — 과제 p.8~9:** 다음 값은 잘린 평가 문장을 복원한 것이 아니라 과제 본문과 기존 설정을 대조한 것이다.

**설명:** PAM의 password 스택에서 pam_pwquality가 새 암호를 검사하고 pam_unix가 저장한다. 본체·drop-in·PAM 인수의 실제 우선순위를 함께 읽는다. 기간은 최소 2일·최대 30일·7일 경고, 품질은 길이 10 이상·대문자/소문자/숫자·동일 문자 연속 최대 3·사용자명 배제·이전 암호와 차이 7이다. root도 품질을 적용하되 이전 암호 비교는 과제의 예외다. `enforce_for_root`와 `difok`의 경로 차이는 [PAM 설명](targets.md#3-비밀번호pam)을 따른다.

**장단점:** 짧고 추측하기 쉬운 암호와 재사용을 줄이고 유출된 암호의 장기 사용을 제한하려는 정책이다. 반면 잦은 변경은 예측 가능한 변형·사용 부담을 낳을 수 있고, 최소 2일은 즉시 재변경을 막는다. 만료 정책만으로 도난 계정·피싱을 해결하지 못한다. ‘과제라서’만으로 설명하지 않는다.

**변경 시연안:** `defenseuser`는 예시다. 평가자와 정한 새 이름으로 바꾸고 먼저 `getent passwd defenseuser`로 기존 계정이 아닌지 확인한다. 기존 `evaluating`이 있으면 다시 만들거나 삭제하지 않는다.

```bash
sudo useradd -m defenseuser
sudo passwd defenseuser
sudo chage -l defenseuser
# evaluating 그룹이 없을 때만
sudo groupadd evaluating
sudo usermod -aG evaluating defenseuser
id defenseuser
```

새 계정의 기간이 2/30/7인지 기본값 적용 결과를 확인한다. 다르면 생성 기본값부터 조사한다. 본인 계정의 `sudo,user42` 요구는 과제 p.8에도 명시되어 **wheel로 대체했다고 설명하면 안 된다.** 기존 가입 기록은 확인되어 있지만 새 계정과 새 세션은 별개다. `-aG`의 `-a`는 기존 보조 그룹을 유지한다. root가 새 계정 암호를 설정하는 시험만으로 이전 암호 유사성 거절을 검증할 수는 없다.

## 5. hostname 변경·재부팅과 파티션

**평가 요구 — p.3:** `login42` 형식, 평가자의 로그인에 맞는 이름으로 변경 후 **재부팅**, 유지 확인 후 원래 이름으로 복원. 파티션을 보여주고 LVM을 설명한다.

```bash
hostname
cat /etc/hostname
# reviewer42를 평가자의 실제 로그인+42로 바꾼다
sudo hostnamectl set-hostname reviewer42
sudo reboot
```

재부팅은 SSH를 끊는다. 콘솔에서 디스크 암호 입력 후 다시 접속해 `hostname`·`cat /etc/hostname`이 새 이름인지 확인한다. 프롬프트만 보지 않는다. 복원은 `sudo hostnamectl set-hostname jungblee42` 후 두 조회로 확인한다. 복원 후 재부팅 유지 확인도 권장하지만 평가표의 변경 후 재부팅과 구분한다.

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo pvs
sudo vgs
sudo lvs
findmnt /
findmnt /home
findmnt /var/log
```

각 마운트 지점과 해당 LV·ext4의 연결을 대조한다.

**설명:** BIOS Boot 영역·비암호화 `/boot`, 암호화 LUKS 안에 PV→VG→LV, 각 LV의 ext4 또는 swap이 이어진다. `/`, `/home`, `/var/log` 분리는 OS·사용자 데이터·로그의 용량 영향을 나누고 VG 여유는 나중 확장에 둔다. LV 확대와 파일시스템 확대는 별도이며 LVM은 암호화나 백업 자체가 아니다. [저장장치 장](concepts/07-storage.md). 필수 구조를 설명하며 보너스 그림을 그대로 만들었다고 주장하지 않는다.

## 6. sudo 정책과 로그

**평가 요구 — p.3~4:** sudo 설치, 새 사용자를 `sudo` 그룹에 배정, sudo의 용도·예시와 엄격한 규칙 설명, `/var/log/sudo/`의 존재·내용·새 명령 후 기록 확인.

```bash
rpm -q sudo
sudo usermod -aG sudo defenseuser
id defenseuser
sudo visudo -c
sudo -l
sudo cat /etc/sudoers.d/90-b2br
sudo ls -la /var/log/sudo
sudo sudoreplay -d /var/log/sudo -l
```

**정책 수치 근거 — 과제 p.9.**

**설명:** sudo는 정책에 따라 다른 사용자 권한으로 명령을 실행한다. 3회 인증 제한·사용자 정의 실패 문구·TTY 요구·secure_path·입출력 로그를 각각 보여준다. secure_path는 명령 검색 경로이며 명령 허용 목록이 아니다. 기존 관리 권한 기록은 `%wheel` 규칙에 따른 것이다. 원문 p.9는 sudo 그룹의 강한 설정을 요구하고, 평가표는 새 사용자의 sudo 가입 뒤 실행·로그를 확인한다. 따라서 권장안은 `%sudo ALL=(ALL:ALL) ALL`로 sudo 그룹에 실제 허용을 연결하는 것이다. 정확한 규칙 문법은 구현 선택이며 원문 지정 줄은 아니다. 기존 Defaults·wheel·관리자 세션을 유지하고, wheel 없는 새 사용자로 새 로그인해 `sudo whoami`와 실행 로그를 확인한다. 이 정책 추가와 검증은 아직 VM에서 수행하지 않았다. [정책 변경·제출 기준 갱신](targets.md#sudo-그룹의-권한-연결과-평가-해석).

비밀 없는 `sudo cat`에 시험 문장을 입력하고 Ctrl+D로 끝낸 뒤 `sudoreplay -l`에서 해당 명령의 TSID를 찾아 재생한다. `sudo sudoreplay -d /var/log/sudo ID`는 출력, `-f stdin,ttyin ID`는 입력을 확인한다. 조회 자체도 sudo 기록을 늘릴 수 있다. 문법 검사·명시적 TTY 거절·입출력 확인의 상세 절차는 [치트시트](cheatsheet.md#9-로그-생성과-출력-재생)를 따른다.

## 7. firewalld: 4242 확인 → 8080 추가 → 삭제

**평가 요구 — p.4:** 설치·동작·역할 설명, 활성 규칙에 4242, 8080 허용 규칙 추가 후 목록 확인, 새 규칙 삭제.

```bash
rpm -q firewalld
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
# ZONE은 실제 게스트 NIC가 속한 zone 이름으로 바꾼다
sudo firewall-cmd --zone=ZONE --list-all
sudo firewall-cmd --permanent --zone=ZONE --list-all
```

**대비 방법:** 일회성 시연은 runtime 규칙으로 충분하다. 평가표가 영구 8080 허용을 요구한다고 쓰지 않는다. 먼저 8080이 원래 없음을 확인하고 다음을 개별 실행한다.

```bash
sudo firewall-cmd --zone=ZONE --add-port=8080/tcp
sudo firewall-cmd --zone=ZONE --list-ports
sudo firewall-cmd --zone=ZONE --remove-port=8080/tcp
sudo firewall-cmd --zone=ZONE --list-all
```

추가 시 `4242/tcp 8080/tcp`, 삭제 후 원래 규칙을 기대한다. 기존 규칙이 있으면 임의로 삭제하지 않는다. permanent를 바꾼 시연이라면 그쪽도 원복하고 적용 결과를 대조한다. 포트를 허용해도 그 포트의 서버가 생기지 않는다. 평가용 8080 서버 설치·NAT 포워딩 추가는 이 항목의 요구가 아니다. [runtime/permanent 근거](https://firewalld.org/documentation/man-pages/firewall-cmd.html).

## 8. SSH: 새 사용자 접속과 root 거절

**평가 요구 — p.4:** 설치·동작·역할, 게스트 SSH가 4242만 사용, 새 사용자 원격 로그인, root SSH 불가.

```bash
rpm -q openssh-server
systemctl is-active sshd
sudo sshd -t
sudo sshd -T | grep -E '^(port|permitrootlogin) '
sudo ss -lntp
```

4242 리스닝과 `permitrootlogin no`를 기대한다. Match 조건이 있으면 실제 사용자·주소의 적용값도 조사한다. 설정만 보거나 모든 접속 실패를 root 거절 증거로 취급하지 않는다.

```bash
# 호스트의 새 터미널
ssh -p 4242 defenseuser@127.42.42.42
# 접속한 새 사용자 셸
whoami
id
# 이 세션은 monitoring 수신용으로 유지한다
# 호스트의 다른 터미널: root는 로그인되어서는 안 된다
ssh -p 4242 root@127.42.42.42
```

**설명:** SSH는 암호화된 원격 채널과 서버·사용자 인증을 제공한다. 호스트 127.42.42.42:4242 NAT 전달 → 게스트 방화벽 → sshd → 계정 인증 순서다. 게스트 자기 루프백과 호스트 주소는 다르다. 키 확인이 나오면 콘솔의 호스트 키 지문과 대조한다. 공개키 또는 비밀번호 인증은 평가지가 허용하지만 현재 실습은 비밀번호 인증이다. [SSH 개념](concepts/12-security.md#ssh는-서버-확인과-사용자-로그인을-연결한다).

## 9. monitoring: 코드 → 매분 → 중단 재부팅 → 복구

**평가 요구 — p.4:** 코드 동작·cron·10분 실행 설정 설명, 정상 동작을 확인한 후 매분 실행과 동적 값 확인, 스크립트 자체를 바꾸지 않고 시작 후 실행 중단, 재부팅 후 같은 위치·같은 권한·내용 유지 확인.

```bash
sudo cat /usr/local/sbin/monitoring.sh
sudo crontab -l
systemctl is-active crond
stat -c '%U:%G %a %n' /usr/local/sbin/monitoring.sh
sha256sum /usr/local/sbin/monitoring.sh
```

**지표 목록 근거 — 과제 p.10, 아래 계산 방식은 현재 치트시트 구현.**

**설명:** root cron이 Bash 스크립트를 실행한다. 시작의 `export LC_ALL=C` 아래에서 모든 값을 변수에 모으고, 마지막 `{ printf...; } | wall`에서 항목 이름과 값을 한 번 방송한다. memory·disk는 awk가 만든 표시 문자열이다. root cron 안에 sudo를 또 넣지 않는다. cron은 달력 기준이며 `*/10`은 매시 0·10·20…분, 부팅 즉시 실행은 별도의 `@reboot`다. 다음 자료가 어떤 출력이 되는지 코드를 짚어 설명한다.

| 출력 | 이 구현의 원자료·해석 |
|---|---|
| Architecture / CPU physical / vCPU | `uname -a`; `lscpu -p`의 고유 온라인 소켓 수·온라인 CPU 행 수. 호스트 물리 CPU를 직접 센 값은 아님 |
| Memory / Disk | `free -m`, `df -m --total`의 used/total. 사용량·전체 모두 MiB, df Use%와 산식 차이 및 tmpfs/devtmpfs 제외 설명 |
| CPU load / Last boot | `vmstat 1 2` 마지막 표본의 `100-id`(직전 1초, 순수 계산 시간만은 아님), `uptime -s` |
| LVM / TCP | `lsblk` TYPE에 lvm 존재; `ss`의 ESTABLISHED TCP 소켓 행 수 |
| User log / Network | `who`의 고유 사용자명 수; enp0s3의 IPv4·MAC. 세션 수와 사용자 수 구별 |
| Sudo | `/var/log/sudo`에 보관된 `sudoreplay -l` 세션 수. 보존 범위가 있으며 시스템 수명 전체 명령 횟수와 같지 않을 수 있음 |

**변경과 검증 순서:** 먼저 crontab 원문·스크립트 해시·소유자/권한/경로를 보관한다. `sudo crontab -e`에서 **기존 주기 줄만** `* * * * * /bin/bash /usr/local/sbin/monitoring.sh`로 바꾼다. PATH와 다른 예약은 보존하고 10분 줄을 동시에 남기지 않는다. 1분 경계를 두 번 관찰하고 콘솔·SSH 각각의 수신을 확인한다. 다른 사용자 SSH 세션의 접속/종료나 비밀 없는 sudo 명령으로 TCP·사용자·sudo 수의 변화를 원자료와 대조한다. CPU·메모리가 반드시 증가해야 한다고 가정하지 않는다.

중단은 같은 편집기에서 monitoring의 주기 줄과 `@reboot` 줄을 **모두 주석 처리**한다. crond 전체 중지, 스크립트 이동·수정·chmod로 우회하지 않는다. `sudo crontab -l` 확인 후 `sudo reboot`한다. 재로그인 후 같은 `stat`·`sha256sum`과 경로로 불변을 확인하고, 현재 부팅의 작업 기록과 분 경계 관찰을 함께 본다.

```bash
sudo journalctl -b
```

`-b`는 이번 부팅의 로그만 선택한다. 화면에서 `/monitoring.sh`로 검색하고 `n`으로 다음 기록, `q`로 종료한다. 중단 상태에서는 해당 cron 호출이 없어야 하며 조용한 화면만으로 판단하지 않는다. 부팅 때 수신자가 없으면 방송이 안 보일 수 있고, CMD·END 기록은 호출·종료 근거이지 12개 지표의 성공이나 터미널 수신 증거는 아니다. 필요하면 [유지 관리 진단](operations/maintenance.md)을 따른다.

**복구:** 원래 PATH 및 아래 두 줄을 활성화하고 중복·매분 줄이 남지 않았는지 조회한다. 다음 10분 경계의 실제 방송까지 확인한다.

```cron
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```

cron 문법 근거는 [crontab(5)](https://man7.org/linux/man-pages/man5/crontab.5.html), 원래 전체 코드는 [치트시트](cheatsheet.md#34-전체-출력을-wall-한-번으로-전달)에 있다.

## 10. 종료·복구와 중단 조건

**권장 복구 체크:** hostname `jungblee42`, 8080 제거와 원래 4242 정책, monitoring 10분+부팅 예약·방송, 스크립트 경로·소유자·권한·내용을 대조한다. 시험 계정은 시연이 끝난 뒤 로그아웃·남은 프로세스·필요 데이터부터 확인하고 이번에 만든 계정만 삭제한다. `evaluating`도 이번에 만들었고 다른 용도가 없을 때만 삭제한다. 실제 평가 요구가 끝나기 전에 정리하지 않는다. 기준 디스크 보존·복원은 별도 절차이며 설정을 되돌렸다는 사실만으로 기존 SHA-1이 복구되지 않는다. 평가 전 실제 그룹·보안 설정을 새로 고쳐 제출 기준으로 삼는다면, 수정 완료 후 종료된 최종 디스크의 백업과 SHA-1·signature.txt를 같은 시점으로 다시 맞춰야 한다. 이 문서 작성은 그 변경이나 제출 갱신을 수행한 것이 아니다.

| 구분 | 평가지에 명시된 조건과 대응 |
|---|---|
| 진행 전제 / 일반 플래그 (p.1~2) | 당사자 참석과 올바른 제출물 필요. 빈 저장소·잘못된 제출 등은 종료 대상. 일반 안내에는 0점, 부정행위는 -42가 명시되어 있고 의심 시 Cheat 플래그와 중단을 신중하게 판단하도록 한다. |
| 시작 단계 중단 (p.2) | 실행 중인 VM은 수락하지 않음. 서명 불일치 또는 예상대로 동작하지 않으면 중단. 시작 시 스냅샷 없음과 평가용 보존 절차를 먼저 확인한다. |
| README 중단 (p.2) | 지정 첫 줄·필수 내용·네 비교 중 누락이 있으면 즉시 종료한다고 명시한다. |
| 필수 기능·설명 중단 (p.2~4) | 각 단계의 동작 실패나 설명 불명확에 중단 문구가 반복된다. 특히 hostname 재부팅 유지, 정책·그룹, sudo 로그, 방화벽 변경, SSH, monitoring 변경·중단을 설명과 함께 준비한다. |
| 보너스 (p.5) | 필수 부분이 완전해야 평가한다. 이번 목표가 아니므로 서비스·포트를 추가하지 않는다. 보너스 미구현을 필수 실격과 혼동하지 않는다. |

평가지의 Ratings에는 설명 불가·미완료 등 플래그도 있다. 잘린 문장의 세부 조건이나 모든 실패의 점수를 추정하지 않는다. 평가 당일 표시되는 완전한 평가표·과제와 차이가 있으면 평가자와 원문을 먼저 대조한다.
