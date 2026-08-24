---
title: 연구 개요
aliases:
  - 프로젝트 개요
  - Project Overview
tags:
  - cross-embodiment
  - continual-reinforcement-learning
  - project-overview
status: draft
---

# 연구 개요

> [!abstract] 문서 목적
> 본 문서는 프로젝트에 대한 전체적인 설명을 진행한다.

> [!info] 관련 문서
> [[wiki|Wiki]] · [[process|Process]] · [[motivation/motivation|Motivation]] · [[related_works/related_works_overview|Related Works]] · [[experiments/experiments_overview|Experiments Overview]]

## 프로젝트 설명

본 프로젝트는 서로 다른 robot embodiment가 순차적으로 주어지는 continual reinforcement learning 환경에서 morphology 정보를 활용한다. 이를 통해 이전 embodiment의 성능을 유지하면서(stability) 새로운 embodiment의 학습에 과거 지식을 활용할 수 있는지 검증한다.

## 파일 구조

- `docs_v2/`
  - [[overview|overview.md]] — 프로젝트 전체 개요
  - [[wiki|wiki.md]] — 단어 정리
  - [[process|process.md]] — 연구 진행 과정과 주요 결정 기록
  - `motivation/`
    - [[motivation/motivation|motivation.md]] — 연구 동기와 필요성
  - `related_works/`
    - [[related_works/related_works_overview|related_works_overview.md]] — 관련 연구 전체 개요
    - [[related_works/continual_reinforcement_learning|continual_reinforcement_learning.md]] — Continual Reinforcement Learning 관련 연구
    - [[related_works/morphology_aware_policy_learning|morphology_aware_policy_learning.md]] — morphology-aware policy learning 관련 연구
    - [[related_works/cross_embodiment_learning|cross_embodiment_learning.md]] — Cross-Embodiment Learning 관련 연구
    - [reference.bib](related_works/reference.bib) — Related Works citation 정보
  - `experiments/`
    - [[experiments/experiments_overview|experiments_overview.md]] — 연구 가설과 각 실험 단계 설명
    - [[experiments/dataset|dataset.md]] — dataset 구성
    - [[experiments/baseline|baseline.md]] — 비교 방법의 설명과 실험 공정성
    - [[experiments/metric|metric.md]] — 실험 성능 측정 지표
    - `round1/`
      - [[experiments/round1/round1_overview|round1_overview.md]] — Round 1의 목표와 실험 범위
      - [[experiments/round1/round1_baseline_reproduce|round1_baseline_reproduce.md]] — Round 1 baseline 재현 및 실행 계획
      - [[experiments/round1/round1_notes|round1_notes.md]] — Round 1 실험 중 참고사항과 메모
      - [[experiments/round1/round1_results|round1_results.md]] — Round 1 주요 결과와 분석

## 프로젝트 공통 규칙
> - 프로젝트 전체에 적용되는 규칙은 이 문서에 기록한다.
> - `docs_v2`의 파일 또는 디렉토리 구조가 바뀌면 위 파일 구조도 같은 변경 작업에서 함께 수정한다.
> - 새로운 단어나 의미상 혼동되는 단어는 [[wiki|wiki.md]] 문서에서 점검한다.
> - 연구 진행 상태가 바뀌거나 새로운 작업이 추가되면 [[process|process.md]]의 체크리스트도 같은 변경 작업에서 함께 수정한다.
> - 본문의 일반 영문 용어와 domain term은 `robot`, `dataset`, `observation`, `morphology`처럼 소문자로 쓴다. 제목과 heading에서는 Title Case를 허용한다.
> - 고유명사와 약어는 `MuJoCo`, `PPO`, `MJCF`, `IQL`처럼 공식 표기를 유지한다.
> - 파일명, field, dataset·robot ID와 변수명은 backtick으로 표시한다.
> - `cross-embodiment`는 하이픈을 포함한 표기를 canonical form으로 사용한다.
