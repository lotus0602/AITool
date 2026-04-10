# GitLab 워크플로

> GitLab 환경에서의 개발 워크플로.
> 공통 브랜치 전략을 GitLab의 MR, CI 파이프라인, 라벨 시스템과 연동한다.

> 브랜치 전략 기본은 [브랜치 전략](../common/branch-strategy.md), 커밋 규칙은 [커밋 컨벤션](../common/commit-convention.md), 리뷰 기준은 [코드 리뷰 가이드](../common/review-guide.md)를 참조한다.

---

## CLAUDE.md 삽입용

아래 블록을 프로젝트 CLAUDE.md에 복사하여 사용한다.

```
## GitLab 워크플로

작업 흐름:
  1. 이슈에서 브랜치 생성 (네이밍: <prefix>/<issue-id>-<description>)
  2. 로컬 작업 → 커밋 (커밋 컨벤션 준수)
  3. push → MR 생성 (MR 템플릿 사용, type 라벨 지정)
  4. CI 파이프라인 통과 + 리뷰 승인 → Squash Merge
  5. 머지 후 소스 브랜치 자동 삭제

MR 머지 조건:
  - CI 파이프라인 통과
  - 최소 1명 Approve
  - 모든 Discussion 스레드 Resolved

라벨 필수: type::* (feat/fix/refactor 등)
이슈 연결: MR 설명에 Closes #이슈번호 기재
```

---

## 1. 기본 전제

이 문서는 아래 공통 규칙 위에 GitLab 특화 내용을 추가한다:

| 공통 문서 | 적용 내용 |
|----------|----------|
| [브랜치 전략](../common/branch-strategy.md) | 브랜치 네이밍, Git Flow / Trunk-based 선택, 머지 전략 |
| [커밋 컨벤션](../common/commit-convention.md) | 커밋 메시지 형식, type 체계 |
| [코드 리뷰 가이드](../common/review-guide.md) | 리뷰 접두사, 체크리스트, AI 리뷰어 가이드라인 |

공통 문서와 중복되는 내용은 여기에 반복하지 않는다.

---

## 2. GitLab Flow 패턴

### 패턴 A: 환경 브랜치 전략

환경(staging, production)별 브랜치를 두어 배포를 제어한다.

```
feature/* ──→ main ──→ staging ──→ production
               ↑ MR     ↑ auto      ↑ manual
               │        deploy      deploy
               │
           리뷰 + CI
```

| 브랜치 | 역할 | 배포 대상 | 머지 방법 |
|--------|------|----------|----------|
| `main` | 통합 브랜치 | 개발 환경 | feature MR 머지 |
| `staging` | 스테이징 | 스테이징 환경 | main에서 자동 또는 수동 머지 |
| `production` | 프로덕션 | 프로덕션 환경 | staging에서 수동 머지 (릴리스) |

**적합한 경우**: 배포 승인이 필요한 프로젝트, 환경별 검증이 필수인 서비스.

### 패턴 B: 릴리스 브랜치 전략

버전 기반 릴리스를 관리한다. 공통 브랜치 전략의 Git Flow를 GitLab에 적용.

```
feature/* ──→ main ──→ release/1.0 ──→ tag: v1.0.0
               ↑ MR         │
               │          bugfix
           리뷰 + CI        ↓
                       release/1.0 ──→ tag: v1.0.1
```

| 브랜치 | 역할 | 머지 방법 |
|--------|------|----------|
| `main` | 통합 브랜치 | feature MR 머지 |
| `release/*` | 릴리스 준비 | main에서 분기, 안정화 후 태그 |

**적합한 경우**: 버전 릴리스가 있는 라이브러리, 모바일 앱.

### 선택 가이드

```
배포 환경이 여러 개인가? (dev/staging/prod)
  ├─ Yes → 환경 브랜치 전략
  └─ No
       └─ 버전 릴리스가 필요한가?
            ├─ Yes → 릴리스 브랜치 전략
            └─ No  → main 단일 브랜치 + Trunk-based
```

---

## 3. MR 프로세스

### 전체 플로우

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  이슈    │ ──→ │ 브랜치   │ ──→ │   MR     │ ──→ │  리뷰    │ ──→ │  머지    │
│  생성    │     │  생성    │     │  생성    │     │ + CI     │     │ + 삭제   │
└──────────┘     └──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                │                │                │                │
     ↓                ↓                ↓                ↓                ↓
  요구사항       이슈에서 직접     MR 템플릿 사용    파이프라인      Squash Merge
  + 라벨       또는 로컬에서      + 라벨 지정      + 코드 리뷰    + 브랜치 삭제
  지정          네이밍 컨벤션                       + 스레드 해결   + 이슈 자동 클로즈
