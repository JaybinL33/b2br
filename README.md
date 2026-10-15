*This project has been created as part of the 42 curriculum by jungblee.*

# Born2beRoot 학습 저장소

이 private GitHub 저장소는 Born2beRoot v5.2의 원문·참고 자료와 실제 실습 기록을 보관한다. **이 루트 README는 학습 저장소 안내이며 제출용 README가 아니다.**

## 제출용 문서

[submission/README.md](submission/README.md)는 **실제 42 제출 저장소의 루트에 `README.md`라는 이름으로 복사할 독립 문서**다. 프로젝트 설명, 실행 방법, 설계 이유와 필수 비교, 참고 자료·AI 활용 설명을 담으며 이 학습 저장소의 문서에 의존하지 않는다.

원문은 제출 저장소 루트의 `README.md`와 실제 가상 디스크 SHA-1을 담은 `signature.txt`를 요구한다. `submission/` 폴더 전체나 학습 자료를 그대로 제출하는 구조가 아니다. 실제 학교 제출 저장소에는 접근하거나 푸시하지 않았으며, 이곳에 `signature.txt`를 임의 생성하지 않는다. 디스크 보존과 서명 대조는 [평가 준비 절차](docs/targets.md#평가-사이의-디스크-보존과-복원)에 정리되어 있다.

## 학습 자료

- [디펜스 핵심 정리](docs/defense.md): 첨부 평가 순서별 답변·시연·복구와 미확인 준비 항목

- [공식 과제 원문](en.subject.pdf)
- [한국어 전체 번역](docs/ko/Born2beRoot-v5.2-ko-full.md) · [PDF](docs/ko/Born2beRoot-v5.2-ko-full.pdf)
- [요구사항·평가 체크리스트](docs/ko/Born2beRoot-v5.2-ko-guide.md) · [PDF](docs/ko/Born2beRoot-v5.2-ko-guide.pdf)
- [개념 설명서](docs/concepts.md): 개념 학습은 여기서 시작한다. 운영체제·Linux·가상화·서버 운영을 16개 장의 본문과 [개념 색인](docs/concepts/inventory.md)으로 설명
- [Linux 디렉터리 지도](docs/directories.md): 경로의 역할·수명·관리 주체와 실제 b2br 설정·실행·로그 위치
- [구성·운영·진단·복구 안내서](docs/targets.md): 현재 구성의 전체 구조와 증상별 진입점, 공식 자료·복구 절차
- [치트시트](docs/cheatsheet.md): 반복 실습의 명령·설정·기대 결과
- [스피드런](docs/speedrun.md): Rocky 10.2 Minimal 신규 재구축의 필수 입력만 압축
- [학습 기록](docs/learning-log.md): 실제 적용 결과, 확인 근거와 남은 검증

공식 입문 자료로 전체 구조를 보고, 당장 필요한 세부사항을 좁혀 읽고 직접 적용한다. 원문의 요구사항, 안내 예시, 실제 실행 결과는 구분하며 완료 여부는 학습 기록에서 관리한다. 실습 구성은 Rocky Minimal·VirtualBox, LUKS 위 LVM, SSH 4242, firewalld·SELinux다.
