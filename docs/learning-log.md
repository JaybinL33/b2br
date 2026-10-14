# Born2beRoot 학습 기록

학습 순서와 자료는 [README](../README.md)에 둔다. 이 문서에는 직접 적용하고 확인한 내용을 간결하고 자연스러운 한국어로 남긴다. 참고 자료의 설명, 사용자 보고, 실제 실행 결과를 구분한다. 실습 후에는 무엇을 하려 했는지, 왜 그렇게 골랐는지, 무엇을 관찰했고 어떻게 이해했는지를 필요한 만큼만 적는다. 아직 하지 않은 작업은 완료 기록으로 채우지 않는다.

## 2026-10-12 — 반복 학습의 출발점

**사용자 보고:** 학교 컴퓨터와 개인 NixOS 환경에서 Rocky 10.2 Minimal로 전체 과정을 해봤다. 아직 숙련되지 않아 부분 단위로 다시 실습하며 이해를 다지고 싶다. 여기의 환경과 버전은 사용자가 알려 준 정보다.

**아직 확인하지 않은 상태:** 실제 설치 버전과 VM 구성, 디스크 구조, 보안 정책의 동작, 모니터링 결과, 평가 통과 여부를 독립적으로 검증하지 않았다. 이번 반복 학습에서 직접 수행한 실습 결과나 학습 완료 기록은 아직 없다.

## 2026-10-12 — 새 VM의 실습 기준 선택

사용자가 제공한 VM 생성 화면을 바탕으로 AI와 함께 아래 값을 실습 기준으로 선택했다. 화면의 호스트 메모리는 16 GB다. **과제의 고정 요구값이 아니며, 설정 적용과 VM 생성·부팅은 아직 확인하지 않았다.**

| 항목 | 이번 선택 | 이유 |
|---|---|---|
| 메모리 | 4096 MB | 설치와 반복 실습에 여유를 두면서 호스트에도 메모리를 남긴다. |
| vCPU | 2개 | 최소 서버 실습에 사용할 시작값으로 잡는다. 실제 CPU 호환성과 동작은 부팅 때 확인한다. |
| EFI | 끔 — BIOS 사용 | 이번에는 BIOS 방식으로 부팅 구성을 익힌다. Rocky는 BIOS와 UEFI를 지원하며, 공식 문서의 일반 권장은 UEFI다. |
| 가상 디스크 | 새 VDI, 30 GB | 설치·업데이트·로그와 디스크 구성 실습에 여유를 둔다. 파티션별 용량은 다음 단계에서 정한다. |
| 전체 크기 사전 할당 | 끔 — 동적 할당 | 호스트에서 처음부터 30 GB를 차지하지 않고 기록한 영역에 따라 파일이 커지게 한다. 30 GB는 가상 디스크의 최대 용량이다. |
| 2 GB 단위 분할 | 끔 | 현재 분할 보관이 필요한 조건은 확인되지 않아 단일 파일로 관리한다. |
| 기존 디스크 | `jungblee42.vdi` 재사용 안 함 | 이전 실습 디스크를 보존하고 새 설치 과정을 다시 밟는다. |
| 설치 방식 | 자동 설치를 건너뛰고 수동 설치 | 설치 과정에서 디스크 구조와 각 선택의 이유를 직접 확인한다. |

새 디스크의 저장 경로는 학교 화면에 나온 `/home/jungblee/goinfre/VirtualBox VMs/b2br/b2br.vdi`를 유지하기로 했다. `goinfre`의 보존 정책은 아직 확인하지 않았다. 보관이 필요한 VM은 완전히 종료한 뒤 별도 위치에 복사해 두는 것을 권장했으며, 실제 복사 여부도 미확인이다. VM 파일은 Git에 넣지 않는다.

