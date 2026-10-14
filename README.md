*This project has been created as part of the 42 curriculum by jungblee.*

# Born2beRoot

## Description

Born2beRoot는 가상 머신에 최소 구성의 Linux 서버를 만들고 디스크, 사용자, 권한, 원격 접속과 보안 정책을 직접 관리하는 42 시스템 관리 과제다. 이 저장소는 **과제 v5.2**의 원문과 참고 자료, 반복 실습에서 실제로 배운 내용을 함께 보관한다.

현재는 학습을 이어 가기 위한 진행 중 문서다. 최종 제출용 설치·실행 안내와 설계 설명은 실제 재현과 확인을 거쳐 보완한다.

## 자료와 현재 상태

- [과제 원문 PDF](en.subject.pdf)
- [한국어 전체 번역](docs/ko/Born2beRoot-v5.2-ko-full.md) · [PDF](docs/ko/Born2beRoot-v5.2-ko-full.pdf)
- [요구사항 정리와 평가 체크리스트](docs/ko/Born2beRoot-v5.2-ko-guide.md) · [PDF](docs/ko/Born2beRoot-v5.2-ko-guide.pdf)
- [학습 기록](docs/learning-log.md)
- [직접 만들고 대조할 목표](docs/targets.md): 디스크 구조 예시와 단위별 필수 상태·확인 방법
- [최소 치트시트](docs/cheatsheet.md): 처음부터 재실습하는 절차·명령·설정·기대 결과

사용자는 학교 컴퓨터와 개인 NixOS 환경에서 **Rocky 10.2 Minimal**로 전체 과정을 해봤으며, 아직 익숙하지 않아 반복 실습하고 싶다고 밝혔다. 환경과 버전은 사용자 제공 정보다. 이전 실습의 설치 버전·구성 상태·평가 통과 여부는 독립적으로 확인하지 않았다. 이 경험을 이번 학습의 완료 기록으로 간주하지 않는다.

**이번 반복 실습의 현재 상태:** SSH 일반 사용자 접속·root 직접 접속 거절을 확인했고, `sudo bash monitoring.sh`로 수집 지표 12개가 권한 오류 없이 출력됨을 확인했다. 현재 전체 소스를 요청해 검토를 기다리는 단계이며 방송·설치·예약은 아직 진행하지 않았다. who와 uptime의 시각 차이는 원인 미확정으로 보존하며, 비밀번호 품질 위반 거절(`difok` 포함)도 미검증이다. 세부 관측값과 검증 범위는 [학습 기록](docs/learning-log.md)에 둔다.

원문·번역·가이드는 요구사항을 찾는 자료이고, 학습 기록은 직접 적용한 결과를 남기는 문서다. 문서를 읽거나 예시를 확인한 것만으로 체크리스트를 완료 처리하지 않는다.

## Instructions

### 학습 흐름

공식 입문 자료로 전체 구조를 살핀 뒤, 지금 필요한 개념과 선택 근거를 좁혀 읽는다. 작은 단위로 직접 실습하고 결과를 확인한 다음, 실제로 배운 내용만 간결한 한국어로 기록한다. 이후 전체 구조에서 이번 작업의 위치를 다시 확인한다. 기본 문법 풀이와 코드의 단순 번역, 미리 완성된 설정을 따라 적는 방식은 피한다.

| 순서 | 학습 단위 | 확인할 질문 |
|---|---|---|
| 1 | 설치 결과 확인 | 부팅한 OS·디스크·마운트가 계획과 일치하는가? |
| 2 | 계정·그룹·권한 | 사용자와 파일 권한을 구분하고 호스트명을 바꿀 수 있는가? |
| 3 | 비밀번호·PAM | 품질·만료 규칙이 기존 계정에도 적용되는가? |
| 4 | sudo 정책·감사 로그 | 권한 위임과 제한, 입출력 기록이 실제 동작하는가? |
| 5 | SSH·firewalld·SELinux | 접속 경로의 각 계층이 허용·거절을 어떻게 결정하는가? |
| 6 | 모니터링·cron·wall | 지표의 근거와 방송·예약·중단을 확인할 수 있는가? |
| 7 | 평가 재현·README·서명 | 재부팅 후 재현하고 선택 이유와 제출물을 설명할 수 있는가? |

