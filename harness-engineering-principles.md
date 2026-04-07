# 하네스 엔지니어링 원칙서

> **Coding Agent = AI Model + Harness**
>
> 하네스 엔지니어링은 AI 모델 자체가 아닌, 모델을 감싸는 설정/도구/오케스트레이션 시스템을 최적화하는 엔지니어링이다.
> 더 똑똑한 모델을 기다리는 것이 아니라, 지금 가진 모델이 최대 성능을 내도록 환경을 설계한다.

> *"The space of interesting harness combinations doesn't shrink as models improve. Instead, it moves."*
> — Anthropic Engineering Blog

---

## 1. 핵심 공식

```
┌─────────────────────────────────────────────┐
│              AI Coding Agent                │
│                                             │
│   ┌───────────┐     ┌───────────────────┐   │
│   │  AI Model │  +  │     Harness       │   │
│   │ (Claude)  │     │                   │   │
│   │           │     │  CLAUDE.md        │   │
│   │  변경     │     │  Skills           │   │
│   │  불가능   │     │  Hooks            │   │
│   │           │     │  MCP Servers      │   │
│   │           │     │  AGENTS.md        │   │
│   │           │     │  Memory           │   │
│   └───────────┘     │                   │   │
│                     │  ← 엔지니어링 대상  │   │
│                     └───────────────────┘   │
└─────────────────────────────────────────────┘
```

모델은 바꿀 수 없다. **하네스는 바꿀 수 있다.** 하네스 엔지니어링의 모든 노력은 오른쪽에 집중된다.

---

## 2. 하네스 구성요소

| 구성요소 | 역할 | 로딩 시점 |
|----------|------|-----------|
| **CLAUDE.md** | 프로젝트 컨텍스트, 규칙, 컨벤션 주입 | 세션 시작 시 자동 |
| **Skills** | 재사용 가능한 지식/작업 모듈 | 트리거 시 온디맨드 |
| **Hooks** | 라이프사이클 이벤트에 자동화 로직 삽입 | 이벤트 발생 시 자동 |
| **MCP Servers** | 외부 시스템 연동 (Git, Issue Tracker, 모니터링 등) | 도구 호출 시 |
| **AGENTS.md** | 서브에이전트 역할/도구/지시사항 정의 | 에이전트 생성 시 |
| **Memory** | 세션 간 지속되는 학습/컨텍스트 | 세션 시작 시 자동 |

### 계층 구조

```
~/.claude/                    ← 사용자 글로벌 설정
  settings.json               ← Hooks, 권한, 플러그인
  skills/                     ← 글로벌 Skills
  CLAUDE.md                   ← 모든 프로젝트 공통 규칙

project-root/                 ← 프로젝트 레벨
  CLAUDE.md                   ← 이 프로젝트의 컨텍스트
  AGENTS.md                   ← 서브에이전트 정의
  .claude/
    settings.json             ← 프로젝트별 Hooks/권한
    settings.local.json       ← 개인 로컬 설정 (gitignore)

  src/module-a/
    CLAUDE.md                 ← 모듈 레벨 상세 (선택)
```

**원칙**: 위에서 아래로 점점 구체적. 상위는 "무엇을", 하위는 "어떻게".

---

## 3. 6대 원칙

### 원칙 1: 컨텍스트 윈도우 관리

> 모델이 매 세션 "발견"해야 할 것을 최소화한다. 핵심 컨텍스트는 선제 주입한다.

**문제**: CLAUDE.md가 없으면 에이전트는 매 세션마다 프로젝트 구조, 빌드 명령, 컨벤션을 탐색하며 컨텍스트를 낭비한다.

> *"Models tend to lose coherence on lengthy tasks as the context window fills."* — Anthropic

**실천**:

- CLAUDE.md에 기술 스택, 빌드 명령, 디렉토리 구조, 컨벤션을 명시
- 각 CLAUDE.md는 **200줄 이내** 유지 (그 이상은 하위 계층으로 분리)
- "에이전트가 처음 2개 메시지 안에 작업을 시작할 수 있는가?"를 기준으로 검증

**안티패턴**:

- CLAUDE.md에 모든 정보를 넣어 500줄 초과 → 컨텍스트 자체가 부담
- CLAUDE.md 없이 매번 구두로 설명 → 세션마다 같은 낭비 반복

---

### 원칙 2: 점진적 정보 공개 (Progressive Disclosure)

