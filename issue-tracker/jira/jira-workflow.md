# Jira 워크플로

> Jira 환경에서의 이슈 관리 워크플로.
> 공통 티켓 워크플로를 Jira의 보드, 스프린트, 자동화, 이슈 링크 시스템과 연동한다.

> 티켓 상태 전이 기본은 [티켓 워크플로](../common/ticket-workflow.md), 티켓 작성 규칙은 [티켓 컨벤션](../common/ticket-convention.md)을 참조한다.

---

## CLAUDE.md 삽입용

아래 블록을 프로젝트 CLAUDE.md에 복사하여 사용한다.

```
## Jira 워크플로

상태 매핑:
  Open        → To Do
  In Progress → In Progress
  In Review   → In Review
  Done        → Done
  Blocked     → 플래그(Flag) 사용 + Blocked 라벨

이슈 키 형식: PROJ-123 (프로젝트 키 + 번호)

스프린트 규칙:
  - 스프린트 목표(Sprint Goal)를 반드시 설정
  - 스프린트 중 범위 추가 최소화
  - 미완료 이슈는 다음 스프린트로 이월 또는 백로그로 복귀

이슈-브랜치 연결:
  브랜치명에 이슈 키 포함: feature/PROJ-123-add-login
  커밋/MR에 이슈 키 포함 → Jira에서 자동 연결
```

---

## 1. 기본 전제

이 문서는 아래 공통 규칙 위에 Jira 특화 내용을 추가한다:

| 공통 문서 | 적용 내용 |
|----------|----------|
| [티켓 워크플로](../common/ticket-workflow.md) | 상태 전이 규칙, 담당자 규칙, 에스컬레이션, 완료 기준 |
| [티켓 컨벤션](../common/ticket-convention.md) | 티켓 유형, 제목 규칙, 설명 작성, 우선순위, 라벨 체계 |

공통 문서와 중복되는 내용은 여기에 반복하지 않는다.

---

## 2. Jira 상태 매핑

### common 상태 → Jira 상태

