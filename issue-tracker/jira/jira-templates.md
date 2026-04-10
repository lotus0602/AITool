# Jira 이슈 템플릿

> Jira 이슈 유형별 템플릿과 필드 작성 가이드.
> AI가 Jira 이슈를 생성하거나 작성을 보조할 때 참조한다.

> 티켓 유형별 설명 작성 기본은 [티켓 컨벤션](../common/ticket-convention.md), Jira 워크플로는 [Jira 워크플로](./jira-workflow.md)를 참조한다.

---

## CLAUDE.md 삽입용

아래 블록을 프로젝트 CLAUDE.md에 복사하여 사용한다.

```
## Jira 이슈 작성

이슈 유형 매핑:
  Bug   → Jira Bug
  Story → Jira Story (사용자 관점, Acceptance Criteria 필수)
  Task  → Jira Task (기술 작업, 완료 조건 필수)
  Epic  → 여러 Story/Task를 묶는 상위 단위

필수 필드: Summary, Description, Priority, Labels
Bug 추가 필수: Environment, Steps to Reproduce
Story 추가 필수: Acceptance Criteria
Summary 형식: [type] 구체적인 설명 (50자 이내)
```

---

## 1. 이슈 유형 매핑

| common type | Jira 이슈 유형 | 비고 |
|-------------|---------------|------|
| bug | Bug | 1:1 대응 |
| story | Story | 1:1 대응 |
| feature | Story 또는 Task | 사용자 관점이면 Story, 기술 작업이면 Task |
| task | Task | 1:1 대응 |
| spike | Task + `spike` 라벨 | Jira 기본 유형에 없으므로 라벨로 구분 |

> 각 type의 상세 정의는 [티켓 컨벤션](../common/ticket-convention.md)을 참조한다.

---

## 2. Bug 템플릿

### 필드 구성

| 필드 | 필수 | 작성 규칙 |
|------|------|----------|
| Summary | 필수 | `[bug] 구체적 증상` (50자 이내) |
| Priority | 필수 | Critical / High / Medium / Low |
| Labels | 필수 | `bug` + 영향 모듈 |
| Components | 권장 | 영향 받는 시스템 모듈 |
| Environment | 필수 | OS, 브라우저, 서버 환경, 앱 버전 |
| Fix Version | 권장 | 수정 목표 버전 |

### Description 템플릿

```markdown
h2. 재현 절차
# [전제 조건/초기 상태]
# [1단계 행동]
# [2단계 행동]
# ...

h2. 기대 동작
[정상적으로 동작해야 하는 결과]

h2. 실제 동작
[현재 발생하는 문제]

h2. 환경 정보
* 브라우저: Chrome 120
* OS: macOS 14.2
* 서버: staging (v2.3.1)
* API 버전: v2

h2. 스크린샷/로그
[해당 시 첨부]

h2. 발생 빈도
[항상 / 간헐적(10회 중 N회) / 1회]
```

### 작성 예시

**Summary**: `[bug] 로그인 5회 실패 후 계정 잠금이 동작하지 않음`

**Description**:
```
h2. 재현 절차
# 로그인 페이지(https://app.example.com/login)에 접속한다.
# 유효한 이메일 + 잘못된 비밀번호로 5회 로그인을 시도한다.
# 6번째 로그인을 시도한다.

h2. 기대 동작
계정이 잠기고 "계정이 잠겼습니다. 30분 후 재시도해주세요." 메시지가 표시된다.

h2. 실제 동작
계정이 잠기지 않고 계속 로그인 시도가 가능하다. 에러 메시지는 일반 "비밀번호가 틀렸습니다."만 표시.

h2. 환경 정보
* 브라우저: Chrome 120.0.6099.129
* OS: macOS 14.2
* 서버: staging (v2.3.1)

h2. 스크린샷/로그
!screenshot-login-attempt.png!

h2. 발생 빈도
항상 (10/10)
```

---

## 3. Story 템플릿

### 필드 구성

| 필드 | 필수 | 작성 규칙 |
|------|------|----------|
| Summary | 필수 | `[story] 사용자가 ~할 수 있다` 또는 `[feature] 구체적 기능` |
| Priority | 필수 | Critical / High / Medium / Low |
| Labels | 필수 | `story` 또는 `feature` + 영향 모듈 |
| Components | 권장 | 영향 받는 시스템 모듈 |
| Story Points | 권장 | 피보나치 (1, 2, 3, 5, 8) |
| Epic Link | 권장 | 소속 Epic |
| Fix Version | 권장 | 목표 릴리스 버전 |

### Description 템플릿

