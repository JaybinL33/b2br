# Born2beRoot 학습 기록

학습 순서와 자료는 [README](../README.md)에 둔다. 이 문서에는 직접 적용하고 확인한 내용을 간결하고 자연스러운 한국어로 남긴다. 참고 자료의 설명, 사용자 보고, 실제 실행 결과를 구분한다. 실습 후에는 무엇을 하려 했는지, 왜 그렇게 골랐는지, 무엇을 관찰했고 어떻게 이해했는지를 필요한 만큼만 적는다. 아직 하지 않은 작업은 완료 기록으로 채우지 않는다. 각 절은 당시 상태를 보존한다. 과거의 “미확인·대기”는 뒤의 확인 기록으로 갱신될 수 있으며 현재 상태는 README와 최신 기록을 함께 읽는다.

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


## 2026-10-14 — VM 주소·sshd 로그 확인과 내부 접속 분리 검사

주 대화가 확인한 사용자 화면에서 `ip -br addr`는 `enp0s3 UP 10.0.2.15/24`를 표시했다. `journalctl -u sshd -n 30 --no-pager`에는 14:56·15:11의 서비스 시작·정상 재시작과 `0.0.0.0`·`::`의 4242 리스닝 기록만 있었으며, 조회 범위에 실패한 접속 관련 로그는 없었다. 이 사실만으로 요청이 sshd에 도달하지 않았다고 확정하지 않으며 연결 종료 원인은 미확정이다.

다음으로 VM 콘솔 안에서 `ssh -p 4242 jungblee@127.0.0.1`로 SSH 자체 연결을 확인하고 로그인되면 `exit`하도록 안내했다. VirtualBox 설정의 네트워크 어댑터 1 전체 화면도 요청했다. 내부 접속 결과와 어댑터 설정은 아직 확인되지 않았다.


## 2026-10-14 — VM 내부 SSH 응답과 NAT 어댑터 확인

주 대화가 확인한 화면에서 VM 내부의 `ssh -p 4242 jungblee@127.0.0.1`은 ED25519 fingerprint 확인 질문까지 도달했다. SSH 응답은 확인했지만 키 지문 대조·승인이나 사용자 인증 성공까지 확인한 것은 아니다. 어댑터 1 전체 화면에는 NAT, Enable Network Adapter 체크, Cable Connected 체크가 보였다. 포트 전달 값은 앞서 확인한 `127.0.0.1:4242` → `10.0.2.15:4242`다.

사용자가 VM 설정 창을 열어 둔 상태여서 바깥 설정 창도 OK로 저장한 뒤 호스트 SSH 재시도를 안내했다. 같은 실패가 이어지면 호스트에서 `ss -lntp 'sport = :4242'`로 포트 점유 프로세스를 확인하도록 했다. 재시도·포트 조회 결과는 아직 없으며, 저장 미완료가 연결 종료의 원인이라고 확정하지 않는다.


## 2026-10-14 — 호스트 리스닝·VirtualBox 전달 규칙 확인

주 대화에서 전달한 관측: 호스트 `ss`에는 `LISTEN 127.0.0.1:4242`, `Send-Q 5`가 표시됐고 Process 칸은 비어 있었다. 따라서 이 출력으로 점유 프로세스를 특정하지 않는다. `VBoxManage showvminfo b2br`의 NIC 1은 NAT, cable on, MAC `08002732C370`이며 규칙 `ssh`는 TCP host `127.0.0.1:4242` → guest `10.0.2.15:4242`였다.

VM 루프백 SSH는 호스트 키 질문까지 응답하지만 호스트 SSH의 인증 전 연결 종료 원인은 여전히 미확정이다. 패킷 관찰을 위해 요청한 tcpdump는 사용자가 없다고 답했다. 설치는 진행하지 않고 호스트에서 다음 로그 조회를 안내했다.

```bash
VBoxManage showvminfo b2br --log 0 | grep -i -e nat -e 4242 -e forward -e bind
```

NAT·포워딩·바인딩 로그 결과는 아직 없다.


## 2026-10-14 — NAT 전달 적용 실패 확인과 호스트 포트 변경 안내

주 대화가 확인한 VBox.log 화면에는 다음 오류가 있었다.

```text
drvNATNotifyApplyPortForwardCommand: Port forward modify FAIL!
VERR_NAT_REDIR_SETUP
NAT#0 configuration error failed to set up redirection of 4242 to 4242
Probably a conflict with existing services or other rules.
```

저장된 전달 규칙이 보여도 실제 전달 적용에 성공한 것은 아님을 확인했다. 로그는 기존 서비스·규칙과의 충돌 가능성을 제시하지만 충돌 소유자는 아직 특정하지 못했다.

기존 `ssh` 규칙의 Host Port만 `14242`로 바꾸고 Host IP `127.0.0.1`, Guest IP `10.0.2.15`, Guest Port `4242`는 유지하도록 안내했다. 두 설정 창을 OK로 저장한 뒤 호스트에서 `ssh -p 14242 jungblee@127.0.0.1`로 재시도하는 단계다. VM의 과제 SSH 포트 4242와 호스트 입구 포트는 서로 달라도 된다. 변경·접속 결과는 아직 없으며 복구 성공으로 처리하지 않는다.


## 2026-10-14 — 호스트 14242 전달과 SSH 지문 일치 확인

주 대화가 확인한 최신 전달 화면은 Host `127.0.0.1:14242` → Guest `10.0.2.15:4242`였다. 호스트의 `ssh -p 14242 jungblee@127.0.0.1`이 ED25519 fingerprint 질문까지 도달했고, 주 대화가 두 이미지를 대조해 앞서 VM 루프백 접속에서 표시된 지문과 일치함을 확인했다. 호스트 포트 변경으로 VM SSH 응답까지 도달했지만 원래 4242 충돌 주체는 아직 미확정이다.

