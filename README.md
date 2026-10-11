# Manufacturing AX Fieldbook

가짜연구소 13기 프로젝트. 열간단조 공정의 에너지 효율 문제를 조사하고, [KAMP](https://www.kamp-ai.kr)(KAMP 인공지능 제조 플랫폼) 데이터로 AX(AI Transformation) 도입 사례를 만든다.

## 무엇을, 왜

- 무엇을: 열간단조 에너지 소비 구조 조사 → AX 유스케이스 설계 → 데이터 전처리 → 절감 요인 분석과 솔루션 제안 → 에너지 예측과 절감 효과 추정.
- 왜: 제조 현장의 에너지 효율 개선을 데이터로 따져 보는 과정을 재현 가능한 형태로 기록한다. 병행 목표는 서브에이전트 오케스트레이션을 실습하고, 프로젝트 관리 도구를 다른 프로젝트에도 쓸 수 있게 만드는 것이다.

## 현재 상태

시작 단계다. 아직 분석 결과나 수치는 없다. 단계별 진행은 `.workbench/project.yaml`에 기록된다.

## 구조

```
docs/DECISIONS.md     프로젝트 결정 사항 (명세)
docs/                 단계별 문서 (research, ax, data, analysis, models, simulation, fieldbook)
src/                  파이프라인과 모델 코드
notebooks/            분석 노트북
data/                 데이터 위치 (커밋하지 않음, data/README.md 참고)
.claude/agents/       도메인 에이전트 6개
.workbench/           프로젝트 단계·태스크 정의 (project.yaml)
CLAUDE.md             Claude Code용 프로젝트 맥락
```

## 에이전트 워크플로 요약

1. 단계는 5개다: 조사, AX 도입 설계, 데이터 전처리, 요인 분석·솔루션 제안, 예측·절감 분석.
2. 단계마다 담당 서브에이전트가 정해져 있고, 메인 세션이 오케스트레이터 역할을 한다.
3. 에이전트는 승인된 명세(`docs/DECISIONS.md`) 안에서 중간 확인 없이 일한다.
4. 에이전트마다 쓸 수 있는 경로(쓰기 범위)와 완료 기준이 정해져 있다.
5. 단계가 끝나면 독립 검토 에이전트가 결론마다 반대 증거를 붙여 검증한다.
6. 사람은 승인 지점에서만 개입한다. 첫 승인 지점은 데이터셋 선택이다.
7. 요인 해석과 솔루션 판단은 사람이 한다. 에이전트는 근거와 초안을 제공한다.
8. 상태는 `.workbench/project.yaml`에, 실행 이벤트는 로컬 로그에 기록된다 (로그는 커밋하지 않는다).
9. 공통 에이전트·대시보드·스킬은 별도 레포 [`agent-workbench`](https://github.com/dpcivl/agent-workbench)의 플러그인으로 제공된다.
10. 단계 실행은 `/agent-workbench:run-phase <phase-id>`로 시작한다.

플러그인은 `agent-workbench`를 내려받은 뒤 이 레포 폴더에서 `claude --plugin-dir <agent-workbench 경로>`로 세션을 열어 로드한다 (세션 한정, 설정을 바꾸지 않는다). 마켓플레이스 설치는 아직 검증하지 않았다.

처음 쓰는 경우의 단계별 안내(설치부터 P1 실행, 승인, 대시보드까지)는 [agent-workbench README](https://github.com/dpcivl/agent-workbench#따라-하기-pseudolab-13th에서-쓰기)를 따라 하면 된다.

## 데이터 재현 방법

KAMP 데이터는 이 레포에 포함되어 있지 않다. 직접 내려받아 사용한다.

1. KAMP에서 사용할 데이터셋을 직접 내려받는다. 이용 조건을 먼저 확인한다.
2. 압축을 풀어 `data/raw/` 아래에 둔다 (원본 파일은 수정하지 않는다).
3. 전처리 파이프라인(`src/pipeline/`, P3 이후 제공)을 실행해 `data/processed/`를 만든다.

자세한 안내는 [data/README.md](data/README.md)를 본다.

## 라이선스

[MIT](LICENSE). 단, KAMP 데이터에는 이 라이선스가 적용되지 않으며 KAMP의 이용 조건을 따른다.

## 문서

- [docs/DECISIONS.md](docs/DECISIONS.md): 목적, 에이전트, 단계 흐름, 데이터 계약, 운영 규칙