각 구간의 [전체 학습 지도와 목표](docs/targets.md)에 공식 자료의 읽을 절, 반드시 설명할 개념, 존재만 알아둘 기능, 심화를 미룰 이유와 다시 읽을 시점을 정리했다. 먼저 [자료 선택과 읽는 깊이](docs/targets.md#자료-선택과-읽는-깊이)를 보고 해당 구간으로 이동한다.

### 첫 단위: 디스크 구조 읽기

원문 인쇄 7–8쪽의 요구사항과 그림에서 시작한다. [설치 결과와 디스크 확인](docs/targets.md#1-설치-결과와-디스크-확인)에 정한 순서로 공식 입문 자료를 읽고, 파티션·암호화·LVM·파일시스템·마운트의 관계를 실제 출력과 연결한다. 자세한 읽기 범위는 목표 문서 한곳에서 관리한다.

### 실행·제출 안내의 현재 범위

과제는 VirtualBox를 사용하며, 사용할 수 없다면 UTM을 사용한다. Debian 또는 Rocky의 최신 안정 버전으로 최소 서비스를 구성하고 그래픽 서버는 설치하지 않는다. 현재 저장소에는 검증된 설치·실행 절차가 없으며 반복 실습 후 여기에 정리한다. 전체 조건은 기존 가이드를 참고한다.

v5.2의 README 요구사항과 제출 장에 따라 최종 제출에는 루트 `README.md`와 실제 가상 디스크의 SHA-1을 담은 `signature.txt`가 필요하다. 일반 지침의 서명 파일만 언급한 문장과 함께 읽어야 한다. VM은 Git에 넣지 않는다. 각 평가는 스냅샷 없이 시작하고 평가용 스냅샷은 이후 생성해 종료 시 삭제한다. VM을 다시 실행하면 서명이 달라질 수 있으므로 최종 디스크와 제출 서명을 대조해야 한다. 현재 서명 파일은 준비하지 않았다.

## Project description — 학습 후 보완할 내용

아래 항목은 아직 이번 반복 실습에서 근거를 정리하지 않았다. 선택하거나 확인한 시점에 채운다.

- OS 선택 이유와 Debian·Rocky 각각의 장단점.
- 파티션과 용량, 보안 정책, 사용자 관리, 설치 서비스의 설계 결정.
- Debian vs Rocky Linux, AppArmor vs SELinux, UFW vs firewalld, VirtualBox vs UTM 비교.

## Resources

과제 조건은 원문을 기준으로 하고 공식 문서는 개념과 도구를 이해하는 데 사용한다. 구간별 읽기 범위와 자료 선정 원칙은 [목표 문서](docs/targets.md#자료-선택과-읽는-깊이)에 모았다. Rocky 공식 입문서와 RHEL 10 문서의 제목·절은 2026-10-13에 대조했으며, 설치 패키지의 동작은 VM에서 별도로 확인한다.

- [42 Born2beRoot v5.2 공식 과제](https://cdn.intra.42.fr/pdf/pdf/197915/en.subject.pdf): 요구사항과 평가·제출 조건.
- [Rocky Linux — File System](https://docs.rockylinux.org/books/admin_guide/07-file-systems/): 디스크에서 파일시스템과 마운트까지의 입문 흐름.
- [RHEL 10 — Disk partitions](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/disk-partitions): 파티션 개요, GPT·MBR 구분, 마운트 지점.
- [RHEL 10 — Overview of logical volume management](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_logical_volumes/overview-of-logical-volume-management): PV·VG·LV의 관계와 LVM의 목적.
- [RHEL 10 — Encrypting block devices by using LUKS](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/encrypting-block-devices-by-using-luks): 첫 단위에서는 17.1절의 암호화 개념부터 읽는다.

### AI 활용 범위

AI는 한국어 번역·요구사항 정리 자료와 README·학습 기록·목표 문서의 문서화, 공식 자료 탐색 및 링크 확인에 활용했다. AI는 monitoring의 단계별 부분 예시와 설명을 준비하고 작업 환경에서 예시를 검사했다. VM 적용·실행은 사용자가 수행했으며, 전달받은 결과와 AI 작업 환경의 검사를 구분해 기록했다. AI가 사용자 VM을 설치하거나 보안 설정을 변경하지 않았다. 이후에도 스스로 판단하고 직접 적용한 결과를 바탕으로 설명을 점검하며, 실제 AI 사용 범위를 이곳에 갱신한다.
