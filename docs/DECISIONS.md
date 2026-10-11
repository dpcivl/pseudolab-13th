# DECISIONS — Manufacturing AX Fieldbook

> 상태: **승인됨** · 승인일 2026-10-11 · 개정 1 (2026-10-11, 12절 참조)
> 이 문서는 구현 전에 합의한 결정만 담는다. 여기 없는 것은 구현하지 않는다. 변경은 사전 보고 후 승인을 받아 이 문서를 먼저 고친다.

## 1. 목적과 범위

- 프로젝트: 가짜연구소 13기 *Manufacturing AX Fieldbook*. 열간단조 공정의 에너지 효율 문제를 조사하고, KAMP 데이터로 AX 도입 사례(전처리 → 절감 요인 분석 → 솔루션 제안 → 절감 예측)를 만든다.
- 병행 목표: 서브에이전트 오케스트레이션(Agentic AI)을 실습하고, 프로젝트 관리 대시보드를 다른 프로젝트에도 갈아끼울 수 있게 만든다.

## 2. 레포와 공개 정책

| 항목 | 결정 |
|---|---|
| 레포 | `agent-workbench`(재사용 도구), `pseudolab-13th`(이 프로젝트). 로컬은 `~/dev/` 아래 형제 폴더 |
| 소유자·공개 | `dpcivl` 개인 계정, 둘 다 **Public** |
| 라이선스 | MIT |
| 커밋 이메일 | 현재 git 전역 설정 유지 |
| 데이터 | KAMP 원본·가공 데이터는 **커밋하지 않는다** (`data/` gitignore). README에 다운로드 안내만 둔다 |
| 실행 로그 | `.workbench/events*.jsonl`은 gitignore |
| 푸시 전 점검 | 두 레포에 `pre-push` 점검 (비밀값, 로컬 절대경로, 이메일 패턴). HEAD뿐 아니라 **푸시되는 커밋 범위의 추가분과 추가됐다 지워진 금지 파일**까지 검사한다 |
| 개인 자료 | 레포에 개인 평가·성향 자료나 그 해석을 넣지 않는다 |

## 3. 구조

- **agent-workbench** (재사용): Streamlit 대시보드, 데이터 계약(스키마·검증기), 공통 에이전트 3개, hook, 스킬 3개, 플러그인 매니페스트.
- **pseudolab-13th** (프로젝트): `CLAUDE.md`(도메인 맥락), 도메인 에이전트 6개, `.workbench/project.yaml`, `src/`, `notebooks/`, `docs/`, `data/`(gitignore).
- **계약 폴더**: 각 프로젝트의 `.workbench/` (`project.yaml`, `events-YYYY-MM.jsonl`, `reviews/`).
- **등록부**: `~/.agent-workbench/workspace.yaml` (프로젝트 id와 경로). 레포에는 `workspace.example.yaml`만 둔다.
- 데이터 모델은 `workspace > project`까지만. 층·루틴 개념은 두지 않는다.

## 4. 에이전트 (9개)

오케스트레이터는 **메인 세션**이다. 서브에이전트는 다른 서브에이전트를 호출할 수 없다.

| 에이전트 | 모델 | 도구 | 쓰기 범위 |
|---|---|---|---|
| `domain-researcher` | sonnet | Read, Write, Glob, Grep, WebSearch, WebFetch | `docs/research/**` |
| `ax-strategist` | opus | Read, Write, Glob, Grep | `docs/ax/**` |
| `data-engineer` | sonnet | Read, Write, Edit, Bash, Glob, Grep | `src/pipeline/**`, `data/processed/**`, `docs/data/**` |
| `energy-analyst` | opus | Read, Write, Edit, Bash, NotebookEdit, Glob, Grep | `notebooks/**`, `docs/analysis/**` |
| `ml-modeler` | opus | Read, Write, Edit, Bash, NotebookEdit, Glob, Grep | `src/models/**`, `models/**`, `docs/models/**`, `results/**` |
| `savings-simulator` | sonnet | Read, Write, Edit, Bash, Glob, Grep | `docs/simulation/**`, `results/**` |
| `pm-tracker` | haiku | Read, Edit, Write, Glob, Grep | `.workbench/project.yaml`(상태 필드만), `docs/reports/**` |
| `reviewer` | opus | Read, Glob, Grep, Write | `.workbench/reviews/**` |
| `fieldbook-writer` | sonnet | Read, Write, Glob, Grep | `docs/fieldbook/**` |