```markdown
h2. 배경
[왜 이 기능이 필요한지, 어떤 문제를 해결하는지]

h2. 사용자 스토리
사용자로서, [목적]을 위해, [기능]을 원한다.

h2. 요구사항
* [구체적 요구사항 1]
* [구체적 요구사항 2]
* [구체적 요구사항 3]

h2. 수용 기준 (Acceptance Criteria)
* [ ] [검증 가능한 조건 1]
* [ ] [검증 가능한 조건 2]
* [ ] [검증 가능한 조건 3]

h2. 디자인/참고 자료
[디자인 시안 링크, 관련 문서 링크]

h2. 기술 메모 (선택)
[구현 시 참고할 기술 사항]
```

### 작성 예시

**Summary**: `[story] 사용자가 Google 계정으로 소셜 로그인할 수 있다`

**Description**:
```
h2. 배경
현재 이메일/비밀번호 로그인만 지원하여, 간편 로그인을 원하는 사용자가 가입 단계에서 이탈하고 있다 (가입 완료율 62%).

h2. 사용자 스토리
사용자로서, 빠른 가입/로그인을 위해, Google 계정으로 소셜 로그인을 원한다.

h2. 요구사항
* Google OAuth2 프로토콜 기반 로그인 지원
* 기존 이메일 계정과 동일 이메일 시 자동 연결
* 최초 소셜 로그인 시 닉네임 입력 화면 노출

h2. 수용 기준 (Acceptance Criteria)
* [ ] 로그인 페이지에 "Google로 로그인" 버튼이 표시된다
* [ ] 버튼 클릭 시 Google OAuth2 인증 플로우가 실행된다
* [ ] 인증 성공 시 JWT 토큰이 발급되고 메인 페이지로 이동한다
* [ ] 기존 이메일 계정과 동일 이메일이면 계정이 자동 연결된다
* [ ] 최초 로그인 시 닉네임 입력 화면이 노출된다
* [ ] Google 인증 실패/취소 시 에러 메시지가 표시된다

h2. 디자인/참고 자료
* 디자인 시안: [Figma 링크]
* Google OAuth2 문서: https://developers.google.com/identity/protocols/oauth2
```

---

## 4. Task 템플릿

### 필드 구성

| 필드 | 필수 | 작성 규칙 |
|------|------|----------|
| Summary | 필수 | `[task] 구체적 작업 내용` |
| Priority | 필수 | Critical / High / Medium / Low |
| Labels | 필수 | `task` + 카테고리 (`refactor`, `infra`, `tech-debt` 등) |
| Components | 권장 | 영향 받는 시스템 모듈 |
| Story Points | 권장 | 피보나치 (1, 2, 3, 5, 8) |
| Due Date | 선택 | 기한이 있는 작업 |

### Description 템플릿

```markdown
h2. 목적
[왜 이 작업이 필요한지]

h2. 작업 내용
* [구체적 작업 1]
* [구체적 작업 2]
* [구체적 작업 3]

h2. 완료 조건
* [ ] [검증 가능한 조건 1]
* [ ] [검증 가능한 조건 2]

h2. 참고 사항 (선택)
[주의할 점, 관련 문서 링크]
```

### 작성 예시

**Summary**: `[task] user 테이블 email 컬럼에 인덱스 추가`

**Description**:
```
h2. 목적
사용자 조회 API 응답 시간이 3초 이상 소요되고 있다. user 테이블(50만 row) email 컬럼에 인덱스가 없어 풀스캔이 발생하는 것이 원인.

h2. 작업 내용
* user 테이블 email 컬럼에 B-tree 인덱스 추가 (마이그레이션)
* 조회 쿼리에서 불필요한 JOIN 제거 (user → user_profile)
* 쿼리 실행 계획(EXPLAIN) 확인

h2. 완료 조건
* [ ] 인덱스 추가 마이그레이션 작성 및 staging 적용
* [ ] GET /api/users?email= 응답 시간 500ms 이내
* [ ] 기존 테스트 전체 통과

h2. 참고 사항
* 인덱스 생성 시 테이블 락 주의 — CREATE INDEX CONCURRENTLY 사용
* 관련 이슈: PROJ-98 (사용자 조회 느림 버그)
```

---

## 5. Epic 가이드

### Epic이란

여러 Story/Task를 하나의 큰 목표로 묶는 상위 단위. 별도 템플릿보다는 운영 가이드가 중요하다.

### Epic 필드

| 필드 | 필수 | 작성 규칙 |
|------|------|----------|
| Epic Name | 필수 | 간결한 기능/목표명 (예: "소셜 로그인", "결제 시스템 리뉴얼") |
| Summary | 필수 | Epic Name과 동일하거나 약간 상세하게 |
| Description | 필수 | 목표, 범위, 성공 기준 |

### Epic Description 구조

```markdown
h2. 목표
[이 Epic이 달성하려는 것]

h2. 범위
* 포함: [이 Epic에 포함되는 것]
* 제외: [명시적으로 제외하는 것]

h2. 성공 기준
* [ ] [측정 가능한 목표 1]
* [ ] [측정 가능한 목표 2]

h2. 하위 이슈
(Jira에서 자동으로 링크됨)
```

