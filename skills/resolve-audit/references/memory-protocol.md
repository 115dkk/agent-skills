# Memory 프로토콜

## 판정: Memory를 반드시 사용한다

resolve-audit는 수십 개의 findings를 처리하는 장시간 세션이다.
Category A의 TDD 루프 하나만으로도 상당한 컨텍스트를 소모하며,
30개 이상의 findings를 처리하면 초반 findings의 맥락이
컨텍스트 윈도에서 밀려난다. /compact가 자동 또는 수동으로
실행되면 구조화된 상태 정보가 손실될 수 있다.

대화 컨텍스트에만 의존하면 치매가 발생한다:
- 어떤 finding을 처리했는지 잊는다
- 재분류 이력을 잊는다
- 의존성 관계를 잊고 순서를 어긴다
- 이미 실패한 접근을 반복한다

## 상태 기록 수단: 파일 + 작업 목록 도구

두 가지를 병용한다.

### 1. audit-state.md (파일 기반, 영구적)

프로젝트 루트에 `audit-state.md`를 생성하여 구조화된 상태를
기록한다. 이 파일은 /compact, 세션 재시작, 에이전트 교체에도
보존된다. `.gitignore`에 추가하여 커밋 대상에서 제외한다.

파일 구조:

```markdown
# Audit State — Issue #<NUMBER>
Updated: <ISO timestamp>

## Classification Table
| ID | Audit ID | Finding | Cat | Severity | Effort | Status | Notes |
|----|----------|---------|-----|----------|--------|--------|-------|
| A-1 | TD-07 | Missing error handling | A | High | S | DONE | commit abc1234 |
| A-2 | TD-11 | Edge case in parser | A | High | M | IN_PROGRESS | Red 완료, Green 진행 중 |
| B-1 | TD-12 | Split god file | B | Medium | L | PENDING | depends on A-1 |
| B-2 | C3 | Remove circular dep | B→A | Medium | M | RECLASSIFIED | 테스트 실패로 A-4로 전환 |

## Dependency Graph
A-1 → B-1 (B-1은 A-1 완료 후 처리)
B-2 → B-4 (파일 겹침)

## Execution Order
1. ✅ A-1 (commit abc1234)
2. ⬜ A-2 (in progress)
3. ⬜ B-1 (blocked by A-1 → resolved)
...

## Reclassification Log
- B-2 → A-4: test_parser_output 실패, 공개 인터페이스 변경 감지

## Test Baseline
- Initial run: 142 passed, 0 failed, 3 skipped
- After A-1: 143 passed, 0 failed, 3 skipped (+1 new test)
- After A-2: (pending)

## Failed Approaches
- A-2: try/catch 방식으로 시도했으나 상위 호출자가
  예외를 기대하는 구조. Result 타입 반환으로 전환.
```

### 2. 작업 목록 도구 (작업 진행 상태, 세션 내)

현재 진행 중인 작업과 다음 작업을 세션이 제공하는 작업 목록 도구(TodoWrite, TaskCreate 등)로 관리한다.
/compact 후에도 TodoList는 유지된다.

```
작업 목록:
- [x] Phase 0: 분류 완료, 사용자 확인 완료
- [x] A-1: Missing error handling (DONE)
- [ ] A-2: Edge case in parser (IN PROGRESS - Green)
- [ ] B-1: Split god file (BLOCKED → A-1 완료로 해제)
- [ ] Phase 2 시작 전: 전체 테스트 실행
```

## 갱신 타이밍

다음 시점에 반드시 audit-state.md를 갱신한다:

1. **Phase 0 완료 시**: 분류 테이블, 의존성 그래프,
   실행 순서, 테스트 베이스라인을 초기 기록.

2. **각 finding 완료 시**: Status를 DONE으로 변경,
   커밋 해시 기록, 테스트 결과 업데이트.

3. **재분류 발생 시**: Reclassification Log에 기록,
   실행 순서 재계산, 의존성 그래프 갱신.

4. **/compact 실행 직전**: 반드시 현재 상태를 파일에
   기록한 뒤 /compact한다. /compact 후 첫 행동은
   audit-state.md를 읽는 것이다.

5. **Phase 전환 시** (1→2, 2→3): 전체 테스트 실행 결과를
   Test Baseline에 추가.

6. **실패한 접근 발견 시**: Failed Approaches에 즉시 기록.
   같은 실수를 반복하지 않기 위해.

## 치매 방지 원칙

### 대화 컨텍스트를 신뢰하지 않는다

상태 정보의 정본(source of truth)은 항상 audit-state.md이다.
"아까 A-2를 처리했던 것 같은데"라는 기억이 아니라,
파일에 기록된 Status 필드가 진실이다.

### 복원 가능한 형태로 기록한다

각 커밋 메시지에 finding ID를 포함하므로, 최악의 경우
git log에서 상태를 복원할 수 있다:

```bash
git log --oneline --grep="audit:" fix/audit-42
```

이 명령으로 어떤 findings가 커밋되었는지 확인 가능.

### 산문이 아닌 구조로 기록한다

```
나쁜 예 (산문):
"A-1을 처리했고, 테스트가 통과했습니다.
다음으로 A-2를 시작했는데, 처음에 try/catch로
접근했다가 실패해서 Result 타입으로 바꿨습니다."

좋은 예 (구조):
| A-1 | ... | DONE | commit abc1234 |
| A-2 | ... | IN_PROGRESS | Red 완료 |
Failed: try/catch → Result 타입 전환
```

산문은 /compact 시 요약되면서 세부 정보가 소실된다.
구조화된 테이블과 키-값 쌍은 정보 밀도가 높아
소실 시에도 핵심이 보존될 확률이 높다.

### 주기적으로 정합성을 검증한다

5개 커밋마다 다음을 수행한다:

1. git log에서 커밋된 finding ID 목록 추출
2. audit-state.md의 DONE 항목과 대조
3. 불일치가 있으면 파일을 git log 기준으로 보정

이것은 파일 갱신을 깜빡한 경우에 대한 안전망이다.

## 세션 재개 시

세션이 끊기거나 새 세션에서 이어서 작업할 때:

1. `cat audit-state.md`로 전체 상태를 읽는다.
2. `git log --oneline fix/audit-<NUMBER>`로
   실제 커밋 상태를 확인한다.
3. 두 정보를 대조하여 현재 위치를 확정한다.
4. 작업 목록을 재구성한다.
5. 다음 미완료 항목부터 이어서 처리한다.

## 종료 시

모든 findings 처리가 끝나면:

1. audit-state.md의 모든 항목이 DONE 또는 NOT_RESOLVED인지
   확인한다.
2. PR 생성 후, audit-state.md를 삭제한다.
   (이미 .gitignore에 있으므로 커밋되지 않았음)
3. PR description이 audit-state.md의 최종 상태를
   반영하는지 확인한다.
