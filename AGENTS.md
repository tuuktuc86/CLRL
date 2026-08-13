# CLRL Project Agent Guide

이 저장소에서 연구 문서나 실험 설계를 수정하는 agent는 아래 문서 체계를 따른다. 이 파일은 작업 규칙과 문서 라우팅만 정의하며, 연구 내용을 중복해서 서술하지 않는다.

## 먼저 읽을 문서

- 연구 문제, 가설, 범위, 예정된 기여 또는 실험 논리를 변경할 때는 [Research Direction](docs/overview/research_direction.md)을 먼저 읽는다.
- 핵심 용어를 작성하거나 기존 표현을 바꿀 때는 [Glossary](docs/overview/glossary.md)를 기준으로 canonical English term을 사용한다.
- Dataset의 shape, field, preprocessing, robot 특성 또는 collection condition을 다룰 때는 [Dataset](docs/experiments/dataset.md)을 확인한다.
- 전체 실험 흐름을 변경할 때는 [Experiment Guideline](docs/experiments/guideline.md)을, Round 1의 model·budget·metric·구현 contract를 변경할 때는 [Round 1 Experiments Guideline](docs/experiments/experiments_1/experiments1_guideline.md)을 읽는다.
- 선행 연구의 분류나 인용을 수정할 때는 [Related Works](docs/overview/related_works.md), `docs/related_works_research/`의 조사 문서와 [reference.bib](docs/reference.bib)을 함께 확인한다.

## Source-of-truth 우선순위

1. 사용자가 가장 최근에 확정한 결정
2. `docs/overview/research_direction.md`의 연구 목적, 범위와 주장 경계
3. `docs/overview/glossary.md`의 프로젝트 용어
4. `docs/experiments/dataset.md`의 dataset contract
5. `docs/experiments/guideline.md`의 전체 실험 방향
6. 각 Round 문서의 세부 실행 명세
7. `docs/related_works_research/`의 조사·포지셔닝 메모

하위 문서가 상위 문서와 충돌하면 충돌을 숨겨서 합치지 말고, 최신 사용자 결정에 맞춰 영향을 받는 문서를 함께 정렬한다. 외부 논문의 고유 용어와 실제 field name은 원문을 유지하되, 프로젝트의 설명 문장은 Glossary 용어에 맞춘다.

## 작성 원칙

- 연구 가설, 예정된 기여와 실험 결과를 구분한다. 측정되지 않은 효과는 가능성이나 검증 질문으로 표현한다.
- Robot별 raw result를 보존하고, normalized score와 canonical morphology distance는 현재 연구 방향에 따라 보조 자료로 취급한다.
- Baseline의 원 논문 설정, 이 프로젝트의 adaptation과 새로 제안하는 method를 명시적으로 구분한다.
- 연구 방향 문서에는 안정적인 결정만 기록하고, model dimension·seed·update 수와 같은 Round별 수치는 해당 실험 문서에 둔다.
- 관련 문서 사이에는 상대 경로 링크를 추가한다. 인용을 추가할 때는 `docs/reference.bib`의 key와 실제 출처가 일치하는지 확인한다.
- 기존 사용자 변경과 무관한 파일은 되돌리거나 재작성하지 않는다.

## 변경 완료 기준

문서 변경은 다음 조건을 만족할 때 완료된다.

1. 변경된 표현이 Glossary와 일치한다.
2. 연구 주장과 실험 protocol이 Research Direction에 모순되지 않는다.
3. 새 링크와 citation target이 실제로 존재한다.
4. 확정되지 않은 결정은 확정된 설정처럼 작성되지 않고 open question으로 남는다.
5. 방향성 결정이 바뀌었다면 관련 overview와 experiment 문서가 같은 결정을 가리킨다.