### Epic-Story-Task 계층

```
Epic: 소셜 로그인
├── Story: 사용자가 Google로 로그인할 수 있다
│   ├── Task: OAuth2 클라이언트 설정
│   └── Task: 로그인 UI 구현
├── Story: 사용자가 Apple로 로그인할 수 있다
│   ├── Task: Apple Sign In 연동
│   └── Task: 로그인 UI에 Apple 버튼 추가
└── Task: 소셜 로그인 통합 테스트 작성
```

### Epic 분할 기준

| 신호 | 대응 |
|------|------|
| 3스프린트 이상 소요 예상 | Epic을 분할 |
| 하위 이슈 10개 초과 | Epic을 분할 |
| 독립적인 두 가지 목표 포함 | 목표별로 별도 Epic |
| "A와 B를 한다" | A Epic + B Epic |

---

## 6. 필드 작성 가이드

### Summary (제목)

- [티켓 컨벤션](../common/ticket-convention.md)의 제목 규칙을 따른다.
- Jira에서는 `[type]` 접두사 + 구체적 설명 형식.
- 50자 이내 권장.

### Labels

| 라벨 | 용도 | 예시 |
|------|------|------|
| type 라벨 | 이슈 유형 | `bug`, `story`, `task`, `spike` |
| 모듈 라벨 | 영향 모듈 | `auth`, `payment`, `search` |
| 상태 라벨 | 특수 상태 | `blocked`, `needs-discussion`, `tech-debt` |

### Components

프로젝트의 구조적 모듈을 정의한다. 프로젝트 관리자가 사전에 등록.

```
예시:
  auth       — 인증/인가
  payment    — 결제
  search     — 검색
  frontend   — 프론트엔드 전체
  api        — 백엔드 API
  infra      — 인프라/DevOps
```

### Fix Version

릴리스 버전을 지정한다. [SemVer](https://semver.org/) 형식 권장.

```
1.3.0        — 다음 마이너 릴리스
1.2.1        — 패치 릴리스 (핫픽스)
2.0.0        — 메이저 릴리스 (Breaking Change)
```

---

## 7. Jira 마크업 참고

Jira Description은 Markdown이 아닌 **Jira 마크업**(위키 문법)을 사용한다. (Jira Cloud의 새 에디터는 WYSIWYG이지만, API로 생성 시 마크업 필요.)

| 용도 | Markdown | Jira 마크업 |
|------|----------|------------|
| 제목 | `## 제목` | `h2. 제목` |
| 굵게 | `**텍스트**` | `*텍스트*` |
| 기울임 | `*텍스트*` | `_텍스트_` |
| 코드 | `` `코드` `` | `{{코드}}` |
| 코드 블록 | ````code```` | `{code}...{code}` |
| 링크 | `[텍스트](url)` | `[텍스트\|url]` |
| 목록 | `- 항목` | `* 항목` |
| 번호 목록 | `1. 항목` | `# 항목` |
| 체크박스 | `- [ ] 항목` | `* [ ] 항목` (Cloud only) |
| 이미지 | `![alt](url)` | `!image.png!` |
| 인용 | `> 인용` | `{quote}...{quote}` |

---

## 8. FAQ

**Q: Jira Cloud와 Jira Data Center에서 이 템플릿이 다른가요?**
A: 대부분 동일하다. 차이점은 Cloud의 새 에디터가 WYSIWYG을 지원하여 마크업 없이 직접 작성 가능하다는 점. API를 통한 자동 생성 시에는 동일한 마크업을 사용한다.

**Q: AI가 Jira 이슈를 직접 생성할 수 있나요?**
A: Atlassian MCP 플러그인이 연결되어 있으면 가능하다. 이 템플릿을 참조하여 필드를 채우고 API로 생성한다.

**Q: spike는 왜 별도 이슈 유형이 아닌가요?**
A: Jira 기본 이슈 유형에 spike가 없다. 커스텀 이슈 유형을 추가할 수 있지만, 워크플로 복잡도가 증가한다. Task + `spike` 라벨로 충분히 구분 가능.

**Q: Sub-task에도 이 템플릿을 적용하나요?**
A: Sub-task는 부모 이슈의 컨텍스트 안에서 동작하므로 간소화해도 된다. Summary + 간단한 작업 내용 + 완료 조건 정도면 충분.

**Q: Description에 Markdown을 쓰면 안 되나요?**
A: Jira Cloud의 새 에디터에서는 일부 Markdown을 자동 변환해준다. 하지만 API로 생성하거나 Data Center를 사용하는 경우 Jira 마크업을 사용해야 정상 렌더링된다. 위 마크업 참고표를 활용한다.