- 공통 금지: `data/raw/**`, `.workbench/events*.jsonl`(hook만 기록), `.git/**`.
- 쓰기 범위가 있는 에이전트도 OS 임시 폴더(세션 scratchpad 포함)에는 쓸 수 있다. 보호 경로 규칙은 그대로다.
- **hook은 옵트인이다.** 폴더에 `.workbench/project.yaml`이 있을 때만 이벤트 기록기와 쓰기 가드가 동작한다. 없으면 아무것도 하지 않는다(기록, 차단, `.workbench/` 생성 없음).
- 모델은 별칭으로 지정한다. 사용량 데이터(7-3)가 쌓이면 근거를 갖고 조정한다.
- 각 에이전트 명세는 같은 양식을 쓴다: `역할`, `읽는 것`, `남기는 것`, `완료 기준`, `하지 않는 것`, `도구`, `모델`, `쓰기 범위`.

### 에이전트별 역할·산출물·완료 기준

| 에이전트 | 하는 일 | 남기는 것 | 완료 기준 |
|---|---|---|---|
| `domain-researcher` | 열간단조 에너지 소비 구조·손실 요인 조사, KAMP 후보 데이터셋 정리 | `docs/research/*.md`, `kamp-candidates.md` | 모든 주장에 출처·접근일, 후보 3개 이상 비교표 |
| `ax-strategist` | AX 유스케이스 후보, 우선순위, KPI, 데이터 요구사항 | `docs/ax/use-cases.md`, `kpi.md` | 유스케이스별 필요 데이터·기대효과·리스크, 선택 근거 |
| `data-engineer` | 원본을 재현 가능한 파이프라인으로 정제 | `src/pipeline/*`, `data/processed/*`, 데이터 사전, 품질 리포트 | 재실행 시 동일 출력, 행 수 변화 로그 |
| `energy-analyst` | 절감 요인 분석, 솔루션 후보 제안 | 노트북, `factors.md`, `solution-proposals.md` | 효과 크기·불확실성 표기, 상관/인과 구분, 반대 증거 |
| `ml-modeler` | 에너지 소비 예측 모델 | `src/models/*`, 모델 카드, `metrics.json` | 베이스라인 대비 개선, 시간순 분할, 누수 점검 통과 |
| `savings-simulator` | 솔루션 적용 시 kWh·비용·CO₂ 절감 예측, 민감도 | `docs/simulation/*`, 시나리오 결과, `metrics.json` | 가정 표(전력 단가·배출계수와 출처), 구간 추정, 민감도 |
| `pm-tracker` | 이벤트·산출물 기반 상태 갱신, 주간 보고 | `project.yaml`(상태), `weekly-*.md` | 상태가 이벤트·산출물 존재 여부와 일치 |
| `reviewer` | 단계 종료 시 독립 검증 | `reviews/<phase>-<n>.json` | 판정, 지적 목록, 결론마다 반대 증거 1건 이상 |
| `fieldbook-writer` | 승인된 단계의 챕터 초안 | `docs/fieldbook/NN-*.md` | 수치마다 원본 링크, 승인된 내용만 반영 |

## 5. 스킬 (3개)와 단계 흐름

- `/project-init`: `.workbench/project.yaml` 생성, 등록부에 프로젝트 추가.
- `/run-phase <phase>`: 아래 흐름을 메인 세션이 실행.
- `/weekly-report`: `pm-tracker`가 사용하는 보고서 절차·형식.

```
/run-phase
 1. pm-tracker      단계를 doing으로 갱신
 2. 단계 에이전트    승인된 명세 안에서 중간 확인 없이 실행 (독립 작업은 병렬)
 3. reviewer        검증, reviews/*.json 기록
 4. pm-tracker      단계를 review로 갱신, gate_requested 기록
 5. 사용자           승인 또는 수정 요청
    승인 → pm-tracker가 done으로 갱신, gate_approved 기록, fieldbook-writer가 챕터 초안 작성
    수정 → change_requested / change_approved 기록 후 2로
```

| 단계 | 에이전트 | 승인 지점 |
|---|---|---|
| P1 조사 | domain-researcher | **데이터셋 선택**(사용자가 후보 중 선택·다운로드) |
| P2 AX 도입 설계 | ax-strategist | 단계 종료 |
| P3 데이터 전처리 | data-engineer | 단계 종료 |
| P4 요인 분석·솔루션 제안 | energy-analyst | 단계 종료 |
| P5 예측·절감 분석 | ml-modeler → savings-simulator | 단계 종료 (외부 가정값 목록을 `reviewer`가 정리해 함께 확인) |

