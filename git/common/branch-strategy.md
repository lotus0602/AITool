# 브랜치 전략

> 서비스(GitHub, GitLab 등)에 무관한 브랜치 네이밍 컨벤션과 전략 패턴.
> 프로젝트 특성에 맞는 전략을 선택하고 일관되게 적용한다.

---

## CLAUDE.md 삽입용

아래 블록을 프로젝트 CLAUDE.md에 복사하여 사용한다.

```
## 브랜치 전략

네이밍: <prefix>/<issue-id>-<short-description>

prefix:
  feature  — 새 기능
  fix      — 버그 수정
  hotfix   — 프로덕션 긴급 수정
  release  — 릴리스 준비
  docs     — 문서 작업
  chore    — 설정, 빌드 등 기타
  refactor — 리팩토링
  test     — 테스트 관련

예시: feature/PROJ-123-add-login, fix/PROJ-456-null-check

기본 브랜치: main (프로덕션)
개발 브랜치: develop (Git Flow 사용 시)
브랜치 생명주기: 생성 → 작업 → 머지 요청 → 머지 → 삭제
머지 후 원격/로컬 브랜치 반드시 삭제
```

---

## 1. 브랜치 네이밍 컨벤션

### 형식

```
<prefix>/<issue-id>-<short-description>
```

- `prefix`: 필수. 브랜치의 목적을 나타내는 접두사.
- `issue-id`: 권장. 이슈 트래커의 식별자.
- `short-description`: 필수. 케밥 케이스(kebab-case)로 작성.

### prefix 목록

| prefix | 용도 | 커밋 type 대응 |
|--------|------|---------------|
| `feature` | 새 기능 개발 | feat |
| `fix` | 버그 수정 | fix |
| `hotfix` | 프로덕션 긴급 수정 | fix |
| `release` | 릴리스 준비 | chore |
| `docs` | 문서 작업 | docs |
| `chore` | 설정, 빌드, 패키지 등 | chore, ci, build |
| `refactor` | 리팩토링 | refactor |
| `test` | 테스트 추가/수정 | test |

> 커밋 type과의 대응은 [커밋 컨벤션](./commit-convention.md)을 참조한다.

### 예시

```
feature/PROJ-123-add-oauth-login
fix/PROJ-456-fix-null-pointer-on-empty-cart
hotfix/PROJ-789-patch-critical-auth-bypass
release/1.2.0
docs/PROJ-101-update-api-guide
chore/PROJ-202-upgrade-dependencies
refactor/PROJ-303-extract-validation-logic
```

### 금지 패턴

| 패턴 | 이유 |
|------|------|
| `my-branch` | prefix 없음, 목적 불명확 |
| `feature/addLogin` | 케밥 케이스 미사용 |
| `Feature/PROJ-123-add-login` | prefix 대문자 |
| `feature/PROJ-123` | 설명 없음, 이슈 번호만으로는 내용 파악 불가 |
| `feature/add-oauth-login-with-google-and-apple-and-kakao-support` | 너무 긺, 30자 이내 권장 |

---

## 2. 전략 패턴

### 패턴 A: Git Flow

릴리스 주기가 있는 프로젝트에 적합하다.

```
main ─────●─────────────────●─────────── (프로덕션)
           ↑                 ↑
release ──●── 1.0.0 ────────●── 1.1.0 ── (릴리스 준비)
           ↑                 ↑
develop ──●──●──●──●────●──●──●──●────── (통합)
              ↑     ↑      ↑     ↑
feature      A     B      C     D        (기능 개발)
```

**브랜치 역할**:

| 브랜치 | 수명 | 역할 |
|--------|------|------|
| `main` | 영구 | 프로덕션 배포 상태. 직접 커밋 금지. |
| `develop` | 영구 | 다음 릴리스 통합 브랜치. feature의 머지 대상. |
| `feature/*` | 임시 | 기능 개발. develop에서 분기, develop으로 머지. |
| `release/*` | 임시 | 릴리스 준비(버전 업, 버그 수정). develop에서 분기, main+develop에 머지. |
| `hotfix/*` | 임시 | 프로덕션 긴급 수정. main에서 분기, main+develop에 머지. |

**머지 규칙**:

```
feature → develop    : squash merge (깔끔한 히스토리)
develop → release    : branch 생성 (머지 아님)
release → main      : merge commit (릴리스 이력 보존)
release → develop   : merge commit (수정사항 반영)
hotfix  → main      : merge commit
hotfix  → develop   : merge commit
```

---

### 패턴 B: Trunk-based Development

CI/CD 기반 지속적 배포 프로젝트에 적합하다.

```
main ──●──●──●──●──●──●──●──●──●──●──── (프로덕션, 항상 배포 가능)
        ↑     ↑        ↑     ↑
short   A     B        C     D           (수명 짧은 feature)
lived  (1-2일)
```