> 필요한 정보를 필요한 시점에 로딩한다. 모든 것을 한번에 주입하지 않는다.

**문제**: 모든 지식을 CLAUDE.md에 넣으면 컨텍스트가 폭발한다. 모든 도구를 항상 로딩하면 선택지가 과다해진다.

**실천**:

- **CLAUDE.md 계층화**: 루트(아키텍처 개요) → 모듈(상세 규칙) → 필요 시만 하위 로딩
- **Skills 분리**: 전문 지식은 Skill로 캡슐화하여 트리거 시에만 로딩
- **references/ 활용**: Skill 내부에 상세 레퍼런스를 별도 파일로 분리
- **MCP 지연 로딩**: 도구 이름만 컨텍스트에 노출, 스키마는 호출 시 로딩 (Claude Code 기본 동작)

**구조 예시**:

```
Skill: create-api-endpoint
  SKILL.md          ← 30줄: 언제/어떻게 트리거하는지
  references/
    patterns.md     ← 200줄: 구체적 코드 패턴 (트리거 후 로딩)
    examples.md     ← 실제 예시 코드
```

---

### 원칙 3: 실패-수정 루프 (Fail & Fix Loop)

> 실패를 빨리 감지하고, 자동으로 교정하는 피드백 루프를 설계한다.

**문제**: 에이전트가 파일을 수정한 후 검증 없이 다음 단계로 넘어가면, 오류가 누적되어 나중에 대규모 수정이 필요하다.

**실천**:

- **PostToolUse Hook**: 파일 수정 후 자동으로 컴파일/타입체크/린트 실행
- **PreToolUse Hook**: 민감 파일(환경변수, 프로덕션 설정) 수정 차단
- **Back-pressure**: 테스트와 타입 체크를 통해 에이전트가 자기 작업을 검증

**Hook 설정 패턴**:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "보호할 파일 패턴 매칭 → 차단 또는 경고"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "수정된 파일 확장자 감지 → 해당 언어 컴파일/타입체크 실행"
          }
        ]
      }
    ]
  }
}
```

**Hook 이벤트 종류**:

| 이벤트 | 시점 | 활용 |
|--------|------|------|
| SessionStart | 세션 시작 | 환경 초기화, 상태 체크 |
| UserPromptSubmit | 사용자 입력 후 | 입력 보강, 컨텍스트 추가 |
| PreToolUse | 도구 실행 전 | 위험 작업 차단, 승인 요청 |
| PostToolUse | 도구 실행 후 | 자동 검증, 포맷팅 |
| Stop | 에이전트 응답 완료 | 작업 요약, 상태 기록 |

---

### 원칙 4: 격리와 병렬성 (Isolation & Parallelism)

> 작업자와 평가자를 분리한다. 독립된 작업은 병렬로 실행한다.

**문제**: 에이전트가 자기 작업을 스스로 평가하면 관대해진다.

> *"When asked to evaluate work they've produced, agents tend to respond by confidently praising the work—even when the quality is obviously mediocre."* — Anthropic

**GAN 패턴 (Planner → Generator → Evaluator)**:

```
[Planner]     요구사항 → 상세 스펙 확장
    ↓ (파일로 전달)
[Generator]   스펙 기반 구현
    ↓ (파일로 전달)
[Evaluator]   독립 검증 → 합격/불합격 판정
    ↓ (불합격 시 Generator로 회귀)
```

**실천**:

- **AGENTS.md로 서브에이전트 정의** — 역할, 모델, 허용 도구, 지시사항 명시
- **작업자 ≠ 평가자** — 코드 작성 에이전트와 리뷰 에이전트를 분리
- **파일 기반 통신** — 에이전트 간 직접 대화 대신 파일(progress.txt, review-result.md)로 소통
- **Git Worktree** — 독립된 브랜치 작업을 물리적으로 격리

**AGENTS.md 기본 구조**:

```markdown
# Agents

## reviewer
- 역할: Evaluator — 코드 변경사항 독립 리뷰
- 모델: sonnet (빠른 피드백)
- 도구: Read, Grep, Glob (읽기 전용)
- 지시사항:
  - 프로젝트 컨벤션 준수 여부 확인
  - 보안 취약점 탐지
  - 누락된 에러 처리 확인

## test-writer
- 역할: Generator — 변경사항 기반 테스트 생성
- 모델: sonnet
- 도구: Read, Grep, Glob, Write
- 지시사항:
  - 정상/에러/경계값 케이스 작성
  - 기존 테스트 패턴 준수