## 6. 데이터 계약

### `project.yaml`

```yaml
project: {id, name, description}
agents:  [{id, role}]
phases:
  - id: p3-data
    name: 데이터 전처리
    status: todo | doing | review | blocked | done   # review = 사용자 승인 대기
    agents: [data-engineer]
    deliverables: [docs/data/data-dictionary.md, ...] # 완료 판단: 파일 존재 + 수정 시각
    gate: phase-end
    tasks: [{id, title, status: todo|doing|done, agent}]
review:                       # 선택. 공통 reviewer가 읽는 프로젝트별 검토 설정
  sources: [docs/research/**] # 선택. 근거 대조를 위해 추가로 읽을 경로(상대 경로 글롭)
  checks:                     # 선택. 공통 기준에 더해 적용할 도메인 점검
    - {id: leakage, text: "...", phases: [p4-analysis]}   # phases 생략 = 모든 단계
fieldbook:
  title: Manufacturing AX Fieldbook   # 선택. 챕터 머리말의 문서 제목, 생략하면 project.name
```

- 공통 에이전트(`reviewer`, `fieldbook-writer`)는 범용이며, 프로젝트 고유 내용은 `review`, `fieldbook`에만 둔다. 생략하면 공통 기준만 적용한다.
- 상태의 단일 기록자는 `pm-tracker`. 다른 에이전트는 이벤트만 남긴다.
- 태스크는 얇게: 담당자·마감·우선순위 없음.

### `events-YYYY-MM.jsonl` (추가 전용, 삭제 안 함)

| 필드 | 설명 |
|---|---|
| `v` | 스키마 버전 |
| `ts` | ISO 8601 시각 |
| `event` | `agent_start`, `agent_stop`, `agent_error`, `gate_requested`, `gate_approved`, `change_requested`, `change_approved` |
| `session_id`, `agent_id`, `agent` | hook이 제공. 오케스트레이터 이벤트의 `agent`는 `orchestrator` |
| `phase` | `project.yaml`에서 `doing`인 단계로 파생한다. `doing`이 없고 `review` 상태인 단계가 **하나뿐이면** 그 단계, 그 외에는 `null`. 대시보드는 `null` 이벤트를 같은 세션의 직전 게이트 이벤트의 단계로 추정해 귀속한다(원본은 바꾸지 않고 추정 표시). |
| `gate` | 게이트 이벤트에만 값 (예: `dataset-selection`) |
| `summary` | 응답 앞 200자, 절대경로 마스킹 (선택) |
| `rev` | 프로젝트 git 짧은 해시, 없으면 null |
| `usage` | `agent_stop`에만: `{model, input_tokens, output_tokens, cache_creation_input_tokens, cache_read_input_tokens}`, 추출 실패 시 null. 입력량의 대부분이 캐시 토큰이라 두 캐시 필드를 함께 기록한다 |

- 프롬프트·응답 원문 전체는 기록하지 않는다. 소요 시간은 시작·종료 이벤트를 `agent_id`로 짝지어 대시보드가 계산한다.
- 종료 이벤트가 없는 `agent_start`는 대시보드가 **정체**로 표시한다.

### `reviews/<phase>-<n>.json`

`verdict`(`pass` | `needs_change`), `findings[{category, severity, resolved}]`, `counter_evidence_count`. 커밋 여부는 구현 단계에서 정한다.

- `severity`: `high` | `medium` | `low`. `category`는 자유 문자열.
- `pass`는 `counter_evidence_count`가 1 이상이고 미해결(`resolved: false`) `high` 지적이 없을 때만 유효하다.
- 지적 상세 설명은 같은 폴더의 `<phase>-<n>.md` 요약에 둔다(JSON에는 상세 필드 없음).

### 산출물 배치 규칙

- `deliverables`에는 구체 경로나 글롭(`*`, `**`)을 쓸 수 있고, 대시보드가 글롭을 펼쳐 존재 여부와 수정 시각으로 완료를 판단한다.
- 확정한 파일 이름: `docs/research/energy-structure.md`, `docs/data/quality-report.md`, `docs/models/model-card.md`, `notebooks/*.ipynb`, `results/p5-forecast/scenario-*`.
- `results/p5-forecast/metrics.json`은 `ml-modeler`가 `model` 키, `savings-simulator`가 `savings` 키만 쓴다. KPI 카드는 이 두 키를 읽으며 별도 스키마는 두지 않는다.
- 리뷰(`.workbench/reviews/`)와 챕터(`docs/fieldbook/`)는 단계 산출물 목록에 넣지 않고, 대시보드가 각각 별도로 읽는다.

