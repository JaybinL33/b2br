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

사용자가 반복 실습에서 재배치와 축소 가능성을 우선해 ext4를 선택했다. 이에 따라 이전 XFS·root 15 GiB·home 잔여 전부 배정안은 아래 안으로 변경했다. 새 30 GB VDI에 적용할 최신 기준은 이 표다. **실습용 설계 선택이며 과제의 고정 용량 조건이나 적용 완료 기록이 아니다.**

| 대상 | 선택 | 이유 |
|---|---|---|
| `/boot` | 2 GiB, Standard Partition, ext4, 비암호화 | initramfs와 커널 업데이트 여유를 둔다. LVM 밖의 일반 파티션이므로 이후 크기 조정이 더 까다롭다. |
| 나머지 디스크 | LUKS 암호화 장치 위에 PV와 VG `LVM` 구성 | root·swap·home을 공통 암호화 계층 안에 둔다. |
| LV `root` | 10 GiB, ext4, `/`에 마운트 | 실측 최소값이 아닌 초기 배정이다. 설치 후 사용량과 업데이트 여유를 확인한다. |
| LV `swap` | 4 GiB, swap | RAM 4 GiB에 대한 RHEL 일반 권장표를 참고했다. 최대절전용 크기로 정한 것은 아니다. |
| LV `home` | 2 GiB, ext4, `/home`에 마운트 | 작은 초기 공간으로 시작하고 필요에 따라 늘린다. |
| VG 잔여 공간 | LV에 배정하지 않고 유지 | 이후 root·home 확장이나 새 LV 실습에 사용한다. 디스크 바깥의 미분할 공간이 아니라 VG 안의 여유 공간이다. |
| BIOS Boot (`biosboot`) | GPT+BIOS일 때 1 MiB, 일반 파티션, 파일시스템·마운트·암호화 없음 | 부트로더용 영역이며 `/boot`나 EFI 시스템 파티션과 다르다. 설치기에서는 BIOS Boot 유형을 선택한다. |

유일한 VG는 LVM 관리 영역임을 알아보기 쉽도록 대문자 `LVM`으로 이름 짓기로 했다. LV 이름은 `root`, `home`, `swap`이다. LVM은 관리 기술의 이름이고, 여기서 VG 이름으로도 사용한다. VG는 PV의 공간을 묶어 LV에 할당한다. 실제 VM에는 아직 적용하지 않았다.

학습자료로 설명한 관계는 `파티션 → LUKS → 열린 암호화 장치 → LVM PV → VG LVM → 각 LV`다. root와 home의 LV에는 ext4 파일시스템을 만들고 각각 `/`와 `/home`에 연결한다. swap에는 디렉터리 마운트 지점을 두지 않는다. 원문 인쇄 7쪽도 암호화 장치 안에 여러 LVM 볼륨을 두는 예시이므로 이 계층을 기준으로 삼았다. 사용자가 이 구조를 이해하거나 실제로 구성했는지는 아직 확인하지 않았다.

설치기에서는 `Volume Group` 옆 `Modify`로 연 **VG 설정 창의 `Encrypt`**를 사용하고, `Size policy`는 `As large as possible`로 선택하도록 안내했다. root·swap·home은 같은 VG에 두고 개별 LV에 추가 암호화는 설정하지 않는 계획이다. VG는 가능한 크게 확보하되 LV는 각각 10·4·2 GiB만 배정한다. home 용량을 비워 잔여 전부를 배정하지 않는다. 마운트 지점 편집 영역의 개별 `Encrypt`와 혼동하지 않도록, **`Begin Installation` 전에 파티션 목록과 VG 설정 화면을 요청했다.** 실제 화면과 적용 상태는 아직 미확인이다.

근거는 [과제 원문](../en.subject.pdf), [RHEL 10 설치기 문서의 10.5절](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/interactively_installing_rhel_from_installation_media/customizing-the-system-in-the-installer), [RHEL 10 LUKS 안내](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/encrypting-block-devices-by-using-luks)다. `/boot` 2 GiB는 RHEL의 베어메탈 권장을 참고해 실습 여유로 선택했으며 VM의 필수값은 아니다. LUKS2가 RHEL 기본값이라는 설명도 실제 생성 결과와 구분한다. [Rocky 10 설치 안내](https://docs.rockylinux.org/guides/installation/)는 자동 파티션을 설명하므로 Custom의 세부 조작은 RHEL 문서를 참고했다. 사용 중인 Rocky 설치기의 정확한 UI와 암호화 계층은 화면과 설치 결과로 확인할 예정이다.

ext4 축소 조건은 학습자료로 안내했다. 마운트를 해제하고 파일시스템을 먼저 줄인 뒤 LV를 줄여야 하며, 실행 중인 시스템의 root는 이 조건을 충족할 수 없어 별도 부팅 환경이 필요하다. 이는 사용자의 이해나 실행을 확인한 기록이 아니다. 근거는 [RHEL 10 ext4 크기 조정](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_file_systems/getting-started-with-an-ext4-file-system)의 32.4절과 [LV 축소](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_logical_volumes/basic-logical-volume-management)의 4.3.5절이다. 실제 축소 명령이나 작업은 수행하지 않았다.

## 2026-10-12 — 설치 전 진행 상태

**사용자 보고:** Kdump를 끄고 hostname을 `jungblee42`로 설정했다고 알렸다. hostname 값은 설치 요약 화면에 나오지 않아 사용자 보고로만 남긴다.

**대화에서 화면으로 확인한 내용:** 설치 요약에 Rocky Linux Minimal 10.2 설치기, Minimal Install, Asia/Seoul, Kdump disabled, enp0s3 connected가 표시되었다. 당시 Automatic partitioning이 선택되어 있었고 root 계정은 비활성, 생성 예정 사용자는 없었으며 `Begin Installation` 버튼도 비활성이었다. 설치기의 버전 표시는 설치된 OS나 부팅 결과를 검증한 것이 아니다.

다음 단계로 `Installation Destination → 새 30 GB 디스크 → Custom → Done`을 안내했고 파티션 구성 뒤 계정을 설정할 예정이다. 최신 ext4 구성의 실제 적용·부팅·이해 완료는 아직 확인하지 않았다. 이번 문서 작업에서는 VM·호스트 설정을 변경하거나 서명을 생성하지 않았다.
