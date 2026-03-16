# EazyPlans

Claude Code 멀티세션 작업에서 플랜이 사라지는 문제를 해결하는 파일 기반 플랜 시스템.

MCP 불필요. 파일과 폴더 구조만으로 작동합니다.

---

## 문제

Claude Code Plan Mode의 결과는 컨텍스트에만 존재합니다.

- 세션이 길어지면 컨텍스트 압축으로 플랜이 사라짐
- 다음 세션에서 "왜 이렇게 설계했지?"를 알 수 없음
- 팀 협업 시 다른 사람이 플랜을 볼 방법이 없음
- Plan Mode 진입 전 리서치를 강제하는 장치가 없어 outdated 정보로 플랜 수립

[Claude Code GitHub에는 이 문제를 요청한 이슈가 7건 이상](https://github.com/anthropics/claude-code/issues/29445) 있습니다. 공식 해결책은 아직 없습니다.

---

## 해결

```
plans/
├── active/     ← 진행 중인 플랜
├── backlog/    ← 대기 중
└── done/       ← 완료 (의사결정 히스토리)
```

각 작업마다 폴더 생성:

```
plans/active/YYYYMMDD-작업명/
├── research.md    ← Plan Mode 전 선행 리서치 (필수)
└── overview.md    ← 단계별 체크리스트 + [보고] 마커
```

`plans/` 폴더를 git에 커밋하면 팀 전체가 같은 플랜을 공유하고, 완료된 플랜은 `done/`에 영구 보존됩니다.

---

## 핵심 메커니즘 3개

### 1. research.md 선행 필수

overview.md 작성 전에 반드시 리서치를 먼저 합니다.

- WebSearch/Context7로 최신 정보 수집
- 라이브러리 버전, 베스트 프랙티스, 기존 코드 충돌 여부 확인
- 기억이 아닌 도구로 수집

이 단계가 없으면 AI가 학습 당시 데이터 기준으로 플랜을 만듭니다.

예외: 외부 의존성 없는 내부 리팩토링은 research.md 생략 가능.

### 2. [보고] 마커

overview.md 단계 옆에 붙이면 그 단계 완료 후 결과를 보고하고 승인을 기다립니다.

```markdown
## 단계
1. 리서치 → [보고]     ← 여기서 멈추고 보고
2. 설계 → [보고]       ← 사용자 확인 후 진행
3. 구현                ← 자율 진행
4. 검증 → [보고]       ← 최종 결과 보고
```

`[보고]` 없는 단계는 자율 진행. 어느 단계에서 개입할지를 플랜 작성 시점에 선언합니다.

### 3. GATE -1 (Plan → Build 전환 검증)

ExitPlanMode 직전 자동 실행. 5항목 중 3개 미만이면 플랜 보완을 요청합니다.

```
[ ] 근본 원인이 명확한가?
[ ] 영향받는 파일을 식별했는가?
[ ] 완료 기준이 구체적인가?
[ ] 기존 코드와 충돌을 확인했는가?
[ ] 검증 방법이 있는가?
```

---

## 설치

```bash
# 1. plans/ 폴더 복사
cp -r plans/ /your-project/plans/

# 2. CLAUDE.md 병합
cat CLAUDE.md >> ~/.claude/CLAUDE.md
# 또는 프로젝트별 적용
cat CLAUDE.md >> /your-project/.claude/CLAUDE.md

# 3. git에 포함
echo "plans/done/" >> .gitignore  # done은 제외할 경우 (선택)
git add plans/
```

---

## 팀 협업

```bash
# 플랜 작성 후
git add plans/active/20260316-새기능/
git commit -m "plan: 새기능 플랜 추가"
git push

# 팀원이 플랜 확인
git pull
cat plans/active/20260316-새기능/overview.md
```

- PR 시 overview.md가 변경 이유의 근거 문서로 활용
- done/으로 이동한 플랜은 의사결정 히스토리로 영구 보존
- 새 팀원이 과거 결정 맥락을 파악하는 온보딩 자료로 활용 가능

---

## 다른 솔루션과 비교

| | EazyPlans | 공식 Plan Mode | SuperClaude | planning-with-files |
|--|-----------|--------------|-------------|-------------------|
| 플랜 영속성 | 파일 (git) | 컨텍스트만 | MCP 기반 | 파일 |
| 리서치 강제 | 필수 | 없음 | 없음 | 없음 |
| 생명주기 | active/backlog/done | 없음 | 계층적 task | 미정의 |
| [보고] 체크포인트 | 있음 | 없음 | 없음 | 없음 |
| GATE -1 검증 | 있음 | 없음 | 없음 | 없음 |
| MCP 의존성 | 없음 | 없음 | 있음 | 없음 |
| 팀 협업 | git 기반 | 불가 | 제한적 | 미정의 |

---

## eazy-claude-harness와 함께 사용

[eazy-claude-harness](https://github.com/Giwoong-ryu/eazy-claude-harness)와 함께 쓰면 전체 개발 흐름을 커버합니다.

```
[EazyPlans]  Plan Mode → research.md → overview.md → GATE -1
[eazy-claude-harness]  GATE → DMAD → Code → sim → Check → Fix
```

---

## 참고

- [Claude Code GitHub Issue: Plan Manager](https://github.com/anthropics/claude-code/issues/29445)
- [Claude Code GitHub Issue: Project-level plans](https://github.com/anthropics/claude-code/issues/21042)
- [Anthropic: Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