사용자는 노트북에서는 4242 → 4242로 됐던 것 같다고 말했다. 해당 구성 자체도 유효하며 호스트별 포트 충돌 가능성이 달라질 수 있다고 설명했다. 현재 학교 실습의 14242는 실행 기록으로 두고 정상 치트시트에는 사용 가능한 호스트 포트를 선택하는 원칙만 보완했다.

`yes` 승인 후 비밀번호로 로그인하고 `hostname`·`whoami`를 각각 `jungblee42`·`jungblee`와 대조하도록 안내했다. 승인·로그인·명령 출력 결과는 아직 없다.


## 2026-10-14 — 호스트 일반 사용자 SSH 로그인 성공

주 대화가 확인한 새 화면에서 호스트의 `ssh -p 14242 jungblee@127.0.0.1`로 키 질문에 `yes`를 입력한 뒤 비밀번호 인증에 성공했다. `hostname`은 `jungblee42`, `whoami`는 `jungblee`였다. 일반 사용자 호스트 SSH 접속 체크포인트를 완료했다.

다음으로 `exit`로 호스트에 돌아온 뒤 `ssh -p 14242 -o NumberOfPasswordPrompts=1 root@127.0.0.1`을 실행하고 올바른 root 비밀번호로도 거절되는지 확인하도록 안내했다. 해당 옵션은 클라이언트 비밀번호 질문을 한 번으로 제한한다. root 접속 거절 결과는 아직 없다.


## 2026-10-14 — root SSH 거절 확인과 monitoring 첫 단위

주 대화가 확인한 최신 화면에서 호스트의 `ssh -p 14242 -o NumberOfPasswordPrompts=1 root@127.0.0.1`은 비밀번호 요청 한 번 후 `Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)`를 표시하고 호스트 프롬프트로 돌아왔다. 비밀번호 자체는 보지 않았다. 기존 `sshd -T`의 `permitrootlogin no`와 함께 root 직접 접속 거절 체크포인트를 기록했다.

사용자는 호스트 4242를 “42 pkg manager”가 차지한다는 이야기를 들었다고 말했다. 이는 충돌 관측과 일관되는 전달 정보지만 점유 프로세스를 직접 확인한 것은 아니다. 학교 프로그램은 그대로 두고 호스트 14242 사용을 유지하기로 안내했다.

monitoring 학습은 상태 수집 → 요구 형식 출력·wall → 부팅 시점·10분 주기 cron의 순서로 시작했다. 저장소 원문 v5.2 인쇄 10쪽의 Bash 스크립트·지표·방송 요구를 다시 대조했다. 첫 단위로 일반 사용자 SSH 재접속 뒤 `uname -a`·`lscpu`를 요청했고 OS·커널·아키텍처와 게스트의 CPU·소켓 수를 읽은 뒤 코드로 옮기기로 했다. 조회 결과는 아직 없으며 전체 스크립트를 작성하지 않았다.


## 2026-10-14 — 호스트 4242 점유 원인 확인으로 복귀

사용자는 14242 우회 성공으로 끝내려던 뜻이 아니라 4242 점유 원인과 그대로 사용할 해결 방법을 명확히 알고 싶다고 정정했다. monitoring 첫 조회는 잠시 미루고 호스트 진단을 이어 간다.

주 대화가 확인한 `ss -lntpe 'sport = :4242'` 화면은 `LISTEN 127.0.0.1:4242`, `Send-Q 5`, `ino:17925`, `sk:1`, `cgroup:/system.slice/supervisor.service`였다. 소켓이 supervisor 서비스 cgroup에 속한다는 근거이며 PID·프로세스명은 보이지 않았다. 42 패키지 관리자라는 실체는 아직 확인하지 못했다. 다음으로 `systemctl status supervisor --no-pager`를 요청했다.

같은 `127.0.0.1:4242`를 쓰려면 기존 점유 조정이 필요하고 학교 서비스라면 관리자와 조율해야 함을 설명했다. 현재 특정 IP 바인딩을 근거로 다른 루프백 `127.0.0.2:4242`는 기술적으로 공존할 가능성이 있으나 시험이 필요하다고 설명했다. 설정 변경을 안내하거나 실행한 것은 아니며 서비스 조회 결과도 아직 없다.


## 2026-10-14 — 다른 루프백의 4242 접속 성공과 과제 조건 대조

사용자는 `127.0.0.2:4242` → guest `10.0.2.15:4242` 접속에 성공했다고 확인했다. 이는 사용자 확인으로 기록하며 학교 서비스는 변경하지 않았다. 일반 사용자 로그인·hostname·whoami와 root 직접 접속 거절의 앞선 화면 검증은 14242 경로에서 수행한 결과다.

추가 전달 내용으로 supervisor 아래 `ftpkg-srv.py`와 다른 worker가 존재함을 확인했다. 해당 Python 파일 읽기는 `Permission denied`였으며, 정확한 4242 소켓 PID를 특정하지 못했다. 서비스 내부의 파일 존재만으로 그 파일이 해당 소켓 소유자라고 단정하지 않는다.

사용자가 두 전달 방식의 과제 적합성을 물어 저장소 원문 v5.2 인쇄 8쪽(PDF 9쪽)을 재확인했다. SSH와 방화벽의 4242 요구는 VM 내부 기준이며 호스트 전달 포트를 지정하지 않는다. 두 방식 모두 해당 포트 요구에 부합하고, 현재 성공한 다른 루프백의 4242를 유지하면 호스트에서도 4242를 쓰려는 목표를 충족한다고 설명했다.


