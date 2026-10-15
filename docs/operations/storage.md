# 부팅·저장장치·파일시스템 운영

[전체 안내서](../targets.md) · [계정·접속](access.md) · [서비스·업데이트·백업](maintenance.md)

설치 용량과 명령을 다시 만드는 절차는 [디스크 설계](../targets.md#1-설치-결과와-디스크-확인)와 [스피드런](../speedrun.md)에 있다. 여기서는 부팅이 멈추거나 공간이 부족할 때 어느 계층을 살필지 다룬다. 아래 사례는 운영 판단 예시이며 장애를 실제로 재현했다는 기록이 아니다.

## 전원을 켠 뒤 셸까지

VirtualBox는 CPU·메모리·디스크·NIC를 게스트에 제공한다. 호스트의 VDI 파일은 게스트에서 블록 장치로 보인다. 이 구성의 BIOS 부팅에서는 GRUB이 `/boot`의 커널과 initramfs를 읽는다. initramfs는 실제 root를 쓰기 전에 필요한 드라이버·암호화 해제·LVM 활성화 등을 준비하는 초기 실행 환경이다. LUKS를 열고 root LV를 찾으면 실제 root로 전환하고 systemd가 마운트·서비스·로그인 환경을 구성한다. 의존성이 있는 작업은 순서를 지키지만 모든 서비스가 하나씩 직렬로 시작하는 것은 아니다.

따라서 LUKS 암호 입력 화면에 멈춘 동안 SSH가 안 되는 것은 아직 sshd까지 도달하지 않은 상태일 수 있다. 암호화 해제 암호는 root 로그인 암호와 별개다. `/boot`가 비암호화인 선택은 초기 부팅 자료를 읽기 위한 구성이고, 사용자 데이터의 암호화 범위와 구분한다. LUKS는 잠긴 디스크의 내용을 보호하며 열린 시스템에서 사용자의 파일 접근 권한을 대신하지 않는다.

부팅 때 사용할 암호화 장치 식별은 `/etc/crypttab`과 initramfs에 포함된 정보·커널 인수 등과 연결된다. LVM은 장치의 메타데이터를 읽어 PV/VG/LV를 활성화하고, fstab은 그 위 파일시스템의 마운트 위치를 연결한다. 따라서 crypttab이나 root 장치 구성을 바꿀 때는 실제 부팅 경로에서 initramfs 재생성이 필요한지도 따로 판단한다. 파일 한 줄 수정이 초기 부팅 이미지까지 자동으로 갱신한다는 뜻은 아니다.

**어디서 멈췄는가:** VirtualBox 자체 오류라면 호스트 저장 공간·연결된 디스크·VM 로그를 먼저 본다. GRUB 이전이면 부팅 매체와 펌웨어 설정, 암호 입력 이후 root를 못 찾으면 initramfs·장치 식별·LVM, emergency 모드라면 실패한 마운트와 부팅 journal을 살핀다. 정상 부팅했는데 SSH만 안 되면 [접속 경로](access.md#ssh가-안-될-때-경계를-하나씩-통과하기)로 이동한다.

게스트 콘솔에서 로그인할 수 있을 때의 조회:

```bash
systemctl --failed
sudo journalctl -b -p err --no-pager
sudo journalctl -b -k --no-pager
findmnt /
```

오류 우선순위만으로 원인을 놓칠 수 있어 실패한 unit을 찾으면 그 unit의 전체 부팅 로그를 이어서 읽는다. 이전 부팅 로그는 보존되어 있을 때만 `journalctl -b -1`로 조회할 수 있다. initramfs 복구 셸에서는 위 명령이 모두 제공된다고 가정하지 않는다. 콘솔 메시지와 마지막 변경을 먼저 기록한다.

부팅 실패를 고치려고 같은 디스크에 다시 설치하거나 암호화 포맷을 실행하면 복구할 데이터를 덮을 수 있다. 복구 매체가 필요한 경우 **VM 종료·디스크 보관본 확보·대상 장치 식별 후** ISO의 rescue 환경에서 조사한다. LUKS 열기, VG 활성화, 파일시스템 마운트는 별도 단계다. 읽기 전용 조사로 시작하되 파일시스템 종류에 따라 읽기 전용 마운트도 journal 재생을 할 수 있으므로 원본 보존은 먼저 한다. 모든 장애에 공통인 `fsck -y`나 부트로더 재설치 명령을 기본 절차로 두지 않는다.

## 저장 공간은 네 곳에서 따로 계산한다

```text
호스트 파일시스템의 여유
  └─ VDI 파일 / 게스트 디스크의 최대 용량
       ├─ BIOS Boot, /boot
       └─ LUKS 컨테이너 → 열린 장치 → PV → VG LVM의 여유
                                                └─ LV → ext4의 여유 → 파일
                                                └─ swap
```

PV는 LVM이 관리할 저장 영역, VG는 PV의 공간을 모은 풀, LV는 그 풀에서 떼어 준 블록 장치다. 파일시스템은 LV 안에서 파일명·디렉터리·inode·데이터를 관리한다. swap LV는 파일시스템으로 마운트하는 영역이 아니다. 이 구성은 LUKS 안에 LVM을 두며 LV마다 암호화 계층을 중복하는 설계가 아니다. [RHEL LVM 개요](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_logical_volumes/overview-of-logical-volume-management)의 `LVM architecture`와 [LUKS](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/encrypting-block-devices-by-using-luks)의 개요를 연결한다.

| 증상 | 좁힐 범위 | 확인과 다음 판단 |
|---|---|---|
| 호스트가 VDI에 쓰지 못함 | 호스트 파일시스템 | 게스트 `df`가 여유 있어도 호스트는 가득 찰 수 있다. 호스트에서 VM 폴더가 있는 파일시스템을 확인한다. |
| `/var/log`만 가득 참 | var_log 파일시스템 | root·home의 빈 공간은 자동으로 공유되지 않는다. 로그 증가 원인과 보존 정책을 찾는다. |
| VG는 여유 있는데 `df`는 가득 참 | VG → LV → 파일시스템 | 미할당 VG 공간은 아직 파일시스템에 주어진 공간이 아니다. 확장 작업이 필요할 수 있다. |
| 바이트 여유는 있는데 새 파일 생성 실패 | inode·권한·읽기 전용 마운트 | 작은 파일 과다, 접근 거절, 오류로 인한 읽기 전용 전환을 구분한다. |
| `du` 합계보다 `df` 사용량이 큼 | 파일시스템과 열린 파일 | 예약 공간·메타데이터·삭제됐지만 프로세스가 열어 둔 파일·마운트 아래 숨은 파일 등을 구분한다. |

게스트에서 계층을 대조할 때:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
sudo pvs
sudo vgs
sudo lvs
findmnt /var/log
df -h / /home /var/log
df -i / /home /var/log
sudo du -xhd1 /var/log
```

`du -x`는 다른 파일시스템으로 넘어가지 않고 파일별 사용량을 조사한다. 큰 트리에서는 읽기 부하가 생기므로 문제가 난 경로부터 좁힌다. 삭제된 열린 파일은 `lsof`가 설치되어 있다면 `sudo lsof +L1`로 소유 프로세스를 찾는다. 프로세스의 로그 재열기 또는 계획된 재시작으로 참조를 해제할 수 있지만, 서비스 영향과 로그 보존을 먼저 판단한다. 공간이 찼다고 `/var/log` 전체를 지우거나 sudo I/O 세션 파일 일부만 잘라내지 않는다.

## 마운트는 디렉터리와 장치를 연결한다

`/var/log`라는 경로가 존재하는 것과 로그 LV가 그곳에 마운트된 것은 다르다. 마운트가 빠진 상태에서 쓴 파일은 root 파일시스템의 같은 디렉터리에 쌓일 수 있다. 나중에 LV를 마운트하면 그 파일이 사라진 듯 보이지만 아래에 가려진 것이다. 따라서 경로 접근 성공만으로 의도한 디스크 사용을 증명하지 않는다.

`/etc/fstab`에는 장치 식별자, 마운트 위치, 유형, 옵션, 덤프·검사 필드가 있다. systemd의 generator는 부팅과 manager reload 때 이를 mount unit으로 해석하며, `mount`도 이 파일을 읽는다. 파일 편집만으로 현재 마운트가 바뀌지는 않는다. 장치 이름 `sda`는 열거 결과이고 UUID는 특정 파일시스템 등을 식별한다. 복제된 파일시스템의 UUID 중복도 가능하므로 UUID가 있다는 이유만으로 연결 대조를 생략하지 않는다. 근거: [util-linux fstab(5)](https://github.com/util-linux/util-linux/blob/master/sys-utils/fstab.5.adoc).

fstab 변경 후 부팅이 막혔다면 해당 줄의 UUID·유형·옵션과 실제 블록 계층을 대조한다. 백업한 원래 줄로 복구하고 구문·연결을 확인한 뒤 마운트와 재부팅을 별도로 검증한다. `findmnt --verify --verbose`는 fstab 진단용이며 실제 마운트를 대신하지 않는다. `mount -a`는 조회가 아니라 여러 항목을 실제 마운트하므로 기본 검사 명령으로 무심코 실행하지 않는다. systemd가 읽는 fstab을 변경했다면 manager의 `daemon-reload`와 필요한 마운트 작업을 구별한다.

## 확장과 축소를 같은 작업으로 보지 않기

로그 공간이 지속적으로 부족하면 먼저 원인을 고치고 보존 기간을 정한다. 이후 VG 여유에서 var_log LV를 늘릴지 결정한다. 확장의 개념 순서는 **백업 → 대상 LV·파일시스템·VG 여유 식별 → LV 확대 → 파일시스템 확대 → `lvs`·`findmnt`·`df`와 쓰기 동작 확인**이다. `lvextend -r`는 파일시스템 확대도 요청하지만 지원 종류·도구·실패 결과를 따로 확인해야 한다. 이미 VG 여유가 있으면 VDI·파티션·LUKS를 먼저 확대할 필요가 없다. 외부 디스크부터 늘릴 경우에는 그 안의 각 계층도 차례로 확장해야 한다.

**영향이 큰 작업:** LV 축소·제거, `mkfs`, `cryptsetup luksFormat`, 파티션 재작성은 데이터 손실을 일으킬 수 있다. ext4 축소는 언마운트 상태에서 파일시스템을 먼저 줄이고 LV를 뒤에 줄이는 순서이며 root는 별도 부팅 환경이 필요하다. XFS는 같은 방식의 축소를 지원하지 않는다. 현재 ext4 구성을 XFS 예제로 복구하지 않는다. 정확한 용량·백업 복원 가능성·작업 중단 시간을 정하기 전에는 실행 명령을 고르지 않는다. ext4의 확대·축소 순서는 [e2fsprogs resize2fs 공식 매뉴얼](https://github.com/tytso/e2fsprogs/blob/master/resize/resize2fs.8.in)의 `DESCRIPTION`과 설치된 `man lvextend`, `man lvreduce`를 함께 읽는다.

LUKS 헤더 백업은 암호화 메타데이터 복구에 쓰이며 파일 데이터 백업과 다르다. 손상된 헤더에 임의의 다른 헤더를 덮어쓰지 않는다. 복구 가능성을 위한 키·헤더·데이터 보관을 함께 설계해야 하며 암호나 키는 Git에 넣지 않는다. 실제 백업과 평가 후 복원은 [기준 디스크 보존 절차](../targets.md#평가-사이의-디스크-보존과-복원)를 따른다.

## 읽는 깊이와 질문

| 깊이 | 이번 범위와 이유 | 공식 자료에서 찾을 부분·질문 |
|---|---|---|
| 직접 설명하고 다룬다 | 부팅 단계, LUKS/PV/VG/LV/파일시스템, fstab·마운트와 네 종류의 여유. 장애 위치와 공간 부족을 구분하는 데 필요하다. | [Rocky System Startup](https://docs.rockylinux.org/books/admin_guide/10-boot/)의 `The GRUB2 bootloader`, `The kernel`, `systemd`; MBR 설명을 우리 GPT 배치와 혼동하지 않는다. “지금 멈춘 단계에서 SSH가 가능해야 하는가?” |
| 직접 설명하고 다룬다 | 소스 장치와 마운트 위치를 대조하고 확인 후 변경한다. | [Rocky File System](https://docs.rockylinux.org/books/admin_guide/07-file-systems/)의 `Mount management commands`, `/etc/fstab file`; 설치된 `man findmnt`, `man df`, `man du`. “VG가 비어 있는데 왜 파일을 못 쓰는가?” |
| 존재와 쓰임을 안다 | initramfs 재생성, LUKS 헤더 백업, rescue 환경, 열린 삭제 파일. 특정 부팅·공간 장애에 필요하다. | RHEL LUKS 문서의 헤더 백업·복구 부분, 설치된 `man cryptsetup`, `man dracut`. “무엇을 되돌리며 무엇은 복구하지 못하는가?” |
| 지금 미룬다 | RAID·thin provisioning·온라인 재암호화·파일시스템 내부 복구. 현재 단일 디스크의 정상 운영에 필요 없고 잘못된 변경 영향이 크다. | 다중 디스크, 풀 초과 할당, 손상 복구가 실제 목표가 될 때 확장한다. |