```

**Evaluator 캘리브레이션**:

QA 에이전트는 기본적으로 관대하다. Few-shot 예시로 판단 기준을 정렬해야 한다:

> *"I watched it identify legitimate issues, then talk itself into deciding they weren't a big deal and approve the work anyway."* — Anthropic

합격/불합격 기준을 구체적으로 명시하고, 실제 사례를 포함한다.

---

### 원칙 5: 피드백 메커니즘 (Feedback Mechanisms)

> 시스템이 성공과 실패로부터 학습하는 채널을 만든다.

**문제**: 피드백 없이는 에이전트가 같은 실수를 반복하고, 좋은 패턴은 잊어버린다.

**실천**:

- **Memory 시스템 활용** — 세션 간 지속되는 학습 (user, feedback, project, reference 타입)
- **작업 요약 기록** — 세션 종료 시 변경 파일, 사용 스킬, 발견된 이슈 기록
- **피드백 메모리 즉시 저장** — 사용자 교정("그렇게 하지 마")과 확인("맞아, 그렇게 해") 모두 저장
- **Stop Hook** — 에이전트 응답 완료 시 자동으로 상태 기록

**피드백 루프 구조**:

```
작업 수행 → 결과 확인 → 패턴 발견
                           ↓
              Memory에 저장 (feedback 타입)
                           ↓
              다음 세션에서 자동 반영
```

---

### 원칙 6: 수정보다 설정 (Configuration over Modification)

> 매번 다르게 말하지 않는다. 반복되는 지시는 설정 파일로 이관한다.

**문제**: "항상 이렇게 해줘"를 매 세션 반복하는 것은 컨텍스트 낭비이자 실수 유발.

> *"Find the simplest solution possible, and only increase complexity when needed."* — Anthropic

**실천**:

- **행동 규칙 → CLAUDE.md**: "커밋은 Conventional Commits로", "SQL 실행 전 검증" 등
- **자동 검증 → Hooks**: "수정 후 포맷팅", "프로덕션 설정 보호" 등
- **전문 작업 → Skills**: 반복되는 복잡한 워크플로를 Skill로 캡슐화
- **외부 연동 → MCP**: API 호출, 이슈 트래커 조회 등을 MCP 서버로 표준화

**이관 판단 기준**:

```
"이 지시를 2번 이상 반복했는가?"
   ├─ Yes → 설정 파일로 이관
   │    ├─ 행동 규칙이면 → CLAUDE.md
   │    ├─ 자동 실행이면 → Hook
   │    ├─ 복잡한 워크플로면 → Skill
   │    └─ 외부 시스템이면 → MCP
   └─ No  → 아직 구두 지시로 충분
