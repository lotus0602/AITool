# 커밋 컨벤션

> [Conventional Commits](https://www.conventionalcommits.org/) 기반 커밋 메시지 규칙.
> 일관된 커밋 히스토리를 유지하고, 자동화 도구(changelog, SemVer)와의 연동을 가능하게 한다.

---

## CLAUDE.md 삽입용

아래 블록을 프로젝트 CLAUDE.md에 복사하여 사용한다.

```
## 커밋 컨벤션

형식: <type>(<scope>): <subject>

type:
  feat     — 새 기능
  fix      — 버그 수정
  refactor — 기능 변경 없는 코드 개선
  docs     — 문서만 변경
  test     — 테스트 추가/수정
  chore    — 빌드, 설정, 패키지 등 기타
  style    — 포맷팅, 세미콜론 등 코드 의미 변경 없음
  perf     — 성능 개선
  ci       — CI 설정 변경
  build    — 빌드 시스템, 외부 의존성 변경

규칙:
- subject: 명령형, 50자 이내, 소문자 시작, 마침표 없음
- body: 선택. "왜" 중심, 72자 줄바꿈
- footer: 선택. BREAKING CHANGE: 또는 이슈 참조
- 금지: 과거형(added), 대문자 시작(Add), 마침표(add feature.)
```

---

## 1. 커밋 메시지 형식

### 기본 구조

```
<type>(<scope>): <subject>
                                    ← 빈 줄
<body>
                                    ← 빈 줄
<footer>
```

- `type`: 필수. 변경의 성격을 나타내는 접두사.
- `scope`: 선택. 변경 대상 모듈이나 기능.
- `subject`: 필수. 변경 내용의 한 줄 요약.
- `body`: 선택. 변경의 동기와 맥락.
- `footer`: 선택. Breaking Change 고지 또는 이슈 참조.

### type 목록

| type | 설명 | SemVer | 예시 |
|------|------|--------|------|
| `feat` | 새로운 기능 추가 | minor | `feat(auth): add OAuth2 login` |
| `fix` | 버그 수정 | patch | `fix(cart): resolve null pointer on empty cart` |
| `refactor` | 기능 변경 없는 코드 개선 | - | `refactor(api): extract validation logic` |
| `docs` | 문서만 변경 | - | `docs: update API usage guide` |
| `test` | 테스트 추가 또는 수정 | - | `test(auth): add login failure cases` |
| `chore` | 빌드, 설정, 패키지 등 기타 | - | `chore: update dependency versions` |
| `style` | 포맷팅, 세미콜론 등 (코드 의미 변경 없음) | - | `style: fix indentation in config` |
| `perf` | 성능 개선 | patch | `perf(query): add index for user lookup` |
| `ci` | CI 설정 변경 | - | `ci: add lint stage to pipeline` |
| `build` | 빌드 시스템, 외부 의존성 변경 | - | `build: upgrade webpack to v5` |

### scope 규칙

- 프로젝트 내 모듈명, 기능명, 레이어명 등을 사용한다.
- 중첩하지 않는다: `auth` (O), `auth/login` (X)
- 프로젝트 내에서 일관된 이름을 사용한다.
- 여러 모듈에 걸치면 생략하거나 가장 핵심 모듈을 지정한다.

```
feat(auth): add password reset flow       ← 모듈 scope
fix(api): handle timeout on slow networks  ← 레이어 scope
docs: update README                        ← scope 생략
```

### subject 작성 규칙

| 규칙 | 좋은 예 | 나쁜 예 |
|------|---------|---------|
| 명령형 사용 | `add login validation` | `added login validation` |
| 소문자 시작 | `fix null pointer` | `Fix null pointer` |
| 마침표 없음 | `update error message` | `update error message.` |
| 50자 이내 | `add retry logic for API calls` | `add retry logic for API calls when the server returns a 503 error` |
| "무엇을 했는지" 요약 | `remove deprecated endpoint` | `I removed the old endpoint that was deprecated` |

### body 작성 규칙

- **"왜" 중심**: 코드 diff로 알 수 없는 동기와 맥락을 적는다.
- 72자에서 줄바꿈한다.
- subject와 빈 줄로 구분한다.

```
fix(cart): resolve race condition on concurrent updates

이전 구현은 optimistic locking 없이 직접 업데이트하여,
동시 요청 시 마지막 쓰기가 이전 변경을 덮어쓰는 문제가 있었다.

SELECT FOR UPDATE로 행 잠금을 추가하여 해결한다.
```

### footer 작성 규칙

**Breaking Change 고지**:

```
feat(api): change response format to JSON:API

BREAKING CHANGE: 응답 포맷이 기존 커스텀 JSON에서 JSON:API 스펙으로 변경됨.
v1 클라이언트는 응답 파서를 업데이트해야 함.
```

**이슈 참조**:

```
fix(auth): prevent session fixation attack

Closes #234
Refs #198, #201
```

---

## 2. 좋은 예시 / 나쁜 예시

| 좋은 커밋 | 나쁜 커밋 | 이유 |
|-----------|-----------|------|
| `feat(search): add fuzzy matching` | `update search` | type 없음, 무엇을 했는지 불명확 |
| `fix(auth): validate token expiry before use` | `fix bug` | 어떤 버그인지 알 수 없음 |
| `refactor(db): extract connection pooling logic` | `refactoring` | subject 없음, scope 없음 |
| `docs: add deployment guide for staging` | `docs update` | 무엇을 업데이트했는지 불명확 |
| `chore: bump eslint from 8.x to 9.x` | `chore: stuff` | 의미 없는 subject |
| `feat(api)!: change auth header format` | `big change` | type, scope, Breaking Change 표시 모두 없음 |

---

## 3. 자동화 연동

### commitlint

커밋 메시지가 컨벤션을 따르는지 자동 검증한다.

```bash
# 설치
npm install --save-dev @commitlint/cli @commitlint/config-conventional

# 설정 (commitlint.config.js)
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'subject-case': [2, 'never', ['start-case', 'pascal-case', 'upper-case']],
    'header-max-length': [2, 'always', 72],
  },
};
```

### husky (Git Hook)

커밋 시점에 commitlint를 자동 실행한다.

```bash
# 설치 및 설정
npm install --save-dev husky
npx husky init

# commit-msg hook 추가
echo 'npx --no -- commitlint --edit "$1"' > .husky/commit-msg
```

### Changelog 자동 생성

Conventional Commits를 따르면 changelog를 자동 생성할 수 있다.

```bash
# standard-version 또는 release-please 등 활용
npx standard-version
```

| type | changelog 섹션 |
|------|----------------|
| `feat` | Features |
| `fix` | Bug Fixes |
| `perf` | Performance |
| `BREAKING CHANGE` | Breaking Changes |
| 그 외 | 포함되지 않음 (설정으로 변경 가능) |

---

## 4. FAQ

**Q: 한국어로 subject를 작성해도 되나요?**
A: 프로젝트 내에서 통일하면 된다. 다만 자동화 도구(changelog, SemVer)와의 호환성을 고려하면 영문 권장.

**Q: scope는 반드시 써야 하나요?**
A: 선택이다. 다만 일관성이 중요하다. 팀에서 scope 목록을 정의해두면 혼란을 줄일 수 있다.

**Q: 하나의 커밋에 여러 type이 해당되면?**
A: 커밋을 분리한다. 하나의 커밋은 하나의 논리적 변경을 담아야 한다. `feat`과 `fix`가 섞이면 각각 별도 커밋.

**Q: WIP(Work In Progress, 작업 진행 중) 커밋은 어떻게 하나요?**
A: 로컬에서는 자유롭게 커밋하되, 머지 전 squash하여 컨벤션에 맞는 커밋으로 정리한다.

**Q: Breaking Change는 어떤 type에서든 가능한가요?**
A: 그렇다. `feat!:`, `fix!:`, `refactor!:` 등 어떤 type이든 `!` 접미사 또는 footer의 `BREAKING CHANGE:`로 고지할 수 있다.