## 2026-10-14 — monitoring 첫 출력 확인과 CPU 계산용 조회 안내

주 대화가 확인한 사용자 화면에서 `uname -a`의 커널 이름은 `Linux`, 호스트명은 `jungblee42`, 릴리스는 `6.12.0-211.16.1.el10_2.0.1.x86_64`, 아키텍처·OS 표기는 `x86_64`·`GNU/Linux`였다. lscpu는 Architecture `x86_64`, CPU(s) `2`, 온라인 `0,1`, Thread(s) per core `1`, Core(s) per socket `2`, Socket(s) `1`을 표시했다. 과제 지표의 근거로 게스트 소켓 1개·가상 CPU 2개를 읽으며 호스트 물리 하드웨어 개수로 해석하지 않는다.

Hypervisor vendor에는 `KVM`이 보였으나 확인된 VirtualBox 환경을 다른 하이퍼바이저 실행으로 단정하지 않았다. 다음은 `lscpu -p=SOCKET,CPU`에서 소켓 ID 중복과 CPU 행 수를 확인하는 단계다. 해당 출력은 아직 없으며 전체 스크립트는 작성하지 않았다.

## 2026-10-14 — 이전 비밀번호 비교 경로 설명 보완

공식 libpwquality pam_pwquality 매뉴얼의 `difok`·`enforce_for_root`를 대조했다. `difok=7`과 root 품질 강제는 유지하되, 일반 사용자가 sudo 없이 `passwd`를 실행하는 경로와 root 권한의 변경을 구분했다. `sudo passwd root`뿐 아니라 `sudo passwd jungblee`도 이전 암호 비교를 생략하므로 `enforce_for_root`가 이를 복원하지 않는다고 보완했다. 기존 정상 변경 성공과 별개로 실제 `difok` 위반 거절은 미검증이다.


## 2026-10-14 — CPU 계산용 두 행 확인과 첫 구현 예시 준비

주 대화가 확인한 사용자 `lscpu -p=SOCKET,CPU` 출력은 `#` 주석 뒤 `0,0`·`0,1` 두 행이었다. 같은 소켓 ID 0에 CPU ID 0·1이 속하므로 게스트 소켓 1개·온라인 CPU 2개라는 앞선 요약과 일치한다.

Architecture·CPU physical·vCPU 세 항목만 출력하는 Bash 예시를 치트시트에 추가하고 명령 치환·파이프·주석 제외·중복 제거·행 수·printf를 해당 흐름에 연결했다. 사용자 파일 작성·실행 결과는 아직 없다.

문서 예시를 임시 파일로 추출해 작업 환경에서 `bash -n`을 통과했고, 소켓 0·0과 CPU 0·1을 제공한 통제 입력에서 1·2 집계 및 세 줄 출력을 확인했다. 작업 환경의 실제 uname·lscpu를 이용한 읽기 전용 실행도 세 줄 출력과 오류 없음을 확인했다. 이 검사는 사용자 VM에서의 적용 검증이 아니며 작업 환경 수치를 VM 관측값으로 기록하지 않았다.


## 2026-10-14 — monitoring 첫 구현 검증과 메모리·디스크 원자료

주 대화가 전달한 사용자 VM의 `bash monitoring.sh` 실행 결과에서 Architecture는 앞서 조회한 원자료와 일치했고 CPU physical은 `1`, vCPU는 `2`였다. OS·CPU 세 항목의 첫 부분 구현을 적용·검증한 것으로 기록한다. 치트시트의 printf도 사용자 코드처럼 세 줄로 나누고 `#CPU physical :`·`#vCPU :` 표기를 맞췄다. 전체 monitoring 스크립트·wall·cron 검증을 완료했다는 뜻은 아니다.

사용자 VM에서 조회한 `free -m`의 값은 다음과 같다. 아래는 전달받은 관측값이며 이 작업 환경에서 VM 명령을 실행한 결과가 아니다.

| 행 | total | used | free | shared | buff/cache | available |
|---|---:|---:|---:|---:|---:|---:|
| Mem | 3909 | 459 | 3419 | 8 | 246 | 3449 |
| Swap | 4095 | 0 | — | — | — | — |

Swap의 free 값은 이번 전달에 없어 채우지 않았다. `free -m`의 단위는 MiB다.

`df -BM --total -x tmpfs -x devtmpfs`의 관측값도 표시 그대로 보존한다.

| 마운트·합계 | Size | Used | Avail | Use% |
|---|---:|---:|---:|---:|
| / | 9980M | 1087M | 8365M | 12% |
| /boot | 1946M | 169M | 1660M | 10% |
| /var/log | 1946M | 13M | 1816M | 1% |
| /home | 3952M | 1M | 3731M | 1% |
| total | 17823M | 1267M | 15571M | 8% |

행별 표시값과 total의 차이를 임의로 보정하지 않았다. 다음은 메모리 필드의 의미, 마운트 파일시스템 집계 범위와 단위·반올림을 읽는 단계다. 메모리·디스크 계산 코드는 아직 제안하거나 구현하지 않았다.

저장된 원문 v5.2 인쇄 10쪽을 다시 읽어 Memory Usage·Disk Usage 예시와 가용량·사용률 요구 문장을 목표 문서에 구분해 남겼다. 디스크 퍼센트의 used/size 또는 df Use% 선택, 집계 범위와 환산·반올림 방식은 원문에 명시되어 있지 않다. 원문 대조는 요구사항 확인이며 실제 VM 계산의 검증이 아니다.


## 2026-10-14 — 메모리·디스크 구현 검증과 CPU·부팅 시각 제안