## 7. 운영 방식

1. 보고는 **결론·추천 → 근거 → 되돌릴 수 있는지** 순서. 결정 요청은 한 번에 3개 이하, 선택지는 3개 이하. 어조는 건조하고 직접적으로.
2. 승인된 명세 안에서는 에이전트가 중간 확인 없이 진행한다. 사용자는 승인 지점에서만 호출된다. 명세·범위·스키마·공개와 관련된 변경은 **사전 보고 후 승인**.
3. 에이전트가 맡는 것: 상태 기록, 보고서 포맷, 문서 초안, 반복 전처리. 사용자가 쥐는 것: 데이터셋 선택, 요인 해석, 솔루션 판단. `reviewer`는 결론마다 반대 증거를 붙인다.

## 8. 대시보드

- **Streamlit**, 로컬 전용. 실행: 등록부의 프로젝트 목록에서 선택.
- 화면: 개요(단계 진행판, "내가 해야 할 일"=`review` 상태), 에이전트 타임라인(정체 표시), 산출물 목록, 리뷰 현황, 운영 지표(소요 시간, 승인 대기, 재작업, 사용량), KPI 카드(`metrics.json`).
- 운영 지표(오케스트레이션)와 프로젝트 지표(모델 정확도, 절감량)를 구분한다.

## 9. 범위에서 제외한 것

Notion·Slack 등 MCP 연동, 픽셀 오피스형 시각화, 층·루틴 같은 비프로젝트 단위, 정적 스냅샷 공유, 프로젝트 레지스트리 UI 편집 기능, 풀 PM 모델(담당자·마감·우선순위). 실제 필요가 생길 때 별도 결정으로 다시 연다.

## 10. 알려진 한계

- **쓰기 범위 강제는 `Write`/`Edit`/`NotebookEdit`에 대해서만 완전하다.** (`Bash`의 알려진 사각지대 예: `cd data/raw && rm x`, `find ... -delete`, `git clean -fdx`, `python -c "os.remove(...)"`) `Bash`가 있는 에이전트(`data-engineer`, `energy-analyst`, `ml-modeler`, `savings-simulator`)는 셸로 경로 제한을 우회할 수 있다. 보완책: `Bash` 호출에서 보호 경로(`data/raw`, `.workbench/events`, `.git`)를 문자열로 감지해 차단하는 best-effort hook, 그리고 `reviewer`의 변경 파일 점검. `reviewer`에는 `Bash`를 주지 않는다.
- 표본이 작다(사용자 1명, 에이전트 9개). 운영 지표는 기술 통계 수준이다.
- `usage` 추출은 트랜스크립트 형식에 의존해 깨질 수 있다. 실패는 `null`로 두고 핵심 스키마에는 영향이 없게 한다.

## 11. 구현 전 검증 항목과 결과 (결정 아님, W3 실측 반영 2026-10-11)

| # | 항목 | 결과 | 구현에 미치는 영향 |
|---|---|---|---|
| 1 | 서브에이전트 실패가 hook으로 들어오는지 | **부분 확인.** 존재하지 않는 에이전트 유형을 호출하면 `PostToolUseFailure`(matcher `Agent`)가 발화하고 `agent_error`가 기록된다. 이때 `agent_id`는 없다(`null`). **서브에이전트가 실행 도중 실패하는 경우는 아직 확인하지 못했다** | `agent_error`의 `agent_id`를 `null` 허용으로 계약을 맞췄다(12절). 실행 중 실패는 정체 표시 폴백이 담당한다 |
| 2 | 트랜스크립트에서 모델명·토큰 추출 | **확인.** 라이브 세션에서 `agent_stop`의 `usage`(모델, 입력, 출력, 캐시 2종)가 기록됨. 같은 요청의 중복 항목은 `requestId`로 제거 | 비공개 형식이므로 실패 시 `null` 유지는 그대로 |
| 3 | 서브에이전트의 스킬 사전 로드 | **확인.** `pm-tracker`의 `skills: [weekly-report]`가 로드됨 | 사전 로드된 스킬이 요청에 없는 작업을 유발했다(주간 보고서를 쓰려 함). 스킬 앞머리에 적용 조건을 추가해 해소를 재시험으로 확인했다 |
| 4 | 플러그인 구조와 소비 방식 | **부분 확인.** `claude plugin validate`가 마켓플레이스와 `plugin.json`을 통과시켰고, `--plugin-dir`로 로드한 세션에서 `/agent-workbench:run-phase`, `agent-workbench:*` 에이전트, hook이 모두 동작했다. **마켓플레이스 경유 영구 설치는 시험하지 않았다** | 설치 문서(README)는 `--plugin-dir`을 검증된 방법으로, 마켓플레이스 설치를 미검증으로 표기한다 |
| 5 | `reviews/` 커밋 정책 | 미정 | 사용자 결정 |
| 6 | `phase` 귀속 | **문제 확인 후 해결(12-1 A안).** 승인 흐름에서 단계가 `doing`을 벗어난 뒤 기록되는 이벤트의 `phase`가 `null`이 되었다 | 파생 규칙에 `review` 단계 폴백과 대시보드 추정 귀속을 추가했다(12절 1.2) |