```

### 단계별 상세

**1단계: 이슈 생성**
- 기능 요구사항, 버그 리포트를 이슈로 등록
- 라벨 지정: `type::*`, `priority::*`
- 담당자(Assignee) 지정

**2단계: 브랜치 생성**
- GitLab 이슈 화면 → "Create branch" 버튼 (자동 네이밍)
- 또는 로컬에서 네이밍 컨벤션에 따라 생성:
  ```bash
  git checkout -b feature/PROJ-123-add-login
  ```

**3단계: MR 생성**
- 첫 push 시 터미널에 출력되는 MR 생성 링크 활용
- 또는 `git push` 옵션 사용:
  ```bash
  git push -o merge_request.create \
           -o merge_request.target=main \
           -o merge_request.title="feat(auth): add OAuth2 login" \
           -o merge_request.label="type::feat"
  ```
- [MR 템플릿](./mr-template.md) 작성
- 작업 중이면 Draft MR로 생성

**4단계: CI + 리뷰**
- CI 파이프라인 자동 실행
- 리뷰어 지정 (CODEOWNERS 또는 수동)
- 리뷰 코멘트 대응 ([코드 리뷰 가이드](../common/review-guide.md) 참조)

**5단계: 머지**
- 머지 조건 충족 확인:
  - CI 파이프라인 통과
  - 최소 1명 Approve
  - 모든 Discussion 스레드 Resolved
- Squash Merge 실행 (MR 제목이 최종 커밋 메시지)
- 소스 브랜치 자동 삭제 설정 확인

---

## 4. 라벨 체계

### 네임스페이스 구조

GitLab은 `::` 구분자로 라벨을 계층화할 수 있다.

| 네임스페이스 | 용도 | 필수 | 값 |
|-------------|------|------|-----|
| `type::` | 변경 유형 | MR 필수 | `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `perf` |
| `workflow::` | 진행 상태 | MR 필수 | `in-progress`, `in-review`, `approved`, `blocked` |
| `priority::` | 우선순위 | 이슈 권장 | `critical`, `high`, `medium`, `low` |
| `size::` | MR 크기 | MR 선택 | `S`, `M`, `L`, `XL` |
| `env::` | 대상 환경 | 선택 | `dev`, `staging`, `production` |

### 워크플로 라벨 전이

```
workflow::in-progress → workflow::in-review → workflow::approved → (머지)
                              ↓
                        workflow::blocked (차단 시)
```

---

## 5. GitLab CI 연동

### 파이프라인 종류

| 파이프라인 | 트리거 | 용도 |
|-----------|--------|------|
| MR 파이프라인 | MR 생성/업데이트 시 | 변경사항 검증 (lint, test, build) |
| 브랜치 파이프라인 | push 시 | 브랜치별 CI |
| 태그 파이프라인 | 태그 생성 시 | 릴리스 빌드/배포 |
| 스케줄 파이프라인 | cron | 정기 검증 (보안 스캔 등) |

### MR 파이프라인 필수 Job

```yaml
# .gitlab-ci.yml 기본 구조
stages:
  - lint
  - test
  - build

lint:
  stage: lint
  script:
    - npm run lint          # 또는 프로젝트에 맞는 린트 명령
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

test:
  stage: test
  script:
    - npm run test          # 또는 프로젝트에 맞는 테스트 명령
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

build:
  stage: build
  script:
    - npm run build         # 또는 프로젝트에 맞는 빌드 명령
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

### 파이프라인 실패 시 대응

| 실패 유형 | 대응 |
|----------|------|
| lint 실패 | 코드 수정 후 재push |
| test 실패 | 테스트 수정 또는 코드 수정 후 재push |
| build 실패 | 빌드 오류 수정 후 재push |
| flaky test | 재실행으로 확인 후, flaky면 이슈 생성하여 별도 수정 |

**규칙**: CI 실패한 MR은 머지하지 않는다. "이번만 예외"는 없다.

---

## 6. 이슈-MR 연결

### 자동 클로즈 키워드

MR 설명에 아래 키워드를 사용하면 머지 시 이슈가 자동 클로즈된다:

```
Closes #123
Fixes #123
Resolves #123
```

여러 이슈 연결:
```
Closes #123, #124
Fixes #123
Refs #125        ← 참조만, 클로즈하지 않음
```

### 전체 흐름 다이어그램

```
┌──────────┐     ┌──────────────────┐     ┌──────────┐     ┌──────────┐
│  이슈    │ ──→ │     브랜치       │ ──→ │    MR    │ ──→ │  머지    │
│  #123    │     │ feature/PROJ-123 │     │ Closes   │     │ 이슈 #123│
│  Open    │     │ -add-login       │     │ #123     │     │ Closed   │
└──────────┘     └──────────────────┘     └──────────┘     └──────────┘
```

---

## 7. 팀 워크플로 예시

일일 개발 사이클 시나리오:

```
09:00  이슈 보드 확인, 오늘 작업할 이슈 선택
       → 이슈에 자신을 Assignee로 지정
       → workflow::in-progress 라벨