```

---

## 4. 하네스 진화 원칙

> *"Every component in a harness encodes an assumption about what the model can't do on its own, and those assumptions are worth stress testing."* — Anthropic

하네스는 고정이 아니다. 모델이 발전하면 함께 진화해야 한다.

### V1 → V2 사례 (Anthropic)

| 구성요소 | V1 (Opus 4.5) | V2 (Opus 4.6) | 판단 근거 |
|----------|--------------|--------------|-----------|
| Sprint 분해 | 필수 | **제거** | 장시간 일관성 유지 가능 |
| Context Reset | 필수 | **제거** | 컨텍스트 윈도우 활용 개선 |
| 매 Sprint QA | 필수 | **최종 1회** | 중간 검증 불필요 |
| Planner | 유지 | **유지** | 스코프 부족 방지에 여전히 필요 |
| Evaluator | 유지 | **유지** | 자기 평가 편향은 모델과 무관 |

### 재평가 규칙

- **제거할 때**: 하나씩. 제거 후 품질 변화를 측정한다.
- **추가할 때**: 필요가 증명된 후에만. 예방적 추가를 하지 않는다.
- **주기**: 모델 메이저 업그레이드 시, 또는 반복 실패 패턴 발견 시.

---

## 5. 구현 로드맵

### Phase 1: 기반 — CLAUDE.md + 기본 Hook

> 가장 높은 ROI. CLAUDE.md 하나만으로 매 세션 탐색 시간이 크게 줄어든다.

- [ ] 루트 CLAUDE.md 작성 (기술 스택, 빌드 명령, 컨벤션)
- [ ] PreToolUse Hook: 민감 파일 보호 (.env, 프로덕션 설정)
- [ ] 프로젝트 디렉토리 구조 정리

### Phase 2: 자동화 — Fail & Fix 루프

> 수정 즉시 검증하는 피드백 루프 구축

- [ ] PostToolUse Hook: 언어별 컴파일/타입체크 자동 실행
- [ ] AGENTS.md: reviewer 서브에이전트 정의
- [ ] 첫 번째 Skill 작성 (가장 반복적인 워크플로 대상)

### Phase 3: 확장 — 에이전트 협업 + 피드백

> GAN 패턴 적용, 학습 루프 구축

- [ ] Evaluator 에이전트 캘리브레이션 (few-shot 합격/불합격 예시)
- [ ] 파일 기반 에이전트 간 통신 패턴 적용
- [ ] Memory 피드백 루프 가동
- [ ] MCP 서버 연동 (프로젝트에 필요한 외부 시스템)

### Phase 4: 최적화 — 측정과 진화

> 불필요한 것을 제거하고, 새로운 가능성을 탐색

- [ ] 각 하네스 구성요소의 실효성 평가
- [ ] 컨텍스트 예산 모니터링 (CLAUDE.md 200줄 이내)
- [ ] 모델 업그레이드 시 하네스 재평가 수행
- [ ] 세션 간 반복 지시 제로 달성

---

## 6. 성숙도 체크리스트

### Level 1 — 기초 (Foundation)

- [ ] CLAUDE.md 존재하고, 기술 스택/빌드 명령/컨벤션이 명시됨
- [ ] PreToolUse Hook: 민감 파일 보호 설정됨
- [ ] 에이전트가 첫 2개 메시지 안에 작업 시작 가능

### Level 2 — 자동화 (Automation)

- [ ] PostToolUse Hook: 수정 후 자동 검증 동작
- [ ] AGENTS.md에 1개 이상 서브에이전트 정의됨
- [ ] 1개 이상 커스텀 Skill 작성됨
- [ ] 반복 지시 3개 이상 설정 파일로 이관됨

### Level 3 — 협업 (Collaboration)

- [ ] 작업자/평가자 에이전트 분리 동작
- [ ] 파일 기반 에이전트 간 통신 적용됨
- [ ] Evaluator few-shot 캘리브레이션 완료
- [ ] Memory 피드백 루프 가동 중

### Level 4 — 최적화 (Mastery)

- [ ] 세션 간 반복 지시 제로
- [ ] 각 CLAUDE.md 200줄 이내
- [ ] 하네스 재평가 수행 기록 있음
- [ ] 6대 원칙 모두 측정 가능하게 구현됨

---

## 7. 빠른 참조

| 원칙 | 핵심 질문 | 주요 도구 |
|------|----------|----------|
| 컨텍스트 윈도우 관리 | "에이전트가 바로 작업을 시작할 수 있는가?" | CLAUDE.md |
| 점진적 정보 공개 | "지금 불필요한 정보가 로딩되어 있지 않은가?" | Skills, references/ |
| 실패-수정 루프 | "수정 직후 오류를 감지할 수 있는가?" | Hooks (PostToolUse) |
| 격리와 병렬성 | "작업자와 평가자가 분리되어 있는가?" | AGENTS.md, Worktree |
| 피드백 메커니즘 | "같은 실수를 다시 하지 않는가?" | Memory, Stop Hook |
| 수정보다 설정 | "이 지시를 2번 이상 반복한 적 있는가?" | CLAUDE.md, Hooks, Skills |

---

## 부록: 서비스 특화 확장 가이드

이 원칙서는 범용 기반이다. 서비스가 결정되면 다음을 점진적으로 추가한다:

```
Phase A: 서비스 컨텍스트 추가
  └─ CLAUDE.md에 도메인 용어, DB 스키마 특이사항, API 규격 추가

Phase B: 워크플로 Skill 작성
  └─ 해당 서비스의 반복 작업을 Skill로 캡슐화
     (예: MR 생성, 배포 트리거, 로그 조회)

Phase C: 외부 시스템 연동
  └─ MCP 서버로 이슈 트래커, CI/CD, 모니터링 연동

Phase D: 도메인 전문 에이전트
  └─ AGENTS.md에 해당 서비스 전문 리뷰어/검증자 추가
```

**순서 원칙**: 공통(이 문서) → 서비스 컨텍스트 → 워크플로 자동화 → 외부 연동 → 전문 에이전트
