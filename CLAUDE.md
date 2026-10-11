# CLAUDE.md — Manufacturing AX Fieldbook

가짜연구소 13기 프로젝트. 이 문서는 도메인 맥락과 운영 규칙의 요약이다. 결정의 원문은 [docs/DECISIONS.md](docs/DECISIONS.md)이고, 충돌하면 그쪽이 우선한다.

## 목적

- 열간단조 공정의 에너지 효율 문제를 조사하고, KAMP 데이터로 AX 도입 사례를 만든다: 전처리 → 절감 요인 분석 → 솔루션 제안 → 절감 예측.
- 병행 목표: 서브에이전트 오케스트레이션 실습, 재사용 가능한 프로젝트 관리 대시보드(`agent-workbench`).

## 레포 구조

```
CLAUDE.md                 이 문서
docs/DECISIONS.md         승인된 결정 (명세)
docs/research|ax|data|analysis|models|simulation|fieldbook|reports/   단계별 문서
src/pipeline, src/models  코드
notebooks/                분석 노트북
data/raw, data/processed  데이터 (커밋 안 함)
results/<phase-id>/       지표·시나리오 결과 (metrics.json은 p5만)
.claude/agents/           도메인 에이전트 6개
.workbench/               project.yaml, events*.jsonl(gitignore), reviews/
```

## 에이전트와 단계

- 오케스트레이터는 메인 세션이다. 서브에이전트는 다른 서브에이전트를 호출하지 못한다.
- 도메인 에이전트 6개는 이 레포의 `.claude/agents/`에 있다. 공통 에이전트 3개(`pm-tracker`, `reviewer`, `fieldbook-writer`)와 스킬·hook은 `agent-workbench` 플러그인이 제공한다.
- 에이전트별 모델·도구·쓰기 범위·완료 기준: DECISIONS 섹션 4.
- 단계: p1-research → p2-ax-design → p3-data → p4-analysis → p5-forecast. 단계별 에이전트와 승인 지점: DECISIONS 섹션 5, 상태는 `.workbench/project.yaml`.
- P1의 승인 지점은 **데이터셋 선택**이다. 사용자가 후보 중 골라 KAMP에서 직접 내려받아 `data/raw`에 둔다.

## 실행 방법

```
/agent-workbench:run-phase <phase-id>     예: /agent-workbench:run-phase p1-research
```

흐름: 상태 doing → 단계 에이전트 실행 → `reviewer` 검증 → 상태 review → 사용자 승인 → done → 챕터 초안. 상세는 DECISIONS 섹션 5.

## 운영 규칙 (DECISIONS 섹션 7)

1. 보고 순서: 결론·추천 → 근거 → 되돌릴 수 있는지. 결정 요청은 한 번에 3개 이하, 선택지도 3개 이하. 어조는 건조하고 직접적으로.
2. 승인된 명세 안에서는 중간 확인 없이 진행한다. 사용자는 승인 지점에서만 호출된다. 명세·범위·스키마·공개 관련 변경은 사전 보고 후 승인을 받고, `docs/DECISIONS.md`를 먼저 고친다.
3. 에이전트는 상태 기록, 보고서 포맷, 문서 초안, 반복 전처리를 맡는다. 데이터셋 선택, 요인 해석, 솔루션 판단은 사용자가 쥔다. `reviewer`는 결론마다 반대 증거를 붙인다.
4. 명세가 부족하면 추측하지 말고 멈춰서 무엇이 부족한지 보고한다.
5. 상태의 단일 기록자는 `pm-tracker`. 다른 에이전트는 상태를 직접 고치지 않는다.

## 데이터 정책

- KAMP 원본·가공 데이터는 커밋하지 않는다 (`data/`는 gitignore, `data/README.md`만 예외).
- `data/raw`는 불변이다. 어떤 에이전트도 쓰지 않는다. 가공은 `data/processed`에만 둔다.
- 공개 레포에 프롬프트·응답 원문, 로컬 절대경로, 이메일, 개인 평가 자료를 넣지 않는다.
- 문서의 사실 주장에는 출처(URL)와 접근일을 붙인다. 확인하지 못한 수치는 `미검증`으로 표시한다.
- 재배포 전에 KAMP 이용 약관을 확인한다 (`data/README.md` 참고).

## 용어

아래는 일반적으로 쓰이는 열간단조 용어의 잠정 정의다. 상세는 P1 조사에서 검증한다.

| 용어 | 잠정 설명 |
|---|---|
| 열간단조 | 금속을 재결정 온도 이상으로 가열해 변형시키는 단조 |
| 가열로 | 소재(빌릿 등)를 단조 온도까지 가열하는 설비 |
| 예열 | 본 가열 전 또는 금형 사용 전에 온도를 미리 올리는 공정 |
| 단조 프레스 | 가열된 소재에 압력을 가해 성형하는 설비 |
| 금형 | 소재를 원하는 형상으로 성형하는 틀 |
| 원단위 | 단위 생산량당 에너지 소비량 (예: kWh/개, kWh/ton) |

## 에이전트 작업 시 유의

- 쓰기 범위는 에이전트 파일의 `write_scope`와 DECISIONS 섹션 4를 따른다. `Bash`는 경로 제한을 강제하지 못하므로 `data/raw`, `.workbench/events*`, `.git`에는 명령으로도 접근하지 않는다 (DECISIONS 섹션 10).
- 단계 산출물의 파일 경로는 `.workbench/project.yaml`의 `deliverables`와 일치시킨다.
- KAMP 데이터셋의 변수·단위·기간을 추측해서 쓰지 않는다. 확인된 것만 쓰고 나머지는 `미검증`으로 표시한다.
