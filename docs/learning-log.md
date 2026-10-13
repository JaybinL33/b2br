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