| common 상태 | Jira 상태 | Jira 카테고리 | 비고 |
|-------------|-----------|-------------|------|
| `Open` | `To Do` | To Do | 기본 생성 상태 |
| `In Progress` | `In Progress` | In Progress | 담당자 착수 시 |
| `In Review` | `In Review` | In Progress | MR/PR 생성 시 |
| `Done` | `Done` | Done | 머지 + 수용 기준 충족 |
| `Blocked` | `In Progress` + Flag | In Progress | 깃발(🚩) 플래그 + `blocked` 라벨 |
| `Closed` | `Done` (Resolution: Won't Do) | Done | 중복/무효/취소 |

**Jira 워크플로 스킴에서의 전이**:

```
┌─────────┐     ┌─────────────┐     ┌───────────┐     ┌─────────┐
│ To Do   │ ──→ │ In Progress │ ──→ │ In Review │ ──→ │  Done   │
│         │     │             │     │           │     │         │
└─────────┘     └─────────────┘     └───────────┘     └─────────┘
                      │    ↑              │
                      │    │              │
                      ↓    │              │
                   (Flag: Blocked)        │
                                          ↓
                                    In Progress
                                    (수정 요청 시)
```

### Jira Resolution 값

| Resolution | 사용 시점 |
|-----------|----------|
| `Done` | 정상 완료 |
| `Won't Do` | 취소, 더 이상 불필요 |
| `Duplicate` | 중복 이슈 |
| `Cannot Reproduce` | Bug 재현 불가 |

---

## 3. 보드 설정

### 스크럼 vs 칸반 선택

| 기준 | 스크럼 보드 | 칸반 보드 |
|------|-----------|----------|
| 릴리스 주기 | 정기 스프린트 (1-4주) | 지속적 흐름 |
| 작업 특성 | 계획 기반, 예측 가능 | 요청 기반, 가변적 |
| 팀 규모 | 5-9명 | 제한 없음 |
| 적합한 팀 | 제품 개발팀 | 운영/유지보수팀, DevOps |
| 핵심 지표 | Velocity, Sprint Burndown | Cycle Time, Throughput |

### 보드 컬럼 구성

```
┌──────────┬──────────────┬───────────┬──────────┐
│  To Do   │ In Progress  │ In Review │   Done   │
│          │              │           │          │
│  PROJ-10 │  PROJ-7      │  PROJ-5   │  PROJ-1  │
│  PROJ-11 │  PROJ-8      │  PROJ-6   │  PROJ-2  │
│  PROJ-12 │              │           │  PROJ-3  │
│          │              │           │  PROJ-4  │
└──────────┴──────────────┴───────────┴──────────┘
```

**WIP(Work In Progress) 제한 권장값**:

| 컬럼 | WIP 제한 | 이유 |
|------|---------|------|
| In Progress | 팀원 수 × 1.5 | 과도한 컨텍스트 전환 방지 |
| In Review | 팀원 수 × 1 | 리뷰 병목 조기 감지 |

---

## 4. 스프린트 운영

### 스프린트 주기

| 주기 | 적합한 상황 |
|------|-----------|
| 1주 | 빠른 피드백 필요, 소규모 팀, 초기 프로젝트 |
| 2주 | 가장 일반적, 균형 잡힌 계획/실행 비율 |
| 3-4주 | 대규모 기능, 외부 의존성 많은 프로젝트 |

### 스프린트 세레모니

| 세레모니 | 시점 | 목적 | 소요 시간 (2주 기준) |
|---------|------|------|-------------------|
| Sprint Planning | 스프린트 시작 | 목표 설정, 이슈 선택, 추정 | 2시간 |
| Daily Standup | 매일 | 진행 상황 공유, 차단 요인 공유 | 15분 |
| Sprint Review | 스프린트 종료 | 완료 작업 데모, 피드백 수집 | 1시간 |
| Sprint Retrospective | 스프린트 종료 | 프로세스 개선점 논의 | 1시간 |

### 스프린트 규칙

| 규칙 | 설명 |
|------|------|
| Sprint Goal 필수 | 스프린트 목표를 한 문장으로 설정. "무엇을 달성할 것인가?" |
| 범위 변경 최소화 | 스프린트 중 이슈 추가는 긴급(critical/high)만 허용 |
| 미완료 이슈 처리 | 스프린트 종료 시 In Progress → 다음 스프린트 이월, To Do → 백로그 복귀 |
| Velocity 추적 | 스프린트별 완료 Story Point 기록, 3회 이상 데이터로 예측 |

### Story Point 추정

피보나치 수열 기반: `1, 2, 3, 5, 8, 13`

| Point | 의미 | 예시 |
|-------|------|------|
| 1 | 매우 작음, 확실함 | 텍스트 수정, 설정 변경 |
| 2 | 작음, 명확함 | 단순 API 엔드포인트 추가 |
| 3 | 보통, 약간의 불확실성 | CRUD 기능 하나 |
| 5 | 큼, 불확실성 있음 | 외부 API 연동 |
| 8 | 매우 큼, 불확실성 높음 | 새로운 모듈 설계 + 구현 |
| 13 | 분할 필요 | 이 크기면 Epic 또는 여러 이슈로 분할 |

---

## 5. JQL 활용

### 자주 쓰는 JQL 쿼리

**내 작업 관련**:

```jql
-- 나에게 할당된 미완료 이슈
assignee = currentUser() AND status != Done ORDER BY priority DESC

-- 내가 리뷰해야 할 이슈
status = "In Review" AND assignee != currentUser() AND project = PROJ

-- 내가 만든 미해결 이슈
reporter = currentUser() AND resolution = Unresolved
```

**스프린트 관련**:

```jql
-- 현재 스프린트 이슈
sprint in openSprints() AND project = PROJ

-- 현재 스프린트 미완료 이슈
sprint in openSprints() AND status != Done AND project = PROJ

-- 이번 스프린트에서 완료된 이슈
sprint in openSprints() AND status = Done AND project = PROJ
```

**트리아지 / 관리**:

```jql
-- 담당자 미배정 이슈
assignee is EMPTY AND project = PROJ AND resolution = Unresolved

-- 7일 이상 To Do 상태인 이슈
status = "To Do" AND created <= -7d AND project = PROJ

-- 플래그(Blocked) 이슈
flagged = "Impediment" AND project = PROJ

-- 이번 버전 미해결 이슈
fixVersion = "1.3.0" AND resolution = Unresolved
```

**버그 관련**:

```jql
-- 미해결 critical/high 버그
type = Bug AND priority in (Critical, High) AND resolution = Unresolved

-- 최근 7일 생성된 버그
type = Bug AND created >= -7d AND project = PROJ
```

---

## 6. 자동화 규칙

Jira Automation으로 반복 작업을 자동화한다.

### 권장 자동화 규칙

| 트리거 | 조건 | 동작 | 목적 |
|--------|------|------|------|
| 이슈 생성 시 | type = Bug, priority = Critical | Slack 알림 발송 | 긴급 버그 즉시 인지 |
| 상태 → In Progress | 담당자 미배정 | 전환한 사람을 담당자로 지정 | 담당자 자동 배정 |
| 상태 → Done | — | 하위 이슈(Sub-task) 전체 Done 확인 | 미완료 하위 작업 방지 |
| 스프린트 종료 시 | 미완료 이슈 | 다음 스프린트로 자동 이월 | 수동 이월 방지 |
| MR 머지 시 | 이슈 키 포함 | 이슈 상태 → Done | 수동 상태 변경 방지 |
| 7일간 상태 변경 없음 | In Progress 상태 | 담당자에게 알림 | 방치 이슈 감지 |

### 자동화 설정 예시

**이슈 생성 시 → 우선순위별 Due Date 자동 설정**:

```
When: 이슈가 생성됨
If:   priority = Critical
Then: Due Date = 생성일 + 1일

If:   priority = High
Then: Due Date = 생성일 + 3일

If:   priority = Medium
Then: Due Date = 현재 스프린트 종료일
```

---

## 7. 이슈 링크

### 링크 유형

| 링크 유형 | 의미 | 사용 시점 |
|----------|------|----------|
| `blocks` / `is blocked by` | 차단 관계 | A가 완료되어야 B를 시작할 수 있을 때 |
| `relates to` | 관련 있음 | 같은 기능 영역의 이슈 |
| `duplicates` / `is duplicated by` | 중복 | 동일한 문제를 다른 이슈로 등록했을 때 |
| `clones` / `is cloned by` | 복제 | 유사한 이슈를 복제하여 생성했을 때 |

**규칙**:
- Blocked 상태 전환 시 반드시 `is blocked by` 링크를 추가한다.
- 중복 이슈 발견 시 `duplicates` 링크 후, 나중에 생성된 이슈를 Won't Do로 닫는다.
- 관련 이슈는 `relates to`로 연결하여 맥락을 보존한다.

---

## 8. Jira + GitLab 연동

### 이슈 키 기반 자동 연결

Jira와 GitLab을 연동하면, 커밋/브랜치/MR에 이슈 키(예: `PROJ-123`)를 포함할 때 Jira에서 자동으로 관련 개발 정보를 표시한다.

| 위치 | 형식 | Jira에서의 표시 |
|------|------|----------------|
| 브랜치명 | `feature/PROJ-123-add-login` | Development 패널에 브랜치 표시 |
| 커밋 메시지 | `feat(auth): add login PROJ-123` | Development 패널에 커밋 표시 |
| MR 제목/설명 | `PROJ-123` 포함 | Development 패널에 MR 표시 |

> 브랜치 네이밍 규칙은 [브랜치 전략](../../git/common/branch-strategy.md), MR 규칙은 [GitLab 워크플로](../../git/gitlab/gitlab-flow.md)를 참조한다.

### 전체 연결 흐름

```
┌──────────┐     ┌──────────────────────┐     ┌─────────────┐     ┌──────────┐
│  Jira    │ ──→ │     GitLab           │ ──→ │  GitLab     │ ──→ │  Jira    │
│ PROJ-123 │     │ feature/PROJ-123     │     │  MR         │     │ PROJ-123 │
│ To Do    │     │ -add-login           │     │  머지       │     │ Done     │
└──────────┘     └──────────────────────┘     └─────────────┘     └──────────┘
     │                    │                         │                   │
     ↓                    ↓                         ↓                   ↓
  이슈 키 확인      브랜치명에               이슈 키 포함          Development
                   이슈 키 포함            → Jira 자동 연결       패널 업데이트
                                                               + 상태 자동 전이
```

### 연동 설정

1. **GitLab → Jira Integration 활성화**: GitLab 프로젝트 설정 → Integrations → Jira
2. **Jira Site URL** 및 **API Token** 입력
3. **Smart Commits 활성화** (선택): 커밋 메시지로 Jira 이슈 상태 변경 가능

**Smart Commit 문법**:

```
PROJ-123 #done           → 이슈 상태를 Done으로 변경
PROJ-123 #in-progress    → 이슈 상태를 In Progress로 변경
PROJ-123 #comment 수정 완료  → 이슈에 코멘트 추가
PROJ-123 #time 2h        → 작업 시간 2시간 기록
```

---

## 9. Jira 프로젝트 설정 권장사항

### 이슈 유형 스킴

| Jira 이슈 유형 | common type 대응 | 아이콘 | 설명 |
|---------------|-----------------|--------|------|
| Epic | (상위 묶음) | 번개 | 여러 Story/Task를 묶는 큰 단위 |
| Story | story | 북마크 | 사용자 관점의 기능 요구사항 |
| Task | task | 체크박스 | 기술 작업 |
| Bug | bug | 벌레 | 결함 |
| Sub-task | (하위 분할) | — | 이슈를 더 작은 단위로 분할 |

> 이슈 유형별 상세 정의는 [티켓 컨벤션](../common/ticket-convention.md)을 참조한다.

### 워크플로 스킴

위 2장의 상태 매핑을 기반으로 워크플로를 설정한다:

```
To Do → In Progress → In Review → Done
```

- 모든 이슈 유형에 동일한 워크플로를 적용한다 (단순성).
- 필요 시 Bug에만 `Verified` 상태를 추가할 수 있다 (QA 검증 프로세스 있는 경우).

### 필드 설정

| 필드 | 필수 여부 | 이슈 유형 |
|------|----------|----------|
| Summary | 필수 | 전체 |
| Description | 필수 | 전체 |
| Priority | 필수 | 전체 |
| Labels | 필수 | 전체 |
| Components | 권장 | 전체 |
| Fix Version | 권장 | Bug, Story |
| Story Points | 권장 | Story, Task |
| Sprint | 자동 | 전체 (보드에서 드래그) |
| Due Date | 선택 | Task, Bug (critical/high) |
| Environment | 필수 | Bug |

---

## 10. FAQ

**Q: Jira에서 Blocked 상태를 별도로 만들어야 하나요?**
A: 별도 상태보다 Flag(깃발) 기능을 활용하는 것을 권장한다. In Progress 상태에서 Flag를 설정하고 `blocked` 라벨을 추가하면 보드에서 시각적으로 구분되면서도 워크플로가 단순하게 유지된다.

**Q: Epic은 어떤 크기여야 하나요?**
A: 1-3 스프린트 안에 완료할 수 있는 크기. 그 이상이면 Epic을 분할한다. Epic 안의 Story/Task가 10개를 초과하면 분할 신호.

**Q: Story Points를 시간으로 환산하면 안 되나요?**
A: 권장하지 않는다. Story Points는 복잡도/불확실성의 상대적 크기이지, 절대 시간이 아니다. "3포인트 = 1.5일"처럼 환산하면 추정의 의미가 퇴색된다.

**Q: Sub-task는 언제 사용하나요?**
A: 하나의 이슈를 여러 사람이 분담하거나, 작업 단계를 명시적으로 추적하고 싶을 때. 단, Sub-task가 3개 이상이면 이슈 분할을 먼저 검토한다.

**Q: Component와 Label의 차이는?**
A: Component는 시스템의 구조적 모듈(auth, payment, search)이고, Label은 유연한 태그(urgent, tech-debt, needs-discussion). Component는 프로젝트 관리자가 정의하고, Label은 누구나 추가할 수 있다.