09:15  브랜치 생성
       $ git checkout -b feature/PROJ-123-add-login

09:15~ 개발 작업 + 커밋
       $ git add -p
       $ git commit -m "feat(auth): add login form component"
       $ git commit -m "feat(auth): add login API integration"

14:00  MR 생성
       $ git push -o merge_request.create
       → MR 템플릿 작성
       → type::feat, workflow::in-review 라벨 지정
       → 리뷰어 지정

14:30  CI 파이프라인 확인
       → 실패 시 수정 후 재push

15:00  리뷰 코멘트 확인 및 대응
       → [must] 항목 수정
       → [should] 항목 논의 또는 수정

16:00  리뷰 승인 + CI 통과
       → Squash Merge 실행
       → 소스 브랜치 자동 삭제
       → 이슈 자동 클로즈 확인

16:10  로컬 브랜치 정리
       $ git checkout main
       $ git pull
       $ git branch -d feature/PROJ-123-add-login
```

---

## 8. GitLab 프로젝트 설정 권장사항

### Merge Request 설정

| 설정 | 권장값 | 이유 |
|------|--------|------|
| Merge method | Squash merge | 깔끔한 히스토리 |
| Squash commits when merging | Encourage (또는 Require) | 일관성 |
| Delete source branch on merge | 활성화 | 브랜치 정리 자동화 |
| Pipeline must succeed | 활성화 | CI 통과 강제 |
| All discussions must be resolved | 활성화 | 미해결 스레드 방지 |
| Minimum approvals | 1 이상 | 리뷰 강제 |

### Branch Protection

| 브랜치 | Push | Merge | Force push |
|--------|------|-------|------------|
| `main` | No one | Maintainers | 금지 |
| `staging` | No one | Maintainers | 금지 |
| `production` | No one | Maintainers | 금지 |
| `release/*` | Developers | Maintainers | 금지 |

### CODEOWNERS

CODEOWNERS는 리포지토리의 **파일/디렉토리별 책임자를 정의하는 파일**이다. 리포지토리 루트에 `CODEOWNERS` 파일을 두면, MR에서 해당 경로가 변경될 때 지정된 사용자/그룹이 **자동으로 리뷰어로 지정**된다.

**역할**:
- MR에 변경된 파일 경로를 매칭하여 리뷰어를 자동 지정
- Approval Rules와 연동하면 해당 CODEOWNER의 승인을 머지 조건으로 강제 가능
- "누가 이 코드를 책임지는가?"를 코드로 명시하여, 담당자 퇴사/이동 시에도 추적 가능

**문법**:

```
# 파일 경로 패턴    @사용자 또는 @그룹

# 전체 코드 — 기본 리뷰어
*                   @team-lead

# 디렉토리 단위
/src/frontend/      @frontend-team
/src/api/           @backend-team

# 특정 파일
/.gitlab-ci.yml     @devops-team
/docker/            @devops-team

# 특정 확장자
*.sql               @dba-team
```

**매칭 규칙**:
- 아래 규칙이 위 규칙을 오버라이드한다 (마지막 매칭이 우선).
- `*`는 전체 파일에 매칭되므로 최상단에 기본값으로 둔다.
- 경로 패턴은 `.gitignore`와 동일한 glob 문법을 사용한다.

**설정 위치**: `Settings > Repository > Protected branches > Code Owners` 에서 CODEOWNER 승인 필수 여부를 설정할 수 있다.

---

## 9. FAQ

**Q: MR 파이프라인과 브랜치 파이프라인 중 어떤 것을 써야 하나요?**
A: MR 파이프라인을 권장한다. `rules: - if: $CI_PIPELINE_SOURCE == "merge_request_event"` 조건으로 MR이 있을 때만 실행하면 불필요한 중복 실행을 방지할 수 있다.

**Q: Squash Merge를 쓰면 커밋 히스토리를 잃는 것 아닌가요?**
A: MR 페이지에서 원본 커밋을 모두 확인할 수 있다. main 브랜치의 히스토리가 깔끔해지는 이점이 더 크다.

**Q: CODEOWNERS 파일은 어떻게 설정하나요?**
A: 위 [CODEOWNERS 섹션](#codeowners)을 참조한다. 리포지토리 루트에 `CODEOWNERS` 파일을 생성하고, 경로별 책임자를 지정한다.

**Q: hotfix는 어떤 프로세스를 따르나요?**
A: 기본 워크플로와 동일하지만, 브랜치를 main(또는 production)에서 직접 분기하고, 리뷰를 최소화(1명 빠른 승인)하여 신속하게 머지한다. 머지 후 develop(또는 staging)에도 반영한다.