주 대화가 전달한 사용자 VM의 `bash monitoring.sh` 실행 결과는 Architecture가 기존과 동일하고 CPU physical `1`, vCPU `2`였으며, 추가 출력은 다음과 같았다.

```text
#Memory Usage: 459/3909MB (11.74%)
#Disk Usage: 1267/17Gb (7%)
```

메모리·디스크 부분 구현의 적용·실행 검증을 완료한 것으로 기록한다. 두 지표의 사용률은 used/total로 통일했고 디스크 퍼센트는 `1267/17823 × 100`의 정수 표시다. `17`은 `17823/1024`의 반올림 표시값이며 비율 계산의 분모가 아니다. MB·Gb 라벨은 원문 예시를 따르지만 실제 단위는 MiB·GiB다. 기존 df의 `8%`와 이번 `7%`는 분모와 올림/반올림 방식이 다르므로 같은 값으로 고치지 않았다.

최신 procps-ng free 매뉴얼의 used = total − available 정의를 확인했다. 표시 정수만 뺀 값이 used와 1MiB 다르더라도 원자료 표시 정밀도 때문에 관측값을 수정하지 않는다. GNU df 매뉴얼의 pcent 요약과 src/df.c의 실제 분모·올림 처리를 구분해 목표 문서에 기록했다.

추가로 전달받은 `LC_ALL=C vmstat 1 2`의 두 번째 수치 행 CPU 필드는 `us 0`, `sy 0`, `id 100`, `wa 0`, `st 0`, `gu 0`이었다. `who -b`는 `system boot 2026-10-14 14:56`이었다. 이 값들은 사용자 VM 원자료이며 이 작업 환경의 실행값이 아니다.

주 대화에서 마지막 vmstat 행의 `100 - $15`와 who의 `$3`·`$4`를 출력하는 두 줄을 제안한 뒤, 사용자가 실행한 CPU load는 `0.0%`로 확인했다. CPU 부분 구현은 적용·실행 검증 완료다. 반면 Last boot는 `Oct 14`로 잘려 나왔다. C 로케일의 who 출력을 ISO 날짜의 고정 열로 가정한 코드가 잘못되었으며 이 항목은 정상 완료로 기록하지 않는다. `LC_ALL=C who -b`의 정확한 원출력은 아직 전달받지 않았다.

정상 절차는 `printf '#Last boot: %s\n' "$(uptime -s | cut -c 1-16)"`로 교체했다. 공식 procps-ng uptime 매뉴얼의 `-s` 형식이 `yyyy-mm-dd HH:MM:SS`임을 확인했고 앞 16자로 분까지 표시한다. 사용자 수정 실행 결과는 아직 없다. 작업 환경에서는 `uptime -s`의 형식만 읽기 전용으로 확인했으며 그 시각을 사용자 VM의 부팅 시각으로 취급하지 않았다. 나머지 지표·wall·cron 구현은 진행하지 않았다.

문서의 부분 코드를 임시 파일로 추출해 `bash -n`을 통과했다. 작업 환경의 통제 입력에서 메모리 `11.74%`, 디스크 `7%`, 마지막 CPU 표본 선택과 부팅 시각의 초 생략을 검사했다. 이는 예시 처리의 로컬 검사이며 사용자 VM에서 수정한 부팅 시각 코드의 실행 검증은 아니다.


## 2026-10-14 — 부팅 시각 형식 수정 실행과 원자료 차이 확인

주 대화가 전달한 사용자 VM의 최신 원출력은 `LC_ALL=C who -b`가 `system boot Oct 14 14:56`, `uptime -s | cut -c 1-16`이 `2026-10-14 14:54`였다. 앞선 날짜 표기 전달과 이번 C 로케일 원출력을 구분하며, 두 도구의 분 단위 시각이 2분 다름을 그대로 기록한다.

사용자가 수정 코드를 스크립트에 적용해 실행한 출력은 다음과 같다.

```text
#Memory Usage: 459/3909MB (11.74%)
#Disk Usage: 1267/17Gb (7%)
#CPU load: 0.0%
#Last boot: 2026-10-14 14:54
```

날짜 형식 수정과 스크립트 실행은 확인했고 Last boot는 전달받은 uptime 원자료의 분 단위 값과 일치했다. who의 `14:56`과 일치한 것은 아니다. 두 도구가 같은 시각을 낼 것이라는 앞선 설명은 철회한다. 차이의 원인은 아직 확인하지 못했으며 공식 소스 조사는 별도 진행 중이다. 원인을 추측해 확정하거나 새 코드로 교체하지 않았다. 사용자 VM 관측과 작업 환경의 예시 검사를 구분하며 VM 조작은 하지 않았다.


## 2026-10-14 — 부팅 시각 출처 구분과 LVM·TCP 조회 안내

공식 소스 조사에서 procps-ng v4.0.4의 uptime-s는 현재 시각에서 가동시간을 빼는 계산임을 확인했고, who-b는 BOOT_TIME 기록을 읽는 경로로 구분했다. systemd v257의 부팅 기록 작성 흐름도 대조했다. 이 설명을 사용자에게 전달했으나 사용자 VM의 2분 차이에 대한 정확한 원인을 입증한 것은 아니다. `who`의 `14:56`과 `uptime` 및 스크립트의 `14:54` 관측을 모두 보존한다. LUKS 지연 등 특정 원인으로 확정하지 않는다.

`uptime -s | cut -c 1-16`을 유지하고 날짜 형식 수정·`14:54` 사용자 실행 검증은 완료로 둔다. 시각 차이의 추가 추적은 현재 과제 진행에서 미루기로 했다.