로컬 개발 중에는 `claude --plugin-dir <agent-workbench 경로>`로 플러그인을 로드한다(세션 한정, 설정 변경 없음).

## 12. 변경 이력

| 개정 | 날짜 | 변경 | 근거 |
|---|---|---|---|
| 1 | 2026-10-11 | `usage`에 `cache_creation_input_tokens`, `cache_read_input_tokens` 추가 (10-1) | 실제 트랜스크립트에서 `input_tokens`가 12인데 캐시 토큰이 약 50만이라 비용 분석이 불가능했다 |
| 1 | 2026-10-11 | 리뷰 규칙(`severity` 값, `pass` 조건)과 쓰기 가드의 임시 폴더 허용 (10-2) | 서브에이전트 보고의 미정 사항 확정, 범위 있는 에이전트와 없는 에이전트의 불일치 해소 |
| 1 | 2026-10-11 | 산출물 배치 규칙 확정 (10-3) | 서브에이전트가 임시로 정한 파일 이름과 `metrics.json` 키 구조의 확정 |
| 1 | 2026-10-11 | `pre-push` 점검을 푸시 범위까지 확대 (보강) | HEAD만 검사하면 공개 레포 이력에 남은 비밀값을 놓친다 |
| 1.1 | 2026-10-11 | `agent_error` 이벤트의 `agent_id`를 `null` 허용으로 변경 | 실제 세션에서 시작되지 못한 호출의 실패는 `agent_id`가 없음을 확인. 계약 문서(`contract.md`)는 이미 `agent_id`로 짝지을 수 없다고 적고 있었다 |
| 1.2 | 2026-10-11 | `phase` 파생 규칙: `doing` 우선, 없으면 `review`가 하나뿐일 때 그 단계, 그 외 `null`. 대시보드는 `null`을 같은 세션의 직전 게이트 단계로 추정 귀속(추정 표시) (12-1 A) | 라이브 세션에서 승인 후 이벤트의 `phase`가 `null`이 되어 단계별 집계에서 빠졌다 |
| 1.3 | 2026-10-11 | hook을 옵트인으로 변경: `.workbench/project.yaml`이 있는 폴더에서만 기록기·가드 동작 | 플러그인을 켠 세션이 workbench가 아닌 폴더에서도 `.workbench/`를 만들고 `write_scope` 없는 에이전트의 쓰기를 막는 문제(재사용 시 영향) |
| 1.4 | 2026-10-11 | `project.yaml`에 선택 항목 `review`(`sources`, `checks`)와 `fieldbook`(`title`) 추가. 공통 `reviewer`·`fieldbook-writer`를 범용으로 변경하고 프로젝트 고유 점검은 이 항목으로 이동 | 공통 에이전트에 이 프로젝트 전용 내용이 들어 있어 다른 프로젝트에 쓸 수 없었다. 플러그인 이름으로 호출되어 프로젝트 파일로 대체할 수도 없었다 |
| 1.4 | 2026-10-11 | 이름이 빈 에이전트(Claude Code 내부 보조 에이전트)의 이벤트는 기록하지 않음 (버그 수정) | 라이브 세션에서 빈 `agent`의 `agent_stop`이 시작 없이 쌓여 계약 검증에 실패했다. 명세는 `agent`를 비어 있지 않은 문자열로 정한다 |
