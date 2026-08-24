---
title: 연구 진행 절차
aliases:
  - Process
  - 연구 체크리스트
tags:
  - research-process
  - checklist
status: living
---

# 연구 진행 절차

> [!abstract] 문서 목적
> 연구의 진행 순서와 현재 상태를 체크리스트로 관리한다.

> [!important] 관리 규칙
> - 완료한 작업은 `[x]`, 아직 완료하지 않은 작업은 `[ ]`로 표시한다.
> - 새로운 작업이 생기거나 진행 상태가 바뀌면 이 문서를 함께 수정한다.
> - 진행 중 처음 보는 단어나 의미가 모호한 단어는 [[wiki|Wiki]]에 추가한다.

## 1. 연구 방향 정리

- [x] 프로젝트 전체 목적 정리 — [[overview|overview.md]]
- [x] 핵심 용어의 초기 정의와 관계 정리 — [[wiki|wiki.md]]
- [x] 연구 동기와 해결하려는 문제 정리 — [[motivation/motivation|motivation.md]]

## 2. 관련 연구 조사

- [x] Related Works 하위 문서 구성과 전체 개요 작성 — [[related_works/related_works_overview|related_works_overview.md]]
- [x] Continual Reinforcement Learning 관련 연구 작성 — [[related_works/continual_reinforcement_learning|continual_reinforcement_learning.md]]
- [x] Cross-Embodiment Learning 관련 연구 작성 — [[related_works/cross_embodiment_learning|cross_embodiment_learning.md]]
- [x] Morphology-Aware Policy Learning 관련 연구 작성 — [[related_works/morphology_aware_policy_learning|morphology_aware_policy_learning.md]]
- [ ] [[experiments/baseline|Baseline]] 확정 후 Related Works의 방법 비중과 비교 대상 재조정

## 3. 실험 설정

- [x] Cross-embodiment dataset 분석 및 1차 작성 완료 — [[experiments/dataset|dataset.md]]
- [x] Metric 설정 — [[experiments/metric|metric.md]]
- [x] Baseline 알고리즘 1차 정리 — [[experiments/baseline|baseline.md]]
- [ ] Round별 실험 정리 — [[experiments/experiments_overview|experiments_overview.md]]

## 4. Round 1 실험

- [x] Round 1의 목적과 실험 범위 1차 정리 — [[experiments/round1/round1_overview|round1_overview.md]]
- [ ] Baseline 재현 및 실행 계획 작성 — [[experiments/round1/round1_baseline_reproduce|round1_baseline_reproduce.md]]
- [ ] Round 1 실험 중 참고사항 기록 — [[experiments/round1/round1_notes|round1_notes.md]]
- [ ] Round 1 주요 결과 정리 — [[experiments/round1/round1_results|round1_results.md]]
