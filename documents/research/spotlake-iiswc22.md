---
title: "SpotLake: Diverse Spot Instance Dataset Archive Service"
notion_title: Spot 회수를 미리 예측해 대응하게 하는 방향 (1/3)
notion_category: 학술 자료 (논문)
source_url: https://arxiv.org/abs/2202.02973
service: https://spotlake.ddps.cloud
source_type: 학회 논문 (IISWC 2022)
authors: Sungjae Lee, Jaeil Hwang, Kyungyong Lee (국민대학교)
published: IISWC 2022 (arXiv v2 2022-10-25)
accessed: 2026-10-13
reliability: A
tags: [korea, dataset, spot-placement-score, interruption-frequency, spot-price, availability-prediction]
---

# SpotLake: Diverse Spot Instance Dataset Archive Service (IISWC 2022)

## 한 줄 요약

AWS가 2017년 Spot 가격 정책을 바꾼 뒤로 **가격 이력은 가용성을 알려 주지 못합니다.** 그 대신 SPS(Spot Placement Score)와 회수 빈도 데이터를 10분마다 모아 이력으로 쌓는 공개 서비스를 만들고, 실제 Spot 요청 실험으로 어떤 신호를 믿어야 하는지 확인했습니다. 결론은 "**확보 여부는 SPS, 회수 여부는 SPS와 회수 빈도를 함께 보라. 두 값이 다르면 SPS를 우선하라**"입니다.

## 다룬 데이터

| 데이터 | 내용 | 당시 문제 |
|---|---|---|
| Spot 가격 이력 | 최대 3개월 이력 제공 | 2017년 정책 변경 후 가격이 거의 안 변해 가용성 예측에 쓸모가 없어짐 |
| Spot Instance Advisor 회수 빈도 | 지난 한 달의 회수 빈도 구간(<5% … >20%)과 절감률 | 웹사이트에서만 보이고 이력이 없음 |
| Spot Placement Score | 요청 성공 가능성 점수 (2021년 출시) | 이력이 없음. 조회 제한이 큼(실측으로 계정당 24시간에 고유 질의 50개, 결과 최대 10개) |

- 논문은 회수 빈도 구간을 SPS와 비교하려고 1.0~3.0 점수로 바꿨습니다("interruption-free score", 높을수록 안정).
- 논문 데이터의 SPS도 1.0~3.0 범위입니다. AWS 공식 범위(1~10)와 왜 다른지는 설명이 없습니다([../aws-spot-facts.md](../aws-spot-facts.md) 3절).

## 분석 결과 (2022-01-01 ~ 2022-06-30, 181일, 10분 간격)

- 평균 SPS 2.8, 평균 interruption-free score 2.22
- **가속 컴퓨팅(GPU 등) 계열이 가장 낮습니다.** 평균보다 SPS 12.07%, interruption-free 34.98% 낮았습니다. 딥러닝 수요 때문으로 추정합니다.
- 가격, SPS, 회수 빈도 사이의 피어슨 상관이 약하고, 서로 반대 정보를 주는 경우가 많습니다. 특히 가격은 가용성과 거의 상관이 없습니다.

## 실제 Spot 요청 실험

- SPS와 interruption-free 점수를 High(3)/Medium(2)/Low(1)로 나눈 조합 5개에서 인스턴스 타입과 AZ를 층화 표본으로 골라 **503건**을 실험했습니다.
- 각 건마다 Spot 요청 1개(입찰가 = On-Demand 가격, persistent)를 내고 24시간 동안 5초마다 상태를 기록했습니다.

| 조합 (SPS-IF) | 24시간 내 확보 실패 | 한 번 이상 회수 |
|---|---|---|
| H-H | 0% | 14.71% |
| H-L | 0% | 40.52% |
| M-M | 25.49% | 39.22% |
| L-H | 58.18% | 30.91% |
| L-L | 45.61% | 45.61% |

- SPS가 높으면 **모든 요청이 확보**되었습니다. SPS가 낮으면 확보 실패의 신호입니다.
- 두 점수 모두 높을 때 28.07%가 1초 안에, 90% 이상이 135초 안에 확보되었습니다. 둘 다 낮으면 확보까지 중앙값 1,322초가 걸렸습니다.
- 확보 후 회수까지 시간의 중앙값은 H-L(SPS 높음) 6,872초, L-H(회수 빈도만 좋음) 2,859초였습니다.

## 예측 실험

이력 데이터(지난 한 달 SPS, interruption-free)로 기본 설정의 랜덤 포레스트를 학습해 "회수 없음 / 회수됨 / 확보 못 함"을 분류했습니다.

| 방법 | Accuracy | F1 |
|---|---|---|
| 현재 interruption-free만 | 0.45 | 0.43 |
| 현재 SPS만 | 0.64 | 0.58 |
| 현재 절감률만 | 0.39 | 0.28 |
| **이력 기반 랜덤 포레스트** | **0.73** | **0.73** |

## 프로젝트에 주는 것

- Spot 인스턴스 타입과 AZ를 고를 때 **가격이 아니라 SPS와 회수 빈도**를 써야 한다는 근거입니다.
- "회수 예측으로 미리 대응한다"는 방향의 출발점입니다. 이력이 있으면 현재 값만 볼 때보다 예측이 낫습니다.
- 국내 연구실(국민대 → 현재 한양대 이경용 교수 연구실)의 연구 계보입니다. HPDC'25와 Ding-Dong Ditch로 이어집니다.

## 한계와 주의점

- 2022년 상반기 데이터입니다. SPS 조회 제한의 공식 설명도 지금은 다릅니다(현재: 오용 패턴이 보일 때 24시간 동안 새 구성 수를 제한).
- 실험은 단일 인스턴스 요청 기준입니다. 여러 노드를 동시에 요청할 때는 다릅니다(HPDC'25 참고).
- 예측 모델은 일부러 단순하게 만든 예시입니다.

## 인용할 때 쓰는 표현

- "SpotLake(IISWC'22)의 503건 실험에서 SPS가 높은 경우 24시간 안에 모든 Spot 요청이 확보되었고, SPS와 회수 빈도가 모두 높아도 14.71%는 한 번 이상 회수되었다."
- "2017년 AWS Spot 가격 정책 변경 이후 가격 이력은 가용성 예측에 거의 쓸모가 없다(SpotLake)."

## 관련 문서

- [multinode-sps-hpdc25.md](multinode-sps-hpdc25.md) — 같은 연구실의 다중 노드 SPS
- [ding-dong-ditch-2026.md](ding-dong-ditch-2026.md) — 같은 연구실의 저비용 가용성 관측과 예측
- [../aws-spot-facts.md](../aws-spot-facts.md) — SPS 공식 설명
