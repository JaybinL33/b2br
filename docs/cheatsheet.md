# Born2beRoot 최소 치트시트

`jungblee`의 일반 셸에서 **한 줄씩 개별 실행**한다. 설명과 목표는 [targets](targets.md), 확인된 결과는 [학습 기록](learning-log.md)에 둔다. 이 목록은 자동 실행 스크립트나 완료 기록이 아니다.

## 1. 설치 후 디스크·VG 확인

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo vgs
```

## 2. 사용자·그룹 확인

```bash
id
getent group user42 sudo
```

10월 13일 사용자 화면의 `id jungblee`에서 UID 1000, 기본 GID 1000, 그룹 `jungblee`, `wheel`, `user42`, `sudo`를 확인했다. 계정의 그룹 가입은 확인했으므로 아래 추가 절차를 다시 적용할 필요는 없다. 인자 없는 `id`로 현재 로그인 세션의 그룹이 갱신되었는지 확인하는 것은 남아 있다. 그룹 가입만으로 sudo 정책까지 검증되지는 않는다.

조회 결과에 **없는 그룹만** 해당 줄을 실행한다. 이미 있으면 건너뛴다.

```bash
sudo groupadd user42
sudo groupadd sudo
```

기존 그룹을 유지하며 두 그룹에 추가하고 계정 정보를 확인한다.

```bash
sudo usermod -aG user42,sudo jungblee
id jungblee
```

현재 로그인 세션을 끝낸다.

```bash
exit
```

`jungblee`로 다시 로그인한 뒤 실행한다. `user42`·`sudo`가 보이고 기존 그룹도 유지되어야 한다.

```bash
id
```

## 3. 비밀번호 기간 정책 — 기간 값 확인, 실제 만료 미검증

아래는 지금까지 안내한 절차다. 사용자 화면에서 root와 jungblee 모두 최소 2일·최대 30일·경고 7일을 확인했지만 `/etc/login.defs` 파일 자체의 내용은 별도 확인하지 않았다.

새 계정의 기본값을 편집한다.

```bash
sudoedit /etc/login.defs
```

기존의 아래 세 항목을 **중복 없이 수정**한다. 파일 전체를 대체하지 않는다.

```text
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7
```

이미 있는 계정에는 별도로 적용한다.

```bash
sudo chage -M 30 -m 2 -W 7 root
sudo chage -M 30 -m 2 -W 7 jungblee
```

기간 값이 최대 30일·최소 2일·경고 7일인지 확인한다.

```bash
sudo chage -l root
chage -l jungblee
```

`Account expires`는 비밀번호 만료와 별개이므로 이번에는 설정하지 않는다. `chage`는 비밀번호 자체를 바꾸지 않는다. 최소 2일 설정 후 일반 사용자의 재변경이 제한될 수 있다. 두 계정 모두 `Last password change`와 `Password expires`가 `never`였다. 다음에는 품질 정책을 구성하고 실제 비밀번호를 바꾼 뒤, 위 조회로 마지막 변경 날짜·30일 뒤 만료일·2/30/7을 다시 확인한다.

참고: [chage(1)](https://man7.org/linux/man-pages/man1/chage.1.html), [login.defs(5)](https://man7.org/linux/man-pages/man5/login.defs.5.html).

## 4. PAM·authselect 현재 상태 조회 — 실행 결과 대기

품질 정책을 편집하기 전에 현재 프로필과 구성 상태, 비밀번호 처리 경로를 읽는다. 아래 조회의 실제 출력은 아직 받지 않았다.

```bash
sudo authselect current
sudo authselect check
grep -nE '^[[:space:]]*password[[:space:]]' /etc/pam.d/passwd /etc/pam.d/system-auth /etc/pam.d/password-auth
```

`login.defs` drop-in 지원과 관련해 실제 VM의 패키지 의존성을 확인할 조회도 남아 있다.

```bash
rpm -q --requires shadow-utils | grep libeconf
```

이 조회는 지원 여부를 판단할 근거이며 drop-in의 실제 적용 검증은 아니다. 기존 `/etc/login.defs` 편집 절차는 유지한다. 개념과 공식 근거는 [비밀번호·PAM 목표](targets.md#3-비밀번호pam)를 참고한다.

## 5. 이후 순서 — 명령은 진행하며 추가

1. 비밀번호·PAM: 품질 정책, root 예외, 기존 계정 비밀번호 갱신.
2. sudo: 인증 제한·TTY·경로·입출력 로그. 그룹 가입만으로 완료되지 않는다.
3. SSH·firewalld·SELinux: 4242, root SSH 금지, 재부팅 후 정책 유지.
4. monitoring.sh·cron·wall: 필수 지표, 부팅 시·10분마다 방송, 수정 없는 중단.
5. 평가 재현·README·서명: 호스트명 변경·새 계정 실습, 실제 디스크 SHA-1, 스냅샷 규정 확인.
