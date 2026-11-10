---
title: Multi-Node Spot Instances Availability Score Collection System
notion_title: Spot 회수를 미리 예측해 대응하게 하는 방향 (2/3)
notion_category: 학술 자료 (논문)
source_url: https://hpdc.sci.utah.edu/2025/assets/slides/posters/cheon.pdf
doi: https://doi.org/10.1145/3731545.3735122
full_translation: ../../spot-proposal/HPDC25_Multi-Node_SPS_ko.md
source_type: 학회 포스터 + 2쪽 논문 (ACM HPDC 2025)
authors: Sungkyu Cheon, Kyumin Kim, Kyunghwan Kim, Moohyun Song (국민대), Kyungyong Lee (한양대)
published: 2025-07 (HPDC '25)
accessed: 2026-10-13
reliability: A
tags: [korea, spot-placement-score, multi-node, dataset, gpu-availability]
---

# Multi-Node Spot Instances Availability Score Collection System (HPDC '25)

> 노션 북마크는 학회 포스터 PDF입니다. 2쪽 논문 전문을 한국어로 옮긴 정리본이 이미 있습니다: [../../spot-proposal/HPDC25_Multi-Node_SPS_ko.md](../../spot-proposal/HPDC25_Multi-Node_SPS_ko.md). 이 문서는 프로젝트에 필요한 부분만 요약합니다.

## 한 줄 요약

AWS의 단일 노드 SPS는 **여러 대를 한꺼번에 확보할 수 있는지** 알려 주지 못합니다. 단일 노드 SPS가 최고점(3)인 타입 32개도 50대를 요청하면 성공률이 약 20%로 떨어졌습니다. 그래서 목표 노드 수별 SPS를 10분마다 모아 SpotLake에 공개했습니다.

## 핵심 내용

- **단일 노드 SPS의 한계:** SPS 3인 타입 32개를 무작위로 골라 요청 대수를 늘리자 성공률이 1대 100% → 50대 약 20%로 떨어졌습니다.
- **수집:** 10분마다 여러 목표 노드 수로 SPS를 조회하고, 가격은 AWS 가격 데이터를 그대로 씁니다. S3에 "연-월-일-시각" 이름으로 저장합니다.
- **데이터 범위:** 2024-07-01 ~ 2025-01-15, 17개 리전, 인스턴스 타입 844개
- **분석 결과**
  - 요청 대수가 늘수록 SPS가 낮아지고, **가속 컴퓨팅(GPU)이 가장 가파르게 떨어집니다.**
  - CPU 계열은 **최신 타입일수록 SPS가 낮습니다**(노드 수가 많을수록 더 뚜렷).
  - GPU 계열은 반대로 최신 타입일수록 SPS가 높습니다(p5.48xlarge가 높게 나옴). 원인은 추정만 있습니다.
- **공개:** [spotlake.ddps.cloud](https://spotlake.ddps.cloud)
- **한계:** AWS만 지원합니다.

## 프로젝트에 주는 것

- 여러 AZ에서 동시에 회수된 뒤 **여러 대를 한 번에 다시 확보해야 할 때**, 단일 노드 SPS만 보면 확보 가능성을 과대평가합니다.
- 인스턴스 타입을 고를 때 "최신 CPU 타입일수록 Spot 여유가 적다"는 관찰을 고려할 수 있습니다.

## 한계와 주의점

- 2쪽짜리 짧은 논문과 포스터라 분석 깊이가 제한적입니다.
- SPS 범위를 1~3으로 씁니다. AWS 공식 1~10과의 차이는 설명이 없습니다.
- 높은 SPS가 실제 회수 감소로 이어지는지는 이 자료에서 다루지 않습니다.

## 인용할 때 쓰는 표현

- "HPDC'25 다중 노드 SPS 연구에 따르면 단일 노드 SPS가 최고점인 타입도 50대를 동시에 요청하면 확보 성공률이 약 20%로 떨어졌다."

## 관련 문서

- [spotlake-iiswc22.md](spotlake-iiswc22.md), [ding-dong-ditch-2026.md](ding-dong-ditch-2026.md) — 같은 연구실의 앞뒤 연구
- [../aws-spot-facts.md](../aws-spot-facts.md) — SPS 공식 설명(인스턴스 타입 최소 3개, 목표 용량 지정)