**브랜치 역할**:

| 브랜치 | 수명 | 역할 |
|--------|------|------|
| `main` | 영구 | 유일한 장기 브랜치. 항상 배포 가능 상태. |
| `feature/*` | 임시 (1-2일) | 짧은 기능 개발. main에서 분기, main으로 머지. |
| `hotfix/*` | 임시 | 긴급 수정. main에서 분기, main으로 머지. |

**머지 규칙**:

```
feature → main    : squash merge
hotfix  → main    : merge commit 또는 squash merge
```

**핵심 원칙**:
- feature 브랜치 수명은 최대 2일. 그 이상이면 분할한다.
- Feature Flag로 미완성 기능을 숨긴다.
- main은 항상 배포 가능 상태를 유지한다.

---

### 선택 기준

| 기준 | Git Flow | Trunk-based |
|------|----------|-------------|
| 릴리스 주기 | 주기적 릴리스 (2주~1달) | 지속적 배포 (매일~매주) |
| 팀 규모 | 5인 이상, 병렬 기능 개발 多 | 5인 이하, 빠른 반복 |
| 환경 분리 | staging / production 별도 | Feature Flag 기반 |
| 배포 프로세스 | 릴리스 승인 필요 | 자동 배포 |
| 장기 기능 개발 | develop 브랜치에서 통합 | Feature Flag로 관리 |
| 복잡도 | 높음 (브랜치 多) | 낮음 (브랜치 少) |

**의사결정 플로우**:

```
릴리스 주기가 정해져 있는가?
  ├─ Yes → Git Flow
  └─ No
       └─ 매일 배포 가능한 CI/CD가 있는가?
            ├─ Yes → Trunk-based
            └─ No  → Git Flow (안전한 기본값)
```

---

## 3. 공통 규칙

### 머지 전략

| 전략 | 언제 사용 | 결과 |
|------|----------|------|
| **Squash Merge** | feature → 통합 브랜치 | 여러 커밋을 하나로 합쳐 깔끔한 히스토리 |
| **Merge Commit** | release/hotfix → main | 머지 이력을 명시적으로 보존 |
| **Rebase** | 로컬에서 최신 코드 반영 | 선형 히스토리, push 전에만 사용 |

**규칙**:
- 이미 push된 커밋은 rebase하지 않는다.
- 공유 브랜치(main, develop)에서는 force push를 금지한다.
- Squash 시 최종 커밋 메시지는 커밋 컨벤션을 따른다.

### 브랜치 생명주기

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  생성    │ ──→ │  작업    │ ──→ │ 머지 요청 │ ──→ │  머지    │
│ (branch) │     │ (commit) │     │ (review) │     │ (merge)  │
└──────────┘     └──────────┘     └──────────┘     └────┬─────┘
                                                        │
                                                        ↓
                                                   ┌──────────┐
                                                   │  삭제    │
                                                   │ (delete) │
                                                   └──────────┘
```

- 머지된 브랜치는 **즉시 삭제**한다 (원격 + 로컬 모두).
- 삭제하지 않으면 브랜치 목록이 오염되어 관리가 어려워진다.

### 보호 브랜치 규칙

| 브랜치 | 직접 push | Force push | 머지 조건 |
|--------|----------|------------|----------|
| `main` | 금지 | 금지 | 리뷰 승인 + CI 통과 |
| `develop` | 금지 | 금지 | 리뷰 승인 + CI 통과 |
| `release/*` | 제한 | 금지 | 리뷰 승인 |
| `feature/*` 등 | 허용 | 허용 (본인만) | - |

---

## 4. FAQ

**Q: 이슈 트래커를 사용하지 않으면 issue-id를 어떻게 하나요?**
A: 생략한다. `feature/add-oauth-login` 형태로 prefix + description만 사용.

**Q: 하나의 이슈에 여러 브랜치가 필요하면?**
A: `feature/PROJ-123-add-login-ui`, `feature/PROJ-123-add-login-api`처럼 같은 issue-id에 다른 description을 붙인다.

**Q: develop 브랜치 없이 Git Flow를 쓸 수 있나요?**
A: 그것은 사실상 Trunk-based에 가깝다. Git Flow의 핵심은 develop을 통한 통합이므로, develop 없이 운영하려면 Trunk-based를 선택하는 것이 명확하다.

**Q: 브랜치 이름을 한국어로 써도 되나요?**
A: 기술적으로 가능하지만 권장하지 않는다. 인코딩 이슈, 도구 호환성 문제가 발생할 수 있다.

**Q: Squash Merge 시 원본 커밋 히스토리가 사라지는 것이 걱정됩니다.**
A: 머지 요청(MR/PR)에 원본 커밋이 기록으로 남는다. 필요 시 이력을 확인할 수 있다. 깔끔한 main/develop 히스토리의 가치가 더 크다.