다음 원자료로 `lsblk -o NAME,TYPE`과 `ss -Htn state established`를 안내했다. TYPE의 lvm 존재 여부와 현재 ESTABLISHED TCP 소켓 행 수를 확인할 예정이며, 아직 사용자 조회 결과는 없다. 치트시트에 정상 조회 흐름, 목표 문서에 공식 읽기 범위를 추가했다. LVM·TCP 출력 코드는 아직 작성하지 않았고 사용자 VM 조작도 하지 않았다.


## 2026-10-14 — VM LVM·TCP 원자료 확인과 부분 코드 제안

사용자 VM `jungblee42`의 `lsblk -o NAME,TYPE`에서 sda3 아래 crypt, 그 아래 LVM-root·swap·var_log·home 모두 TYPE `lvm`을 확인했다. LVM 사용 yes의 원자료 근거다. `ss -Htn state established`는 한 행이며 local `10.0.2.15:4242`, peer `10.0.2.2:41682`였다. 현재 관측된 ESTABLISHED TCP 소켓은 1개다.

직전 학교 호스트 첨부는 VM 결과가 아니므로 이번 판단의 근거로 사용하지 않았다. 호스트의 상세 연결정보는 저장하지 않았다.

TYPE의 lvm 존재 여부를 if와 grep으로 판단하는 부분 코드, 필터된 ss 소켓 행을 wc로 세는 부분 코드를 사용자에게 제안했다. 사용자 스크립트 적용·실행 결과는 아직 없다. 원자료 확인을 코드 실행 완료로 간주하지 않는다. 나머지 지표 구현이나 사용자 VM 조작은 하지 않았다.

작업 환경에서 문서 예시의 Bash 구문과 통제 입력 네 경우(LVM 존재/부재 × TCP 0/1행)를 검사해 통과했다. 부재 입력에는 `notlvm`도 넣어 행 전체 일치 조건을 확인했다. 이 검사는 사용자 VM 실행 검증이 아니다.


## 2026-10-14 — LVM·TCP 실행 검증과 로그인·네트워크 조회 안내

사용자 VM의 스크립트 실행에서 `#LVM use: yes`와 `#Connections TCP : 1 ESTABLISHED`를 확인했다. LVM·TCP 부분 코드의 적용·실행 검증을 완료로 기록한다. 기존 일곱 항목도 출력되었고 CPU load는 이번 시점에 `1.0%`, Last boot는 `2026-10-14 14:54`였다. CPU 수치 변화는 조회 시점이 다른 관측으로 보존하며 이전 `0.0%`를 수정하지 않는다.

다음 원자료로 VM에서 `who`, `ip -br address`, `ip -br link`를 조회하도록 안내했다. 로그인 세션과 사용자명 중복, enp0s3의 IPv4·MAC을 확인할 예정이며 아직 결과는 없다. 해당 집계·출력 코드는 작성하지 않았다. 남은 수집 항목은 사용자 수·IPv4/MAC·sudo 실행 횟수이며 이후 방송·예약을 다룬다. 사용자 VM 조작은 하지 않았다.


## 2026-10-14 — 사용자 수·IPv4·MAC 적용 검증과 sudo 로그 조회 안내

사용자 VM의 who 원자료에서 `jungblee`가 `tty1`, `pts/0`의 두 세션으로 나타났다. 사용자명 중복을 제거한 로그인 사용자 수는 1이다. enp0s3의 IPv4는 `10.0.2.15/24`, MAC은 `08:00:27:32:c3:70`으로 확인했다.

사용자가 부분 코드를 적용해 실행한 결과는 다음과 같았으며 이 두 출력의 적용·실행 검증을 완료로 기록한다.

```text
#User log: 1
#Network: IP 10.0.2.15 (08:00:27:32:c3:70)
```

원문의 사용자 수 요구와 대조해 중복 제거한 로그인 사용자명 수를 선택한 해석을 목표 문서에 명시했다. 원문에는 세션 수를 계산하라는 지정이 없다.

남은 수집 항목인 sudo 횟수의 실제 로그 형식을 보기 위해 `sudo journalctl _COMM=sudo -n 10 --no-pager -o cat`를 요청했다. 아직 결과가 없어 sudo 집계 코드는 작성하지 않았고 방송·cron도 구현하지 않았다. 사용자 VM 조작은 하지 않았다.

작업 환경의 통제 입력으로 동일 사용자 두 세션의 중복 제거, IPv4 접두사 제거와 MAC 출력 조합을 검사했고 Bash 구문 검사도 통과했다. 사용자 VM 원자료와 로컬 검사 입력은 구분했다.


## 2026-10-14 — sudo journal 구조 확인과 I/O 세션 목록 조회 안내

사용자 VM의 최근 sudo journal 출력에서 `TTY`·`PWD`·`USER`·`TSID`·`COMMAND`를 포함한 명령 기록과 `pam_unix`의 session opened/closed 메시지가 함께 나타났다. 조회 자체도 TSID `00001E`·`00001F`로 기록되었다. 상세 명령 인자는 복사하지 않고 메시지 구조만 보존한다.

journal 전체 줄 수를 실행 횟수로 쓸 수 없고 COMMAND 단순 검색도 거절된 시도를 포함할 수 있음을 설명했다. 기존 log_input/log_output 설정과 `/var/log/sudo`를 재사용해 보관된 I/O 실행 세션 목록을 집계하는 방향을 안내했다. 영구 누적 횟수나 sudo 셸 내부 명령 수를 의미하는 것은 아니다.

다음 검증으로 `sudo sudoreplay -d /var/log/sudo -l | tail -n 5`를 요청했으며 아직 출력은 없다. 공식 sudoreplay·sudoers 매뉴얼에서 목록 모드·로그 필드·보관 범위를 대조하고 정상 조회 흐름을 문서에 추가했다. sudo 집계 코드·방송·cron은 구현하지 않았고 사용자 VM 조작도 하지 않았다.