참고한 공식 자료는 [Rocky Linux 10 하드웨어 요구사항](https://docs.rockylinux.org/guides/minimum_hardware_requirements/)과 [Oracle VirtualBox 7.2 가상 저장장치 문서](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/storage.html)다. Rocky 자료는 10.0 기준이며 RAM 수치가 본문과 요약표 사이에 다르게 적혀 있어 하나의 최소값으로 단정하지 않았다. 이번 4 GB·2 vCPU·30 GB는 해당 문서를 참고한 실습용 판단이다. Oracle 문서에서는 고정 크기와 동적 할당의 차이를 확인했다. 실제 호스트나 VM 설정을 변경한 기록은 아니다.

## 2026-10-12 — 최신 디스크 구성과 확인할 화면

사용자는 보너스를 목표로 하지 않되 필수 서버 관리·모니터링에 적절한 구성을 원한다고 명확히 했다. 재배치와 축소 가능성을 고려한 ext4 선택을 유지하고, home을 4 GiB로 잡고 `/var/log`용 LV를 분리하는 안으로 정리했다. 보너스 그림 전체를 복제하는 안은 아니다. 새 30 GB VDI에 적용할 최신 기준은 이 표다. **실습용 설계 선택이며 과제의 고정 용량 조건이나 적용 완료 기록이 아니다.**

| 대상 | 선택 | 이유 |
|---|---|---|
| `/boot` | 2 GiB, Standard Partition, ext4, 비암호화 | initramfs와 커널 업데이트 여유를 둔다. LVM 밖의 일반 파티션이므로 이후 크기 조정이 더 까다롭다. |
| 나머지 디스크 | LUKS 암호화 장치 위에 PV와 VG `LVM` 구성 | root·home·var_log·swap을 공통 암호화 계층 안에 둔다. |
| LV `root` | 10 GiB, ext4, `/`에 마운트 | 실측 최소값이 아닌 초기 배정이다. 설치 후 사용량과 업데이트 여유를 확인한다. |
| LV `swap` | 4 GiB, swap | RAM 4 GiB에 대한 RHEL 일반 권장표를 참고했다. 최대절전용 크기로 정한 것은 아니다. |
| LV `home` | 4 GiB, ext4, `/home`에 마운트 | 한 명의 실습 파일을 위한 초기 예산이다. |
| LV `var_log` | 2 GiB, ext4, `/var/log`에 마운트 | sudo 입출력 로그와 시스템 로그 증가를 root 공간과 분리한다. |
| VG 잔여 공간 | 약 8 GiB를 LV에 배정하지 않고 유지 | 이후 root·home 확장이나 새 LV 실습에 사용한다. 디스크 바깥의 미분할 공간이 아니라 VG 안의 여유 공간이다. |
| BIOS Boot (`biosboot`) | GPT+BIOS일 때 1 MiB, 일반 파티션, 파일시스템·마운트·암호화 없음 | 부트로더용 영역이며 `/boot`나 EFI 시스템 파티션과 다르다. 설치기에서는 BIOS Boot 유형을 선택한다. |

`/var` 전체·`/tmp`·`/srv`는 별도 마운트하지 않는다. root 10 GiB에는 OS·패키지·캐시·임시파일 등이 함께 들어가므로 home보다 큰 초기 예산을 둔다. `/srv`에는 현재 서비스 데이터를 둘 계획이 없다. 로그 2 GiB는 무한 보관을 보장하지 않으며 보관 정책과 사용량 점검이 필요하다. 가득 차면 sudo에도 영향을 줄 수 있다. 모든 용량은 실측 최소값이 아니며 설치 후 확인해 VG 여유 공간으로 확장한다. 약 8 GiB의 여유는 디스크가 약 30 GiB일 때의 계산으로, 단위·메타데이터·반올림에 따라 달라진다. [목표 문서](targets.md)에 대조할 구조와 확인 방법을 정리했다.

유일한 VG는 LVM 관리 영역임을 알아보기 쉽도록 대문자 `LVM`으로 이름 짓기로 했다. LV 이름은 `root`, `home`, `var_log`, `swap`이다. LVM은 관리 기술의 이름이고, 여기서 VG 이름으로도 사용한다. VG는 PV의 공간을 묶어 LV에 할당한다. 실제 VM에는 아직 적용하지 않았다.

학습자료로 설명한 관계는 `파티션 → LUKS → 열린 암호화 장치 → LVM PV → VG LVM → 각 LV`다. root·home·var_log에는 ext4 파일시스템을 만들고 각각 `/`, `/home`, `/var/log`에 연결한다. swap에는 디렉터리 마운트 지점을 두지 않는다. 원문 인쇄 7쪽도 암호화 장치 안에 여러 LVM 볼륨을 두는 예시이므로 이 계층을 기준으로 삼았다. 사용자가 이 구조를 이해하거나 실제로 구성했는지는 아직 확인하지 않았다.

설치기에서는 `Volume Group` 옆 `Modify`로 연 **VG 설정 창의 `Encrypt`**를 사용하고, `Size policy`는 `As large as possible`로 선택하도록 안내했다. 네 LV는 같은 VG에 두고 개별 LV에 추가 암호화는 설정하지 않는 계획이다. VG는 가능한 크게 확보하되 root·home·var_log·swap에 각각 10·4·2·4 GiB만 배정한다. home 용량을 비워 잔여 전부를 배정하지 않는다. 마운트 지점 편집 영역의 개별 `Encrypt`와 혼동하지 않도록, **`Begin Installation` 전에 파티션 목록과 VG 설정 화면을 요청했다.** 실제 화면과 적용 상태는 아직 미확인이다.

근거는 [과제 원문](../en.subject.pdf), [RHEL 10 설치기 문서의 10.5절](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/interactively_installing_rhel_from_installation_media/customizing-the-system-in-the-installer), [RHEL 10 LUKS 안내](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/encrypting-block-devices-by-using-luks)다. `/boot` 2 GiB는 RHEL의 베어메탈 권장을 참고해 실습 여유로 선택했으며 VM의 필수값은 아니다. LUKS2가 RHEL 기본값이라는 설명도 실제 생성 결과와 구분한다. [Rocky 10 설치 안내](https://docs.rockylinux.org/guides/installation/)는 자동 파티션을 설명하므로 Custom의 세부 조작은 RHEL 문서를 참고했다. 사용 중인 Rocky 설치기의 정확한 UI와 암호화 계층은 화면과 설치 결과로 확인할 예정이다.

ext4 축소 조건은 학습자료로 안내했다. 마운트를 해제하고 파일시스템을 먼저 줄인 뒤 LV를 줄여야 하며, 실행 중인 시스템의 root는 이 조건을 충족할 수 없어 별도 부팅 환경이 필요하다. 이는 사용자의 이해나 실행을 확인한 기록이 아니다. 근거는 [RHEL 10 ext4 크기 조정](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_file_systems/getting-started-with-an-ext4-file-system)의 32.4절과 [LV 축소](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_logical_volumes/basic-logical-volume-management)의 4.3.5절이다. 실제 축소 명령이나 작업은 수행하지 않았다.

## 2026-10-12 — 설치 전 진행 상태

**사용자 보고:** Kdump를 끄고 hostname을 `jungblee42`로 설정했다고 알렸다. hostname 값은 설치 요약 화면에 나오지 않아 사용자 보고로만 남긴다.

**대화에서 화면으로 확인한 내용:** 설치 요약에 Rocky Linux Minimal 10.2 설치기, Minimal Install, Asia/Seoul, Kdump disabled, enp0s3 connected가 표시되었다. 당시 Automatic partitioning이 선택되어 있었고 root 계정은 비활성, 생성 예정 사용자는 없었으며 `Begin Installation` 버튼도 비활성이었다. 설치기의 버전 표시는 설치된 OS나 부팅 결과를 검증한 것이 아니다.

다음 단계로 `Installation Destination → 새 30 GB 디스크 → Custom → Done`을 안내했고 파티션 구성 뒤 계정을 설정할 예정이다. 최신 ext4 구성의 실제 적용·부팅·이해 완료는 아직 확인하지 않았다. 이번 문서 작업에서는 VM·호스트 설정을 변경하거나 서명을 생성하지 않았다.

## 2026-10-12 — 설치 시작 보고

**사용자 보고:** 안내한 구성으로 설치를 시작했다고 알렸다. 실제 파티션 배치·설치 완료·첫 부팅은 아직 검증하지 않았다. 이전 설치 요약 화면은 설치 시작 전의 관찰 기록이다. 다음 학습은 [전체 지도](targets.md)에 따라 설치 결과 확인부터 계정·PAM·sudo·접속 보안·모니터링·평가 재현 순서로 진행한다. 이번에는 학습 지도를 정리했으며 VM 설정이나 실행 검증은 하지 않았다.

## 2026-10-12 — 설치 후 디스크 구조 화면 확인

**대화에서 사용자 스크린샷으로 확인:** 설치 후 `jungblee@jungblee42` 셸에서 `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS`와 `sudo vgs` 결과가 표시되었다. `sudo vgs` 실행은 성공했다. 아래는 화면의 값이며 암호화 장치의 UUID만 생략했다. 앞선 설치 전·시작 단계의 미확인 상태는 당시 기록이다.

```text
NAME              SIZE TYPE  FSTYPE      MOUNTPOINTS
sda                30G disk
├─sda1              1M part
├─sda2              2G part  ext4        /boot
└─sda3             28G part  crypto_LUKS
  └─luks-<UUID>    28G crypt LVM2_member
    ├─LVM-root     10G lvm   ext4        /
    ├─LVM-swap      4G lvm   swap        [SWAP]
    ├─LVM-var_log   2G lvm   ext4        /var/log
    └─LVM-home      4G lvm   ext4        /home
```

| VG | #PV | #LV | #SN | Attr | VSize | VFree |
|---|---|---|---|---|---|---|
| LVM | 1 | 4 | 0 | wz--n- | 27.98g | 7.98g |

화면에서 확인한 계층·LV 용량·파일시스템·마운트 지점은 최신 목표와 일치한다. VG의 `VFree 7.98g`는 아직 LV에 배정하지 않은 공간이다. 각 파일시스템 안의 빈 공간과는 다르며 그 사용량은 별도로 확인해야 한다. 이 차이는 학습 설명으로 다뤘고 사용자의 이해 완료까지 확인한 것은 아니다.

**아직 확인하지 않은 것:** `sda1`의 BIOS Boot 유형은 이 출력만으로 증명되지 않는다. LUKS 버전, 설치된 OS 버전, 전체 보안 정책과 그룹 멤버십도 이번 화면으로 확인하지 않았다. 프롬프트 표시는 관찰했지만 hostname 설정의 별도 조회·지속성 검증을 대신하지 않는다. `sudo vgs` 성공은 sudo 사용 가능 여부의 증거이며 그룹 구성이나 모든 sudo 정책 충족의 증거는 아니다. `#SN 0`은 LVM 스냅샷 수로 VirtualBox 스냅샷 규정 검증과도 다르다.

다음으로 `id` 출력을 요청했다. 이번 문서 작업에서 VM을 직접 조작하거나 비밀번호·암호문구를 기록하지 않았다.

## 2026-10-12 — 그룹 변경 전 확인

대화에서 확인한 `id` 결과는 UID 1000, 기본 GID 1000이며 소속 그룹은 `jungblee`, `wheel`이다. `user42`·`sudo` 가입 적용과 재로그인 후 결과는 아직 미확인이다. 지금까지 다룬 명령은 [최소 치트시트](cheatsheet.md)에 정리했다. 이번 작업은 문서 작성이며 VM에서 그룹 변경 명령을 실행한 것은 아니다.

## 2026-10-12 — 비밀번호 기간 정책 안내

새 계정 기본값인 `/etc/login.defs`와 기존 계정에 적용하는 `chage`의 역할을 구분해 안내했다. 목표는 최대 30일·최소 2일·만료 경고 7일이다. 계정 자체의 만료일은 비밀번호 만료와 별개이며, `chage`는 비밀번호 자체를 변경하지 않는다. 최소 2일 정책은 일반 사용자의 재변경을 제한할 수 있다. 품질 정책과 최종 기존 비밀번호 갱신은 다음 단위다.

이 내용은 학습자료로 안내한 개념이며 사용자 적용이나 이해 완료를 확인한 기록은 아니다. 기간 정책과 `user42`·`sudo` 그룹 가입 모두 아직 적용 결과가 미확인이다. 이번 문서 작업에서 VM 설정은 변경하지 않았다. 명령과 공식 매뉴얼은 [치트시트](cheatsheet.md)에 연결했다.

## 2026-10-13 — 그룹 가입과 비밀번호 기간 값 확인

**사용자 화면에서 확인한 결과:** `id jungblee`는 다음과 같았다.

```text
uid=1000(jungblee) gid=1000(jungblee) groups=1000(jungblee),10(wheel),1001(user42),1002(sudo)
```

계정의 `user42`·`sudo` 가입과 기존 그룹 유지를 확인했다. 이 결과는 현재 로그인 프로세스의 그룹 갱신이나 sudoers 정책을 검증한 것은 아니다. 재로그인 후 인자 없는 `id` 결과는 아직 없다.

root와 jungblee의 `chage -l` 화면에서 두 계정 모두 최소 2일·최대 30일·경고 7일을 확인했다. 그러나 `Last password change`와 `Password expires`는 둘 다 `never`였다. 따라서 기간 값은 확인했지만 실제 비밀번호 만료는 미검증이다. `/etc/login.defs`의 편집 결과도 이 출력만으로 확인할 수 없다.

다음 순서는 품질 정책을 구성한 뒤 실제 비밀번호를 변경하고, 마지막 변경 날짜와 30일 뒤 만료일, 2/30/7을 다시 확인하는 것이다. 아직 품질 정책 적용이나 최종 비밀번호 변경을 완료한 기록은 아니다.

**학습자료로 안내한 개념:** `passwd`의 PAM `password` 스택에서 `pam_pwquality.so`가 품질을 검사하고 뒤의 모듈이 실제 비밀번호 갱신을 이어 간다. authselect 관리 파일을 편집하기 전에 현재 프로필·구성 상태·처리 경로를 조회하기로 했다. 조회 명령은 [치트시트](cheatsheet.md), 공식 근거와 설명은 [목표 문서](targets.md#3-비밀번호pam)에 둔다. 실제 조회 결과와 사용자의 이해 완료는 아직 확인하지 않았다.

`login.defs`의 drop-in은 libeconf를 사용한 shadow-utils 빌드에서 지원하는 기능으로 설명했다. 실제 VM의 libeconf 의존성 조회는 대기 중이며 기존 직접 편집 안내를 대체하지 않는다. drop-in도 기존 계정의 기간 값을 갱신하는 `chage`를 대신하지 않는다.

이번 작업은 제공된 화면 근거와 학습 내용의 문서 동기화다. VM 명령 실행이나 VM·호스트 설정 변경은 수행하지 않았으며 비밀번호는 기록하지 않았다.


## 2026-10-13 — 재실습 절차와 PAM 조회 방식 정리

사용자는 치트시트에 최신 별도 파일 작성 절차가 빠져 있고 PAM 조회 정규식이 과하게 복잡하다고 지적했다. 치트시트의 진행 상태 설명은 위 확인 기록으로 모으고, 필요한 명령을 따라가는 흐름으로 정리했다.

**이번에 정리한 절차:** libeconf 지원 빌드에서는 `/etc/login.defs.d/90-local.defs`에 `PASS_MAX_DAYS 30`, `PASS_MIN_DAYS 2`, `PASS_WARN_AGE 7`을 적는다. 공식 shadow 코드와 libeconf 문서에서 지원 조건과 기본 파일 뒤 drop-in 처리·파일명 순서에 따른 우선순위를 확인했다. 미지원의 원본 편집은 대안으로만 두며, 기존 원본 변경은 복구하거나 삭제하지 않는다. 기존 계정에는 별도로 `chage`를 사용한다.

PAM은 `cat /etc/pam.d/passwd`로 시작하고 그 파일이 참조하는 `system-auth`를 따라 읽는다. authselect 조회는 관리 상태를 확인하는 용도로 유지했다. 품질 정책 다음에 root와 jungblee의 비밀번호를 실제 변경하고 날짜와 2/30/7을 대조하는 순서도 유지했다. 이전 비밀번호 비교 경로를 확인하기 전에는 최종 변경 명령을 확정하지 않았다.

이는 학습 절차의 갱신이며 새 VM 실행 결과를 추가한 것은 아니다. 앞서 확인한 그룹·기간 값과 `never` 출력은 위 기록에 남긴다. 사용자가 다음 단계로 진행하면 학습을 이어가되, 그 진행 자체를 출력 검증으로 간주하지 않는다. 이번 작업에서는 문서만 수정했다.


## 2026-10-13 — authselect와 PAM 품질 모듈 연결 확인

**사용자 화면에서 확인:** `authselect current`는 `Profile ID: local`, `Enabled features: None`을 표시했고, `authselect check`는 `Current configuration is valid.`를 반환했다.

`/etc/pam.d/passwd`의 비밀번호 처리 부분은 다음 순서였다.

```text
password substack system-auth
password optional pam_gnome_keyring.so use_authtok
password substack postlogin
```

`system-auth`에는 authselect가 생성한 파일이라는 경고와 다음 비밀번호 스택이 있었다.

```text
password requisite pam_pwquality.so
password sufficient pam_unix.so yescrypt shadow nullok use_authtok
password required pam_deny.so
```

**화면을 바탕으로 설명한 흐름:** `passwd`가 `system-auth`를 호출한다. 품질 모듈이 실패를 반환하면 `requisite` 제어로 해당 스택이 실패하고, 통과한 새 비밀번호는 `pam_unix.so`가 `use_authtok`로 받아 로컬 갱신에 사용한다. 품질 모듈은 이미 연결되어 있어 PAM 직접 편집은 필요하지 않다. 구성 유효와 모듈 연결 확인을 과제의 품질 규칙 적용 완료로 보지는 않는다.

다음 조회는 `cat /etc/security/pwquality.conf`와 `ls /etc/security/pwquality.conf.d`다. drop-in보다 본체가 나중에 읽히고 PAM 인수가 우선하므로 본체의 중복 설정도 살핀 뒤 규칙을 작성할 예정이다. 이 두 조회의 출력과 품질 규칙 작성·적용 결과는 아직 추가되지 않았다. 이번 작업은 사용자 화면 근거의 문서화이며 VM 설정은 변경하지 않았다.


## 2026-10-13 — 학습 자료와 읽기 범위 점검

사용자는 자료·필독 범위·설명할 개념·존재만 알아둘 기능·현재 범위에서 미룰 내용과 그 근거가 충분한지 점검을 요청했다. 기존 문서는 전체 지도와 목표·검증 구분은 갖췄지만 스토리지 외 구간은 읽을 절과 깊이 선택의 이유가 부족했다.

목표 문서의 일곱 구간에 공식 입문 자료와 정확한 제목·절, 설정 시 찾을 매뉴얼 항목을 연결했다. 과제의 필수 조건과 교육적 범위 판단을 구분하고 미룬 내용을 다시 읽을 상황을 적었다. Rocky 입문서의 일반 개념, Rocky 10 설치 안내, RHEL 10 문서, 실제 설치 버전의 역할을 나눴다. login.defs와 libpwquality의 우선순위 차이는 실제 적용값에 영향을 주므로 필수 이해 범위에 넣었다.

이는 교육 문서 감사와 읽기 안내 보완이다. 새 구간을 학습·구현·검증했다는 기록이 아니며 VM 설정은 변경하지 않았다. 치트시트는 실행 흐름으로 유지하고 긴 읽기 안내를 중복하지 않았다.


## 2026-10-13 — 기존 품질 설정 보고와 별도 파일 작성 안내

**사용자 보고:** `cat /etc/security/pwquality.conf` 출력은 모두 주석이고 `ls /etc/security/pwquality.conf.d` 결과는 비어 있다고 알렸다. 이는 사용자 보고로 기록하며 새 화면이나 직접 실행 검증으로 바꾸어 적지 않는다. 앞서 화면에서 확인한 authselect `local`·구성 유효 상태와 인수 없는 `pam_pwquality.so` 연결을 함께 근거로 삼았다.

`sudoedit /etc/security/pwquality.conf.d/90-local.conf`로 별도 파일 작성을 안내했다. 길이 10, 대문자·소문자·숫자 최소 1개, 기타 문자 가산 없음, 연속 동일 문자 최대 3개, 사용자명 검사, 이전 비밀번호와 차이 7, 실패 거절과 root 품질 검사를 명시한다. 파일 내용은 [치트시트](cheatsheet.md#5-별도-품질-설정-파일-작성), 옵션 의미와 공식 근거는 [목표 문서](targets.md#품질-규칙과-선택-이유)에 둔다. 사전 검사 등 기본값은 유지한다.

현재는 작성 안내까지다. 실제 파일 저장·정책 적용·허용 및 거절 동작을 완료했다고 기록하지 않는다. 다음 묶음은 root와 jungblee의 실제 비밀번호 변경, 일반 사용자 직접 변경 경로의 이전 비밀번호 비교, 마지막 변경일·30일 뒤 만료·2/30/7 확인이다. 이번 작업에서 VM은 조작하지 않았으며 비밀번호는 기록하지 않았다.


## 2026-10-13 — 비밀번호 변경 성공과 만료일 확인

첫 화면에서 값 없는 `enforce_for_root` 저장·편집 종료와 jungblee의 sudo 없는 `passwd` 실행을 확인했다. `Current password`, `New password`, `Retype new password` 입력 뒤 `password updated successfully`가 표시됐다. 비밀번호 내용은 기록하지 않는다.

다음 화면에서 jungblee 로그인 후 `chage -l jungblee`와 `sudo chage -l root` 결과는 같았다.

| 항목 | 두 계정의 확인값 |
|---|---|
| Last password change | Oct 13, 2026 |
| Password expires | Nov 12, 2026 |
| Password inactive | never |
| Account expires | never |
| Minimum / Maximum / Warning | 2 / 30 / 7 |

jungblee의 변경 성공과 두 계정의 변경일·30일 뒤 만료일·기간 값을 확인해 날짜·기간 체크포인트를 완료했다. root 변경 명령 자체의 성공 메시지는 화면에 없었다. 이전 기록의 변경일·만료일 `never`는 변경 전 상태이며, 현재의 비활성 기한·계정 만료 `never`는 이번에 추가 설정할 항목이 아니다.

정상 변경 성공과 각 품질 위반 조건의 거절 검증은 다르므로 거절 테스트는 남은 항목으로 둔다. 다음 학습은 sudo 권한과 감사 로그다. 이번 작업은 화면 근거의 문서화이며 새 sudo 설정 작성이나 VM 조작은 하지 않았다.


## 2026-10-13 — sudo 현재 정책 읽기로 진행

사용자 요청에 따라 현재 설정과 무관해진 품질 옵션의 표기·파서 설명을 정리하고, 값 없는 `enforce_for_root`와 root 품질 검사·이전 비밀번호 비교 예외 설명을 유지했다. 실제 변경 성공과 만료일 확인 기록은 그대로 둔다.

다음 학습은 sudo의 권한 허용·실행 조건·입출력 기록 세 묶음이다. 먼저 `sudo -l`, `sudo cat /etc/sudoers`, `sudo ls /etc/sudoers.d`를 조회하도록 안내했다. 확인된 그룹 가입과 실제 허용 규칙을 구분하고, 활성 wheel 그룹 규칙과 include 지시를 찾아 읽는다. 공식 sudoers·visudo 매뉴얼의 읽을 부분은 [목표 문서](targets.md#4-sudo-정책과-감사-로그)에 연결했다.

현재 정책 조회 출력은 아직 받지 않았다. 별도 파일의 visudo 편집은 다음 단계이며 새 sudo 정책 적용이나 VM 조작은 하지 않았다.


## 2026-10-13 — sudo 권한 확인과 별도 설정 안내

**화면 확인:** `sudo -l`에 `(ALL) ALL`, `secure_path=/sbin:/bin:/usr/sbin:/usr/bin`과 기본 환경 항목이 보였다. sudoers 본체에는 활성 `%wheel ALL=(ALL) ALL`, `root ALL=(ALL) ALL`, `#includedir /etc/sudoers.d`가 있었고 NOPASSWD wheel 규칙은 주석이었다. **사용자 보고:** `/etc/sudoers.d` 목록은 비어 있었다.

wheel 규칙을 구성원·호스트·대상 사용자·명령의 네 부분으로 읽었다. secure_path는 sudo의 PATH이며 절대 경로 명령의 허용 목록은 아니라는 점을 설명했다. 기존 권한과 경로는 유지하고 별도 파일 `90-b2br`에 인증 3회·선택한 오류 문구·TTY 요구·입출력 로그와 경로를 작성하도록 안내했다. 구체적인 내용은 [치트시트](cheatsheet.md#8-sudo-실행-조건과-입출력-로그-설정)에 둔다.

저장 후 현재 터미널을 유지하며 전체 문법 검사와 적용 목록을 확인하는 순서다. 실제 저장·검사 출력, 실패 횟수·TTY 거절·입출력 로그 동작은 아직 확인하지 않았다. 이번 작업은 문서 갱신이며 VM 설정은 변경하지 않았다.


## 2026-10-13 — sudo 파일 모드 오류 확인과 교정 안내

사용자 화면에서 `/etc/sudoers: parsed OK`가 표시됐지만 `/etc/sudoers.d/90-b2br: bad permissions, should be mode 0440` 오류가 이어졌다. `sudo -l`에는 인증 3회·지정 오류 메시지·requiretty·입출력 기록·`iolog_dir=/var/log/sudo`가 모두 표시됐다. 적용 목록을 확인했으나 전체 검사는 통과하지 못한 상태다.

`sudo chmod 0440 /etc/sudoers.d/90-b2br` 후 `sudo visudo -c`로 재검사하도록 안내했다. 두 파일 모두 `parsed OK`이고 오류가 없는 것이 완료 기준이다. 교정 성공은 아직 확인하지 않았다. 소유권 변경은 안내하지 않았다.

공식 visudo 매뉴얼의 `-P` 기본 활성 조건을 확인해, 별도 파일 편집 시 문법 검사와 전체 검사에서의 소유권·모드 확인을 구분했다. 당시 chmod 교정을 안내했으며 기본 재실습 절차는 아래처럼 개선했다. 이번 작업에서 VM은 조작하지 않았다.


## 2026-10-13 — visudo에서 파일 모드까지 처리하도록 개선

사용자는 작성 후 chmod가 유일하거나 더 일반적인 방법인지 물었다. 공식 visudo 매뉴얼의 `-P`는 편집 모드에서 기본 sudoers 모드를 설정하므로, 재실습 기본 흐름을 `sudo visudo -P -f /etc/sudoers.d/90-b2br` → 저장 → `sudo visudo -c` → `sudo -l`로 바꿨다. 보급도에 대한 판단이 아니라 같은 도구로 모드까지 처리해 누락 단계를 줄이려는 선택이다.

chmod 0440은 이미 발생한 오류의 유효한 교정으로 남긴다. 이미 교정했다면 재편집하지 않고 전체 검사만 진행하면 된다. 교정 성공 출력은 아직 받지 않았으며 VM 조작은 하지 않았다.


## 2026-10-13 — sudo 파일 검사 통과와 로그 재생 안내

사용자 화면에서 `sudo chmod 0440 /etc/sudoers.d/90-b2br` 실행 뒤 `sudo visudo -c`가 `/etc/sudoers`와 `/etc/sudoers.d/90-b2br` 모두 `parsed OK`를 표시했다. `sudo -l`에도 안내한 다섯 설정 줄의 항목이 유지됐다. 모드 오류 교정과 전체 검사 체크포인트를 완료했다. 실제 복구는 chmod였으며 재실습 권장 흐름인 `visudo -P -f`는 유지한다.

다음으로 `sudo cat`에 비밀이 아닌 `b2br-log-test`를 입력하고 Enter, 빈 줄에서 Ctrl+D로 종료하도록 안내했다. 지정 로그 경로에서 sudoreplay 목록을 보고 cat의 TSID로 재생하는 순서는 [치트시트](cheatsheet.md#9-로그-생성과-출력-재생)에 추가했다. 문장 재생은 기록 생성과 출력 재생의 증거이며 모든 입력 스트림 검증과는 구분한다.

로그 실행·재생 결과는 아직 받지 않았다. 이후 인증 실패 횟수와 TTY 조건을 확인한다. 이번 작업은 문서 동기화이며 VM 명령을 실행하지 않았다.


## 2026-10-13 — 로그 생성과 출력 재생 확인

사용자 화면에서 `sudo cat`에 `b2br-log-test`를 입력한 뒤, `sudo sudoreplay -d /var/log/sudo -l` 목록의 첫 다섯 sudo 명령 기록과 cat 세션 TSID `000004`를 확인했다. 해당 세션 재생에서 `Replaying sudo session: /bin/cat`과 문장이 두 번 표시됐다. 처음 ID의 자리수를 잘못 입력한 오류는 실제 ID로 고쳐 재생했다.

로그 생성과 출력 재생 체크포인트를 완료했다. 기본 재생은 출력 스트림을 보여 주므로 문장 두 번을 입력 스트림 검증으로 간주하지 않는다. 입력 스트림 명시 검증, 인증 실패 횟수·TTY 조건 검증은 남아 있다.

사용자는 치트시트가 처음부터 재실습할 때 보는 문서임을 분명히 했다. 전체 치트시트에서 사용자 보고·완료 상태·결과 대기·복구 이력을 제거하고 절차·설정·기대 결과·조건별 분기만 남겼다. 설명은 targets, 현황은 README, 증거와 이력은 이 기록으로 구분한다. 이번 작업에서 VM은 조작하지 않았다.


같은 cat 세션의 입력 확인으로 `sudo sudoreplay -d /var/log/sudo -f ttyin 000004`를 안내했다. 터미널 입력에서 `b2br-log-test`를 확인하는 절차를 안내했으며, 결과 판단은 아래 정정 기록에 둔다. 치트시트에는 세션 고유 번호 대신 목록에서 찾은 `ID`를 쓰도록 했다.


## 2026-10-13 — 입력 재생 결과 정정

사용자는 “재생은 성공한거야”라고 명시했다. 첨부 화면에서도 `sudo sudoreplay -d /var/log/sudo -f ttyin 000004` 재생 중 `b2br-log-test`가 보였고, 종료 뒤 그 줄이 사라진 것을 확인했다. 입력 재생 성공은 사용자 확인과 화면으로 확인됐다. 종료 후 화면만으로 미확정으로 취급한 판단을 정정하고 추가 진단은 정상 학습 흐름에서 제거한다.

## 2026-10-13 — 목표 문서의 역할 정리

targets 전체에서 실제 출력·날짜·사용자 보고·진행 상태를 분리했다. 설치·계정·PAM·기간·sudo 확인 사실은 이 문서의 기존 기록에 이미 있어 중복해서 옮기지 않았다. 목표 문서에는 목표 상태, 개념, 선택 근거, 공식 자료의 읽을 범위·깊이와 재사용 가능한 검증 기준을 유지했다. 현황은 README, 증거·이력은 학습 기록, 절차는 치트시트에 둔다. 이번 편집에서 VM은 조작하지 않았다.


## 2026-10-13 — TTY 검증 절차 준비

공식 sudo·sudoers, util-linux setsid, Linux setsid(2) 매뉴얼과 대조해 일반 사용자 셸에서 `setsid -w sudo -n true </dev/null`로 TTY 요구를 확인하는 절차를 정리했다. targets에는 개념·읽을 범위, 치트시트에는 명령과 거절 판단 기준을 추가했다. 비밀번호 요구나 임의의 실패를 TTY 검증 성공으로 간주하지 않는다.

이번 변경은 학습 절차의 문서화다. 사용자 VM에서 명령을 실행하지 않았고 결과도 받지 않았으므로 TTY 검증은 미완료다. 기존 로그 입력·출력 재생 성공 기록은 유지한다.


## 2026-10-13 — TTY 거절 확인과 sudo 인증 경로 조회 안내

사용자 첨부 화면을 확인한 주 대화의 전달 근거: jungblee 일반 셸에서 `setsid -w sudo -n true </dev/null` 실행 뒤 `sudo: sorry, you must have a tty to run sudo`가 표시됐다. 명시적인 TTY 거절로 requiretty 체크포인트를 완료했다. 화면에 `echo $?` 결과는 없지만 이를 이유로 추가 재검사를 요구하지 않는다.

다음 검사인 `passwd_tries=3`·`badpass_message`에 앞서 `cat /etc/pam.d/sudo`를 안내했다. 앞서 본 system-auth에는 faillock이 없었지만 sudo의 실제 auth·account 호출 경로를 확인해야 한다. 이번 조회 결과는 아직 받지 않았다. 공식 sudo·sudoers·Linux-PAM 매뉴얼을 대조해 인증 경로 조회와 잠금 위험 확인을 선행 조건으로 둔 실패 검사 절차를 준비했다. 실패 횟수·사용자 지정 오류 메시지의 실제 동작은 미검증이다. 이번 문서 작업에서 VM은 조작하지 않았다.


## 2026-10-13 — sudo의 system-auth 연결 확인

사용자 첨부 화면을 확인한 주 대화의 전달 근거: `cat /etc/pam.d/sudo` 출력은 PAM-1.0 헤더 아래 다음 네 줄이었다.

```text
auth include system-auth
account include system-auth
password include system-auth
session include system-auth
```

sudo의 auth·account 처리도 `system-auth`로 연결됨을 확인했다. 앞서 확인한 system-auth auth 스택에는 `pam_faillock`이 없었다. 이를 근거로 일반 사용자 터미널에서 `sudo -k` 다음 `sudo -v`를 실행하고, 틀린 입력 세 번으로 지정 오류 문구와 네 번째 요청 없이 종료되는지를 한 번만 확인하도록 안내했다. 기존 targets·치트시트의 인증 경로 확인 후 검사 순서와 일치한다.

검사 결과는 아직 받지 않았으므로 `passwd_tries=3`·`badpass_message`의 동작 검증은 미완료다. 이번 작업은 전달된 화면 근거의 문서화이며 VM 명령 실행이나 보안 설정 변경은 하지 않았다.


## 2026-10-13 — sudo 인증 실패 검사 완료와 원격 접속 구간 시작

사용자 최신 화면을 확인한 주 대화의 전달 근거: `sudo -k` 후 `sudo -v`에서 비밀번호 요청 세 번, 재시도 중 `Authentication failed. Please try again.` 두 번, 마지막 `sudo: 3 incorrect password attempts`와 셸 복귀가 보였다. `passwd_tries=3`과 `badpass_message` 체크포인트를 완료했다. 그 뒤 대기 중인 프롬프트는 새로 실행한 `sudo -v`의 요청이므로 이전 호출의 네 번째 시도가 아니다. 올바른 비밀번호 입력 또는 Ctrl+C를 안내했다.

이어서 sshd·firewalld·SELinux의 역할과 일반 사용자 4242 접속·root SSH 거절 목표를 설명했다. `systemctl is-active sshd firewalld`, `systemctl is-enabled sshd firewalld`, `getenforce`, `ip -br addr` 조회를 안내했으며 결과는 아직 받지 않았다.


## 2026-10-14 — 원격 접속 구간 시작 상태 확인

사용자 화면을 확인한 주 대화의 전달 근거: sshd와 firewalld는 모두 `active`·`enabled`, SELinux는 `Enforcing`이었다. `enp0s3`는 `UP`이며 IPv4 주소는 `10.0.2.15/24`였다. 이 주소만으로 NAT 모드를 확정하지 않는다.

다음 조회는 `sudo cat /etc/ssh/sshd_config`와 `sudo ls /etc/ssh/sshd_config.d`다. 설정 내용·4242 리스닝·일반 사용자 접속·root SSH 거절은 아직 확인하지 않았다.


## 2026-10-14 — SSH 포함 설정 확인과 별도 파일 작성 안내

주 대화가 사용자 화면 여섯 개에서 확인한 내용: 본체 상단에 활성 `Include /etc/ssh/sshd_config.d/*.conf`가 있고 `#Port 22`, `#PermitRootLogin prohibit-password`, 마지막 Match 예시는 모두 주석이었다. 활성 항목으로 `AuthorizedKeysFile .ssh/authorized_keys`와 `Subsystem sftp /usr/libexec/openssh/sftp-server`를 확인했다.

디렉터리에는 `40-redhat-crypto-policies.conf`와 `50-redhat.conf`가 있었다. 40 파일은 `Include /etc/crypto-policies/back-ends/opensshserver.config`를 지정하며 해당 포함 파일 내부는 미확인이다. 50 파일의 활성 값은 다음과 같다.

```text
SyslogFacility AUTHPRIV
ChallengeResponseAuthentication no
GSSAPIAuthentication yes
GSSAPICleanupCredentials no
UsePAM yes
X11Forwarding yes
PrintMotd no
```

`sudoedit /etc/ssh/sshd_config.d/00-b2br.conf`로 `Port 4242`·`PermitRootLogin no` 두 줄을 작성하고 `sudo sshd -t`와 `sudo sshd -T`의 해당 출력으로 검사하도록 안내했다. 파일 저장·검사 결과는 아직 없으며 SELinux 허용 준비 전에는 재시작하지 않도록 했다.


## 2026-10-14 — SSH 설정 검사 통과와 SELinux 포트 등록 안내

사용자 화면을 확인한 주 대화의 전달 근거: `sudo sshd -t`는 암호 입력 후 오류 없이 종료했고, `sudo sshd -T`의 필터 출력은 `port 4242`와 `permitrootlogin no`였다. 설정 검사 체크포인트를 완료했다. 실행 중 서비스의 반영이나 실제 접속 성공까지 확인한 것은 아니다.

다음으로 `policycoreutils-python-utils` 설치, `semanage port -a -t ssh_port_t -p tcp 4242` 등록, `semanage port -l`의 SSH 유형 조회를 sudo로 실행하도록 안내했다. 관리 도구·옵션과 포트 유형 등록의 목적, 방화벽 개방과의 차이를 설명하고 이미 정의 오류에는 임의의 `-m` 변경 대신 기존 유형을 확인하도록 했다. 실행 결과는 아직 없으며 다음은 firewalld 현황 확인이다.


## 2026-10-14 — SELinux 포트 등록 확인과 firewalld 조회 안내

주 대화가 확인한 사용자 화면에서 `policycoreutils-python-utils` 설치가 `Complete`로 끝났고, 최종 `sudo semanage port -l | grep ssh_port_t` 출력은 `ssh_port_t tcp 4242, 22`였다. SELinux의 TCP 4242 등록 체크포인트를 완료했다. 처음 유형 이름을 `ssh_port`로 입력해 invalid type 오류가 났으나 사용자가 `ssh_port_t`로 고쳐 재실행했다. 목록의 22는 실제 리스닝·방화벽 개방과 다르며 삭제를 지시하지 않았다.

다음으로 `sudo firewall-cmd --get-active-zones`와 `sudo firewall-cmd --list-all`을 안내했다. 기본 zone 출력이 `enp0s3`의 활성 zone과 같은지 대조하도록 설명했다. 조회 결과는 아직 없으며 zone 이름·허용 서비스는 미확인이다.


## 2026-10-14 — firewalld 변경 확인과 sshd 재시작 안내

주 대화에서 전달한 화면 근거: 변경 전 기본·활성 zone은 `public`, 인터페이스는 `enp0s3`, 허용 서비스는 `cockpit dhcpv6-client ssh`였다. permanent public에 `4242/tcp` 추가, 세 서비스 제거, reload가 모두 `success`였다. IPv4 실습의 선택이며 DHCPv6 환경에 같은 제거를 무조건 적용하지 않도록 설명했다.

변경 후 runtime·permanent의 `--list-all` 모두 `services`는 비어 있고 `ports`는 `4242/tcp`였다. runtime 인터페이스는 `enp0s3`, permanent 인터페이스 칸은 비어 있었다. 방화벽 변경과 두 구성의 대조를 확인했다.

다음으로 `sudo systemctl restart sshd`, `sudo systemctl is-active sshd`, `sudo ss -lntp`를 안내했다. 결과는 아직 없으며 4242 리스닝·22 리스닝 없음과 호스트 SSH 접속, VirtualBox 네트워크 설정 확인은 남아 있다.


## 2026-10-14 — SSH 리스닝 확인과 NAT 전달·접속 안내

주 대화가 확인한 화면에서 sshd 재시작은 오류가 없었고 `is-active`는 `active`였다. `ss -lntp`에는 sshd의 `0.0.0.0:4242`와 `[::]:4242` LISTEN 두 줄뿐이며 22 리스닝은 없었다. 서비스 반영·리스닝 체크포인트를 완료했다.

이후 VirtualBox Port Forwarding Rules 화면에는 `Rule1`, TCP, Host IP 빈칸, Host Port 0, Guest IP 빈칸, Guest Port 0의 미완성 행이 있었다. Name `ssh`, TCP, Host IP `127.0.0.1`, Host Port·Guest Port `4242`를 안내했다. 처음 Guest IP `10.0.2.15`를 제안했으나 사용자의 질문 후 일반 NAT 내장 DHCP라면 생략 가능하므로 빈칸을 권장하고 수동 고정 IP일 때 지정하는 것으로 설명을 정정했다. 어댑터 방식과 DHCP 자체는 직접 확인하지 않았다.

저장 후 호스트에서 `ssh -p 4242 jungblee@127.0.0.1`로 접속하고, 최초 키 지문을 VM 콘솔에서 해당 알고리즘의 호스트 공개키와 대조하도록 안내했다. 일치하면 승인하고 터미널에서 비밀번호를 입력한 뒤 `hostname`·`whoami`로 `jungblee42`·`jungblee`를 확인하는 것이 목표다. 실제 규칙 저장·접속 결과는 아직 없다.


## 2026-10-14 — 호스트 SSH의 인증 전 연결 종료 관측

주 대화가 확인한 사용자 포트 전달 화면은 Name `ssh`, TCP, Host `127.0.0.1:4242`, Guest `10.0.2.15:4242`였다. Guest IP 생략 가능 설명과 별개로 사용자는 명시한 주소를 유지했다.

호스트에서 `ssh -p 4242 jungblee@127.0.0.1`과 `ssh -p 4242 jungblee@localhost` 모두 `kex_exchange_identification: Connection closed by remote host`, `Connection closed by 127.0.0.1 port 4242`로 실패했다. 사용자 인증 전 연결 종료이며 원인은 아직 확인되지 않았다. 이를 비밀번호 오류나 root SSH 거절 검증으로 해석하지 않는다.

VM 콘솔에서 `ip -br addr`와 `sudo journalctl -u sshd -n 30 --no-pager`를 요청했다. 현재 Guest IP와 sshd 도달·거절 로그를 확인할 단계이며 결과는 아직 없다.
