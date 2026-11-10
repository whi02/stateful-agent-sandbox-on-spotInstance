---
title: "SkyServe: Serving AI Models across Regions and Clouds with Spot Instances"
notion_title: 여러 AZ에서 동시에 Spot 회수가 진행되었을 때 대처 방법
notion_category: 학술 자료 (논문)
source_url: https://arxiv.org/abs/2411.01438
code: https://github.com/skypilot-org/skypilot (SkyServe는 SkyPilot의 serve 기능)
source_type: 학회 논문 (EuroSys 2025)
authors: Ziming Mao, Tian Xia, Zhanghao Wu, Wei-Lin Chiang, Tyler Griggs, Romil Bhardwaj, Zongheng Yang, Scott Shenker, Ion Stoica (UC Berkeley, ICSI)
published: EuroSys '25 (arXiv v2 2025-03-03)
accessed: 2026-10-13
reliability: A
tags: [correlated-preemption, multi-az, multi-region, multi-cloud, on-demand-fallback, overprovisioning, gpu-spot, llm-serving]
---

# SkyServe: Serving AI Models across Regions and Clouds with Spot Instances (EuroSys '25)

## 한 줄 요약

Spot GPU 회수는 **같은 리전 안의 여러 존에서 함께 일어나는 경향**이 강하다는 것을 측정하고, Spot 복제본을 여러 리전·클라우드에 흩고(Dynamic Placement), Spot을 조금 더 띄워 두고(Overprovisioning), Spot이 모자라면 그때만 On-Demand를 띄우는(Dynamic Fallback) 정책 **SpotHedge**를 제안했습니다. On-Demand만 쓸 때보다 41~44% 저렴하고 요청 실패율은 0.34~0.62%였습니다.

## 측정으로 확인한 문제 (2장)

| 관찰 | 수치 | 조건 |
|---|---|---|
| Spot GPU 가격 | On-Demand의 8~50% | 2024-10-23 각 클라우드 API 가격 |
| 한 리전 전체가 Spot GPU를 못 주는 시간 | 33.1% | AWS 트레이스 1개, 리전 내 모든 존 |
| us-west-2 Spot 불가 시간 | 21% | 논문 end-to-end 실험 기간 |
| **같은 존에서 회수 후 5분 안에 추가 회수** | **83~97%의 경우** | AWS p3.2xlarge 16대, 3개 존, 3주 트레이스 |
| 같은 존의 다른 Spot이 150초 안에 회수 | 34~95% | GCP a2-ultragpu-4g, 6개 존, 3일 트레이스 |
| 존 간 회수 상관계수 | 같은 리전 최대 0.59, 다른 리전 0.03~0.27 | AWS V100, 3개 리전 8개 존, 2개월 트레이스 |
| Spot GPU 확보 가능 시간 비율 | 16.7~90.4% | 트레이스별 |
| Spot CPU 확보 가능 시간 비율 | 95.6~99.9% | GCP c3-highcpu-176 |
| Spot Instance Advisor 회수 빈도 | GPU 대개 >20%, CPU <5% | |
| Llama-2-7b(vLLM) 인스턴스 기동~서빙 준비 | 183초 | AWS, 이미지 사전 설치. 이미 2분 알림보다 김 |
| 순수 Spot ASG(단일 리전) | 요청의 49.4%가 실패 또는 시간 초과 | |
| 고정 비율 ASG(On-Demand 1 + Spot 4) | 순수 Spot 대비 비용 1.56배, Spot 변동기 실패율 36% | g5.48xlarge, On-Demand $16.3/h, Spot $4.9/h |

정리하면, 같은 리전에서 AZ만 나눠서는 GPU Spot의 동시 회수와 용량 부족을 피하기 어렵다는 것이 이 논문의 출발점입니다.

## SpotHedge 정책

1. **Dynamic Placement:** 존 목록을 "사용 가능(Z_A)"과 "회수가 잦음(Z_P)"으로 나눕니다. 회수가 일어난 존은 Z_P로, 기동에 성공한 존은 Z_A로 옮깁니다. 새 복제본은 Z_A 중 현재 배치가 적고 싼 존에 둡니다. Z_A가 2개 미만이 되면 Z_P를 모두 Z_A로 되돌려, 한 존에 몰리는 것을 막습니다. 탐색 범위를 단일 존에서 여러 리전으로 넓히자 가용성이 A100은 29.9% → 95.8%, V100은 68.2% → 99.2%로 올랐습니다(트레이스 분석).
2. **Overprovisioning:** 필요한 수(N_Tar)보다 Spot 복제본을 N_Extra개 더 띄웁니다. 존들이 서로 상관이 적으면 N_Extra는 작아도 충분했다고 보고합니다.
3. **Dynamic Fallback:** Spot이 회수되면 바로 On-Demand를 띄우고, Spot이 다시 준비되면 On-Demand를 내립니다. On-Demand 수는 `O(t) = min(N_Tar, N_Tar + N_Extra − S_ready(t))`입니다. 항상 켜 두는 고정 On-Demand 풀은 두지 않습니다.
4. 비교 기준으로 전체 회수 이력을 안다고 가정한 최적 정책(Omniscient, ILP)을 함께 제시했습니다.

## 시스템과 동작