## 2026-10-14 — I/O 세션 목록 확인과 sudo 집계 코드 제안

사용자 VM의 `sudo sudoreplay -d /var/log/sudo -l | tail -n 5` 결과에서 각 세션이 한 줄로 표시됨을 확인했다. 마지막 TSID `00001G`는 조회 자체의 sudoreplay 실행이었다. 상세 명령 인자는 복사하지 않고 세션 행 구조와 자기 조회 기록만 보존한다.

sudoreplay 전체 목록을 wc로 세어 `#Sudo : … cmd`로 출력하는 두 줄을 제안했다. 수동 검증은 `sudo bash monitoring.sh`로 안내했고 스크립트 내부에는 sudo를 넣지 않았다. 내부 조회가 별도 sudo 세션을 만들지 않지만 수동 실행 자체는 집계에 포함될 수 있다. 현재 보관된 I/O 실행 세션 수라는 정의를 유지한다.

사용자 스크립트 적용·실행 결과는 아직 없다. 방송·cron은 아직 안내하지 않았으며 설정이나 코드를 추가하지 않았다. 사용자 VM 조작도 하지 않았다.

작업 환경에서 문서 예시의 Bash 구문과 통제 입력 0세션·3세션의 집계 출력을 검사해 통과했다. 실제 sudo 로그를 읽거나 사용자 VM에서 실행한 검사는 아니다.


## 2026-10-14 — 수집 지표 12개 실행 확인과 전체 소스 요청

사용자가 VM에서 `sudo bash monitoring.sh`를 실행해 12개 항목이 모두 출력되고 권한 오류가 없음을 확인했다. 최신 관측값은 다음과 같다.

| 항목 | 전달받은 실행 결과 |
|---|---|
| Architecture | 기존과 동일 |
| CPU physical / vCPU | 기존 1 / 2 유지 |
| Memory Usage | 461/3909MB (11.79%) |
| Disk Usage | 1268/17Gb (7%) |
| CPU load | 0.0% |
| Last boot | 2026-10-14 14:54 |
| LVM use | yes |
| Connections TCP | 1 ESTABLISHED |
| User log | 1 |
| Network | IP 10.0.2.15 (08:00:27:32:c3:70) |
| Sudo | 53 cmd |

수집 코드의 사용자 적용·실행 확인을 완료로 기록한다. Sudo 53은 현재 보관된 I/O 세션 목록을 집계한 스크립트 출력이며, 53개 세션을 독립적으로 전수 대조했다는 뜻은 아니다. 다른 변동 수치도 이번 실행 시점의 관측으로 보존한다. 출력 성공만으로 전체 소스 검토·오류 처리·평가 요구 전체 검증을 완료 처리하지 않는다.

사용자에게 `cat monitoring.sh` 전체 소스를 요청했으며 아직 전달받지 않았다. 전체 검토 후 wall 단일 방송, root 소유 경로 설치, cron 순서로 진행할 예정이지만 해당 단계의 구체 명령이나 변경을 아직 제안·적용하지 않았다. 이번 동기화에서도 방송·설치·예약 구현과 사용자 VM 조작은 하지 않았다.


## 2026-10-14 — 전체 소스 확인과 wall 단일 방송 제안

사용자가 `cat monitoring.sh`로 전체 소스를 제공했고 주 대화에서 지금까지 안내한 12개 수집 항목 코드와 일치함을 확인했다. 앞선 전체 소스 대기 상태를 해소했다. 이 확인은 모든 오류 상황이나 예약 실행까지 검증했다는 뜻은 아니다.

shebang 다음 줄에 `{`를 열고 기존 코드를 모두 안에 둔 뒤 마지막 새 줄에 `} | wall`을 추가하도록 제안했다. 전체 표준출력을 wall 한 번에 전달하는 구조다. `bash -n monitoring.sh` 후 `sudo bash monitoring.sh`를 실행해 tty1·SSH 양쪽에서 12개 항목의 한 방송을 확인하도록 요청했다. 사용자 적용·수신 결과는 아직 없다. root 소유 설치 경로와 cron은 아직 안내·적용하지 않았다.

작업 환경에서 같은 그룹·파이프 구조에 12개 통제 출력 항목을 넣어 Bash 구문을 검사했다. 가짜 wall 함수가 한 번 호출되어 12개 항목을 순서대로 수신함을 확인했다. 실제 wall이나 사용자 VM 방송을 실행한 검사가 아니며, 사용자 전체 스크립트의 실방송 검증을 대신하지 않는다.


## 2026-10-14 — wall 방송 출력 확인과 중복 표시 확인 대기

사용자 VM에서 `bash -n monitoring.sh`가 무출력으로 성공했고, `sudo bash monitoring.sh` 실행의 wall 방송에 12개 항목이 표시됨을 확인했다. 최신 전달 값에는 Sudo `54 cmd`, CPU load `1.0%`, Memory Usage `460/3909MB (11.77%)`가 포함된다. 수치는 이번 실행 시점의 관측이며 Sudo의 독립 전수검증을 뜻하지 않는다.

붙여 준 내용에는 같은 타임스탬프 `18:28:36`의 방송 두 개가 있었다. 두 터미널의 결과를 함께 붙인 것인지, 한 터미널에서 중복 표시된 것인지 사용자에게 질문했으며 아직 답변이 없다. 따라서 방송 출력 성공은 확인했지만 tty1·SSH 양쪽 수신 완료나 단일 터미널 중복 발생을 확정하지 않는다.

