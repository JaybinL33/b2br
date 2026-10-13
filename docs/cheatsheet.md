# Born2beRoot 최소 치트시트

`jungblee`의 일반 셸에서 **필요한 줄만 개별 실행**한다. 재실습 흐름이며 자동 실행 스크립트가 아니다. 개념·근거는 [targets](targets.md), 진행 상태와 확인한 출력은 [학습 기록](learning-log.md)에 둔다.

## 1. 디스크·VG 확인

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo vgs
```

## 2. 사용자·그룹

```bash
id
getent group user42 sudo
id jungblee
```

없는 그룹만 만들고, 계정에 빠진 그룹이 있을 때 추가한다. 이미 가입되어 있으면 변경을 건너뛴다.

```bash
sudo groupadd user42
sudo groupadd sudo
sudo usermod -aG user42,sudo jungblee
id jungblee
```

그룹을 변경했다면 `exit`로 로그아웃하고 다시 로그인한 뒤 `id`로 세션 반영을 확인한다.

## 3. 비밀번호 기간 정책

새 계정 기본값은 아래 **두 방식 중 하나만** 사용한다. 먼저 libeconf 지원 빌드인지 의존성을 조회한다.

```bash
rpm -q --requires shadow-utils | grep libeconf
```

**libeconf 지원 시:** 별도 파일을 만들거나 편집한다.

```bash
sudo mkdir -p /etc/login.defs.d
sudoedit /etc/login.defs.d/90-local.defs
```

파일에 다음 세 값을 적는다. 기존 파일이면 해당 항목을 중복 없이 수정한다.

```text
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7
```

**미지원 시 대안:** 별도 파일 방식 대신 `sudoedit /etc/login.defs`로 위 세 항목을 수정한다. 지원 여부가 불분명하면 패키지 정보를 먼저 확인한다. 원본 파일의 기존 변경을 임의로 복구하거나 삭제하지 않는다.

별도 파일은 기본 파일 뒤에 읽히며, drop-in은 파일명 사전순으로 처리된다. `90-local.defs`보다 뒤의 파일이 같은 값을 덮을 수 있다. [지원 조건·우선순위 근거](targets.md#3-비밀번호pam).

기존 계정에는 어느 방식이든 `chage`로 기간 값을 적용한다.

```bash
sudo chage -M 30 -m 2 -W 7 root
sudo chage -M 30 -m 2 -W 7 jungblee
sudo chage -l root
chage -l jungblee
```

최소 2일·최대 30일·경고 7일을 대조한다. `chage`는 비밀번호 자체를 바꾸지 않으며 `Account expires`는 비밀번호 만료와 별개다. 최소 2일은 일반 사용자의 재변경을 제한할 수 있다.

참고: [chage(1)](https://man7.org/linux/man-pages/man1/chage.1.html), [login.defs(5)](https://man7.org/linux/man-pages/man5/login.defs.5.html).

## 4. PAM 진입점과 관리 상태 읽기

먼저 비밀번호 변경의 진입점을 읽는다.

```bash
cat /etc/pam.d/passwd
```

그 파일의 `password` 항목이 `system-auth`를 참조하면 이어서 읽는다.

```bash
cat /etc/pam.d/system-auth
```

편집 전에 authselect의 현재 프로필과 관리 구성의 일관성을 확인한다.

```bash
sudo authselect current
sudo authselect check
```

품질 모듈은 이미 연결되어 있다. PAM 파일을 직접 편집하지 않고 기존 품질 설정을 읽는다.

```bash
cat /etc/security/pwquality.conf
ls /etc/security/pwquality.conf.d
```

libpwquality는 drop-in → 본체 순으로 읽으며 PAM 인수가 우선한다. 본체에 같은 활성 설정이 있는지 함께 확인한 뒤 규칙을 작성한다. [확인된 연결과 우선순위 설명](targets.md#3-비밀번호pam).

## 5. 다음 비밀번호 단계

품질 정책을 구성한 뒤 root와 jungblee의 실제 비밀번호를 변경하고 다음 조회로 **마지막 변경 날짜·30일 뒤 만료일·2/30/7**을 확인한다.

```bash
sudo chage -l root
chage -l jungblee
```

품질 정책은 다음 학습 단계다. 최종 비밀번호 변경 명령은 이전 비밀번호 비교가 적용되는 경로를 확인한 뒤 정한다.

이후에는 sudo 정책·로그 → SSH·firewalld·SELinux → monitoring.sh·cron·wall → 평가 재현·README·서명 순으로 진행하며 명령을 추가한다.