- SkyPilot 위에 약 7,000줄 Python으로 구현했습니다. vLLM, TGI, Triton과 함께 씁니다.
- 서비스 컨트롤러(복제본 수명 주기, readiness probe), 오토스케일러(지난 1분 평균 요청률 기준, 10분간 계속 넘을 때 조정), 로드밸런서(라운드로빈 또는 최소 부하)로 구성됩니다.
- **회수 시 진행 중이던 요청은 중단되고 클라이언트나 프록시가 다시 보냅니다.** 상태를 옮기지 않습니다.
- 회수 알림은 best effort라 가용성 문제를 해결하지 못한다고 보고, 알림에만 의존하지 않습니다.

## 실험 조건

- 실제 클라우드에서 실시간 회수를 겪으며 약 22시간 실행했습니다. 요청 133k건, 비용 $4.1k입니다.
- 실험 1: Llama-2-70B, vLLM, `g5.48xlarge`(A10G 8개). 실험 2: OPT-6.7B, SpotServe, `g4dn.12xlarge`(T4 4개)
- 부하: Chatbot Arena 실제 트레이스(급증 포함). 요청 제한 시간 100초(70B), 20초(6.7B)
- SkyServe 리전: us-east-2, us-west-2, eu-central-1. 비교 대상은 us-west-2 한 리전
- 비교 대상: AWS ASG(On-Demand 10%), MArk, AWS Spot node pool, SpotServe
- Spot Available(확보율 91~100%)과 Spot Volatile(45~46%) 두 상황으로 나눴습니다.

## 핵심 결과

| 항목 | SkyServe | 비교 |
|---|---|---|
| 요청 실패율 (70B) | **0.34~0.62%** | ASG 0.51%/36%, AWSSpot 49%/94%, MArk 6.8%/79% (Available/Volatile) |
| 요청 실패율 (6.7B, SpotServe 위) | 0.05~0.4% | 다른 시스템 52~95% |
| 지연 (ASG 대비) | P50 1.1~1.6배, P90 1.1~2.5배, P99 1.4~1.9배 개선 | |
| 지연 (AWSSpot 대비) | P50 2.6~3.9배, P90 2.5~3.1배, P99 1.9~2.7배 개선 | |
| 비용 (On-Demand만 쓸 때 = 100%) | Available 56%, Volatile 59% | ASG 70%/78%, AWSSpot 65%/37%, MArk 47%/42% |
| On-Demand만 쓸 때 대비 절감 | **41~44%** (초록은 평균 43%) | |
| Dynamic Fallback 비용 | 전체 비용의 26~31% | |

AWSSpot과 MArk는 Volatile 상황에서 SkyServe보다 쌌지만, Spot을 충분히 확보하지 못해 실패율이 79~94%였습니다.

## 프로젝트에 주는 것

- CFP의 **"여러 AZ에서 Spot 회수가 동시에 일어나도 견뎌야 한다"**를 정량으로 뒷받침하는 가장 직접적인 자료입니다. 같은 리전 안의 존들은 함께 회수되는 경향이 있습니다.
- 대응 방법 세 가지(실패 도메인 확장, Spot 과다 확보, 동적 On-Demand fallback)를 비용과 실패율로 비교할 수 있습니다.
- "고정 On-Demand 풀"은 Spot이 충분할 때 돈을 낭비하고, "고정 Spot 풀"은 Spot이 부족할 때 실패한다는 측정이 있어 혼합 비율을 설계할 때 근거가 됩니다.
- GPU와 CPU의 차이(확보 가능 시간 16.7~90.4% 대 95.6~99.9%)는 워크로드를 CPU로 할지 GPU로 할지 고를 때 중요한 근거입니다.

## 한계와 주의점

- **LLM 요청은 처리에 수 초가 걸리므로 리전 간 왕복(미국–유럽 약 100ms)이 묻힌다**는 전제입니다. p95 300ms 같은 짧은 웹 요청에는 그대로 옮길 수 없습니다. 논문도 TTFT가 늘 수 있다고 인정하고, 지연에 민감한 요청은 가까운 존으로 보내는 것을 향후 과제로 남겼습니다.
- 회수 시 진행 중 요청은 버리고 재시도합니다. 상태 있는 워크로드의 상태 이전은 다루지 않습니다.
- 100% Spot이 아니라 Spot이 부족할 때 On-Demand로 채우는 설계입니다.
- end-to-end 실험은 약 22시간, 4회입니다. 상관관계 분석은 몇 주~두 달짜리 트레이스입니다.
- 주로 GPU를 다뤘습니다. CPU Spot의 동시 회수 패턴은 측정하지 않았습니다.

## 인용할 때 쓰는 표현

- "SkyServe(EuroSys'25)의 3주 AWS 트레이스에서는 한 존에서 Spot GPU 회수가 일어나면 83~97%의 경우 5분 안에 추가 회수가 이어졌다."
- "같은 리전 존 간 회수 상관계수는 최대 0.59, 다른 리전 간은 0.03~0.27이었다(AWS V100, 2개월)."
- "SpotHedge는 On-Demand만 쓸 때보다 41~44% 저렴하면서 요청 실패율 0.34~0.62%를 기록했다."

## 관련 문서

- [spotserve-asplos24.md](spotserve-asplos24.md) — 복제본 내부의 병렬화·상태 이전 (SkyServe는 복제본 배치를 다룸)
- [ding-dong-ditch-2026.md](ding-dong-ditch-2026.md) — 같은 타입·AZ 안의 동시 회수 통계(3분 안에 92.9%)
- [../company-cases/duolingo-spotinst-casestudies.md](../company-cases/duolingo-spotinst-casestudies.md) — Zalando의 "클러스터 동시 회수" 우려
- [../aws-spot-facts.md](../aws-spot-facts.md)