sudo I/O 로깅의 PTY 관련 가능성은 조사 중이며 원인으로 확정하지 않았다. root 소유 설치 경로와 cron은 아직 적용하지 않았고 이번 작업에서도 사용자 VM을 조작하지 않았다.


## 2026-10-14 — 같은 터미널의 방송 중복 확인과 PTY 조회 대기

사용자가 앞선 두 방송을 같은 터미널에서 두 번 수신했다고 확인했다. 두 터미널 결과를 합쳐 붙인 경우가 아니며, tty1·SSH 양쪽 수신 완료 여부는 여전히 별도 확인 대상이다.

원인을 구분하기 위해 `tty`와 `sudo sh -c 'tty; who'` 조회를 요청했으며 아직 결과는 없다. 주 대화에서 공식 wall 소스와 sudoers의 set_utmp·I/O logging 설명을 근거로, sudo I/O 로깅 PTY와 원래 SSH PTY가 각각 방송을 받은 뒤 전달되어 중복 표시될 가능성을 제시했다. 이는 조사 가설이며 실제 PTY·로그인 기록과 대조하기 전에는 원인으로 확정하지 않는다.

로깅이나 requiretty를 비활성화하지 않고, 최종 root cron에서는 스크립트를 sudo 없이 실행하는 방향을 설명했다. 설정을 변경하거나 cron을 적용한 것은 아니다. 일회 진단 명령은 이 기록에만 두고 정상 치트시트에는 추가하지 않았다.


## 2026-10-14 — sudo PTY·로그인 기록 확인과 설치 단계 제안

사용자의 일반 셸 `tty`는 `/dev/pts/0`, sudo sh 안의 `tty`는 `/dev/pts/1`이었다. sudo 안의 who에는 기존 tty1·pts/0과 추가 pts/1이 모두 jungblee로 표시되었다. sudo 실행의 추가 PTY와 로그인 기록은 확인했다. 이는 원래 터미널의 직접 수신과 sudo PTY에서 전달된 수신이 겹친다는 설명에 부합하지만 wall의 실제 write 대상을 추적한 것은 아니므로 전달 경로까지 확정하지 않는다.

로깅·requiretty 등 보안 설정을 유지하고 최종 root cron에서 sudo 없이 실행해 단일 수신 여부를 확인할 예정이다. root cron의 실제 방송은 아직 검증하지 않았다.

다음으로 `sudo install -o root -g root -m 755 monitoring.sh /usr/local/sbin/monitoring.sh`, 설치본 `ls -l`, `systemctl status crond --no-pager`를 제안했다. 복사와 동시에 소유권·권한을 지정하고 root 예약 실행 파일의 일반 사용자 수정을 막는 이유를 설명했다. 아직 사용자 실행 결과는 없으며 cron 등록도 제안하지 않았다. 이번 작업은 문서 반영이며 사용자 VM의 파일 설치나 서비스 변경을 수행하지 않았다.


## 2026-10-14 — root 소유 설치 확인과 crontab 제안

사용자 VM의 `/usr/local/sbin/monitoring.sh`는 `-rwxr-xr-x.`, 소유자·그룹 `root root`, 크기 1218바이트로 확인되었다. crond는 `active (running)`이고 `enabled`였다. 설치본 소유권·권한과 데몬 상태 확인을 완료했으며 이것을 예약 등록이나 실행 확인으로 간주하지 않는다.

기존 내용을 보존하고 중복을 피하며 root crontab에 PATH, 매시 10분 경계 실행, 별도의 @reboot 실행을 등록하도록 제안했다. root 사용자 열과 sudo를 넣지 않고 crond 재시작도 필요 없음을 설명했다. `sudo crontab -l` 확인을 요청했다.

안내 당시 전달된 시각은 18:35 KST였고 18:40 방송에서 SSH·콘솔 각각 한 번씩 수신되는지 확인하도록 요청했다. 아직 등록 내용이나 예약 방송 결과는 없다. 부팅 실행도 미검증이다. 이번 작업에서는 문서만 반영했으며 VM 설치·서비스 변경·cron 등록을 직접 수행하지 않았다.

Cronie의 run_reboot_jobs와 REBOOT_LOCK 처리를 대조해 일반 서비스 재시작을 부팅 테스트로 삼지 않도록 목표 문서를 보완했다. 부팅 실행 로그와 로그인 터미널 수신을 구분했다. 두 monitoring 예약 줄만 주석 처리하는 중단 방향은 문서상 검토 내용이며 사용자 안내·적용은 아직 없다.


## 2026-10-14 — root 예약 등록과 첫 자동 방송 확인

사용자 VM의 `sudo crontab -l`에서 PATH와 `*/10`, `@reboot` 두 항목이 제안한 내용과 정확히 일치함을 확인했다. 18:40:02 KST에 12개 항목의 자동 wall 방송이 해당 터미널에 한 번 표시되었고, 발신 위치는 `somewhere`, Sudo 출력은 `60 cmd`였다. root 예약 등록·첫 주기 실행·해당 터미널의 수신 확인을 완료로 기록한다. SSH·콘솔 양쪽의 동시 수신을 확인한 것은 아니다. Sudo 60도 현재 I/O 세션 집계 출력이며 독립 전수검증이 아니다.

다음으로 저장 후 `sudo reboot`, VM 콘솔의 LUKS 해제·로그인, 호스트의 `ssh -p 4242 jungblee@127.0.0.2` 재접속을 안내했다. VM에서 sshd·firewalld·crond의 is-active, getenforce, 이번 부팅의 crond journal을 조회하도록 요청했다. 아직 결과가 없으며 @reboot 실행·재부팅 후 서비스·재접속은 미검증이다. 부팅 직후 로그인 사용자가 없으면 wall 화면이 없을 수 있어 실행 로그와 수신 화면을 구분한다.

