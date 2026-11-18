# SCANPREFIX 테스트 케이스

이 문서를 읽으면 `SCANPREFIX` 명령어가 어떤 상황에서 어떻게 동작해야 하는지 케이스별로 확인할 수 있어요. prefix로 시작하는 살아 있는 key를 `{"key", "value", "ttl"}` 목록으로 돌려주는 명령어예요.

## 한눈에 보기

- 전체 15개이고 모두 **작성 완료**예요.
- 결과 순서는 정해져 있지 않아서, 테스트는 key로 정렬해서 비교해요.
- 시간은 [Phase 4](../phase4/test-cases.md)와 같은 가짜 시계로 건너뛰어요.

## 테스트 방식

| 접두어 | 대상 | 방식 | 파일 |
| --- | --- | --- | --- |
| `SCP-01` ~ `SCP-12` | 명령어 | HTTP 없이 `dispatcher.execute`를 바로 불러요. 가짜 시계를 넣은 빈 저장소로 시작해요. | `tests/test_dispatcher.py` |
| `SCP-13` | 클라이언트 | 서버를 띄우고 `MiniRedis.scanprefix`로 요청해요. 실제 시계를 써요. | `tests/test_client.py` |
| `SCP-14` ~ `SCP-15` | CLI 출력 | `format_result`를 바로 불러요. | `tests/test_cli.py` |

표기법은 [Phase 4 표기법](../phase4/test-cases.md#표기법)과 같아요. `scan P`는 `SCANPREFIX P`를 key로 정렬한 결과예요.

## 동작

| ID | 시나리오 | 준비 (Given) | 실행 (When) | 기대 결과 (Then) | 상태 |
| --- | --- | --- | --- | --- | --- |
| SCP-01 | prefix가 맞는 key만 돌려줌 | set msg:1 a, set msg:2 b, set likes:1 0 | scan msg: | `[{msg:1, a, -1}, {msg:2, b, -1}]` | 작성 완료 |
| SCP-02 | 맞는 key가 없으면 빈 목록 | set likes:1 0 | scan msg: | `[]` | 작성 완료 |
| SCP-03 | 빈 prefix는 모든 key | set a 1, set b 2 | scan "" | key `a`, `b` | 작성 완료 |
| SCP-04 | ttl은 남은 초를 올림 | set msg:1 a 1800 | now 18.2 → scan msg: | ttl 1782 | 작성 완료 |
| SCP-05 | 만료 key는 빼고 TOMBSTONE으로 정리 | set msg:1 a 10, set msg:2 b 100 | now 10 → scan msg: → info | key `msg:2`만 → size 1, tombstones 1 | 작성 완료 |
| SCP-06 | prefix가 다른 만료 key도 정리 | set likes:1 0 10 | now 10 → scan msg: → info | `[]` → size 0 | 작성 완료 |
| SCP-07 | 대소문자 구분 | set MSG:1 a | scan msg: | `[]` | 작성 완료 |
| SCP-08 | 테이블이 커진 뒤에도 모두 찾음 | msg:1 ~ msg:20 저장 | scan msg: | 20개 | 작성 완료 |

## 인자 검사

| ID | 시나리오 | 실행 (When) | 기대 결과 (Then) | 상태 |
| --- | --- | --- | --- | --- |
| SCP-09 | 인자 개수 | SCANPREFIX / SCANPREFIX msg: x | WRONG_ARGS (400) | 작성 완료 |
| SCP-10 | 문자열이 아닌 prefix | SCANPREFIX 123 | WRONG_TYPE (400) | 작성 완료 |
| SCP-11 | 99byte를 넘는 prefix | SCANPREFIX k×100 | KEY_TOO_LONG (400) | 작성 완료 |
| SCP-12 | 명령어 이름 대소문자 무시 | scanprefix msg: | 성공 | 작성 완료 |

## 클라이언트와 CLI

| ID | 시나리오 | 실행 (When) | 기대 결과 (Then) | 상태 |
| --- | --- | --- | --- | --- |
| SCP-13 | 클라이언트 메서드 | set msg:1 a 1800, set likes:1 0 → `client.scanprefix("msg:")` | 1개, ttl 1 ~ 1800 | 작성 완료 |
| SCP-14 | CLI는 항목마다 번호, 둘째 줄부터 들여쓰기 | `format_result([{msg:1, a, 30}, {msg:2, b, -1}])` | `1) key: msg:1` / `   value: a` / … | 작성 완료 |
| SCP-15 | CLI 빈 목록 | `format_result([])` | `(empty array)` | 작성 완료 |