이번 문서 작업에서 기존 체크아웃과 직전 커밋을 정상적으로 읽었고 작업 트리는 깨끗했다. 저장 환경 복구 작업은 필요하지 않았다. 사용자 VM의 재부팅이나 설정 변경을 직접 수행하지 않았다.


## 2026-10-14 — 재부팅 후 서비스 유지 확인과 부팅 작업 로그 추가 조회

사용자 VM 재부팅 후 sshd·firewalld·crond는 모두 `active`, SELinux는 `Enforcing`으로 확인했다. 서비스 실행 상태와 SELinux 모드 유지 검증을 완료로 기록한다.

`journalctl -b -u crond`에는 18:41:57의 STARTUP(Cronie 1.7.0)과 기본 INFO 메시지만 보였고 monitoring 작업 실행 기록은 보이지 않았다. 이것만으로 @reboot 실행 완료 또는 실행 실패를 확정하지 않는다. 서비스 시작과 monitoring 작업 실행은 별도 증거다.

작업 로그의 journal 식별자가 다를 가능성도 있어 `sudo crontab -l`과 `sudo journalctl -b -t CROND --no-pager`를 추가 요청했다. 아직 결과는 없다. @reboot 실행은 미검증으로 유지하며 이번 문서 작업에서 VM 설정이나 예약을 변경하지 않았다.


## 2026-10-14 — @reboot 시작·종료 기록 확인과 중단 테스트 제안

재부팅 후 root crontab은 기존 PATH·주기·@reboot 항목을 유지했다. `journalctl -b -t CROND`에서 18:41:58 root의 `CMD (/bin/bash /usr/local/sbin/monitoring.sh)`, 18:41:59 CMDEND를 확인했다. 부팅 시각과 별도 @reboot 예약을 대조해 부팅 작업의 시작·종료 기록 확인을 완료로 기록한다. 이 로그만으로 개별 명령의 오류 부재나 방송 수신까지 완료 처리하지 않는다. 이 환경에서 작업 기록 조회에 CROND 식별자가 유효함을 정상 절차에 반영했다.

다음으로 root crontab의 두 monitoring 예약 줄만 #으로 주석 처리하고 PATH·다른 예약은 유지하도록 제안했다. 목록을 확인하고 18:50 KST 방송이 없는지 관찰한 뒤 주석을 제거해 복구할 예정이다. 아직 주석 적용 증거는 없으며 중단·복구도 미검증이다. crond 전체 중단이나 스크립트 수정은 제안하지 않았다. 이번 작업에서도 사용자 VM 변경을 직접 수행하지 않았다.


## 2026-10-14 — 문서 전체 검토와 재현 경로 정리

과제 v5.2의 필수 조건·README·제출 장과 학습 기록을 대조했다. README의 오래된 미검증 문구와 긴 진행 이력을 줄이고, 이미 문서화한 설계 근거를 링크했다. 미작성 비교와 미완료 검증은 유지했다.

치트시트에 흩어진 monitoring 코드 조각을 기존 명령 그대로 34절 한 블록에 모았다. 항목별 조회·대조 순서와 읽기 링크는 유지했다. SSH 예시 주소를 한 경로로 맞추고 실제 포워딩 값이 다르면 모든 접속 명령에 함께 반영하도록 명시했다. root 암호의 비교 예외·추가 비밀번호 계정의 갱신 범위를 보완했다. targets의 중복·오래된 단계 안내를 정리했다.

기존 가이드의 mandatory·README·서명·스냅샷 조건은 원문과 일치해 그대로 두었다. 빈 체크박스는 완료 현황이 아닌 요구사항 점검용이다. 이번 변경은 문서 구조·설명 수정이며 monitoring 기능·VM 설정·실제 검증 상태를 바꾸지 않았다.

최종 검사에서 통합 코드가 기존 조각을 순서대로 묶은 코드와 동일함을 확인했다. Bash 구문과 통제 입력의 12항목·가짜 wall 1회 수신, 네 문서의 로컬 파일·제목 앵커 링크, 코드 블록, README 첫 줄을 검사해 통과했다. 실제 VM 방송이나 미완료 검증을 다시 수행한 것은 아니다.


## 2026-10-14 — 18:50 정상 방송과 예약 편집 내용 확인 대기

사용자 화면에서 18:50:01 자동 방송의 12개 항목을 확인했다. Last boot는 `18:41`로 재부팅을 반영했고 Sudo는 `64 cmd`였다. 그 뒤 `sudo crontab -e`의 `installing new crontab` 메시지가 나타났지만 편집한 내용은 제공되지 않았다. 따라서 18:50 방송은 편집 전 정상 실행이며 중단 테스트 실패의 증거가 아니다.

현재 내용을 확인하기 위해 `sudo crontab -l`을 다시 요청했다. 두 예약 줄의 주석 적용·중단·복구는 모두 아직 미검증이다. 저장 메시지만으로 어떤 예약이 바뀌었는지 단정하지 않는다.

별도로 사용자가 Host IP `127.42.42.42:4242` 사용 가능성을 물었다. 주 대화에서는 Linux의 127/8 루프백 범위라는 점을 근거로, 사용 가능한 경우 Host IP만 변경하고 guest 4242를 유지하는 방안을 조건부로 안내했다. 변경 적용이나 접속 성공은 아직 확인되지 않았다. 이는 검토안이며 치트시트의 기준 접속 경로 `127.0.0.2:4242`는 유지한다. 이번 작업에서 사용자 VM·호스트 설정은 변경하지 않았다.
