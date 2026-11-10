---
title: "SpotServe: Serving Generative Large Language Models on Preemptible Instances"
notion_title: stateful 한 LLM 추론에서 상태를 옮기는 방법
notion_category: 학술 자료 (논문)
source_url: https://arxiv.org/abs/2311.15566
code: https://github.com/Hsword/SpotServe
source_type: 학회 논문 (ASPLOS 2024)
authors: Xupeng Miao, Chunan Shi, Jiangfei Duan, Xiaoli Xi, Dahua Lin, Bin Cui, Zhihao Jia (CMU, 북경대, 홍콩중문대)
published: arXiv 2023-11-27, ASPLOS '24
accessed: 2026-10-13
reliability: A
tags: [llm-serving, gpu-spot, kv-cache, stateful-inference, live-migration, model-parallelism, grace-period]
---

# SpotServe: Serving Generative LLMs on Preemptible Instances (ASPLOS '24)

## 한 줄 요약

여러 GPU에 나눠 올린 LLM을 Spot에서 서빙하는 첫 시스템입니다. 회수 알림이 오면 병렬화 구성을 다시 정하고, 이미 GPU에 올라간 모델 가중치와 **KV cache(추론 중간 상태)**를 다른 인스턴스로 옮겨서 생성 중이던 요청을 끊긴 토큰부터 이어 갑니다. On-Demand만 쓸 때보다 비용은 최대 54% 줄었지만, **평균 지연은 최대 18%, P99 지연은 최대 90% 늘었습니다.**

## 풀려는 문제

- LLM은 데이터·텐서·파이프라인 병렬화로 여러 GPU 인스턴스에 나눠 올립니다. 그래서 인스턴스 1대가 회수되면 같은 파이프라인의 나머지 인스턴스도 멈춥니다(논문 표현: chain crashing).
- 회수 후 처음부터 다시 띄우면 비용이 큽니다. 예: GPT 120B 모델을 저장소에서 읽어 올리는 데 AWS에서 2분이 넘게 걸렸습니다.
- 기존 방식은 끊긴 요청을 다른 파이프라인으로 보내 **처음부터 다시 생성**합니다(rerouting). 이미 만든 토큰이 버려집니다.

## 핵심 기법

| 구성 요소 | 하는 일 |
|---|---|
| 추론 서버 | **On-Demand CPU 인스턴스**에서 실행. 요청 관리자, 인스턴스 관리자(회수·확보 알림 수신), 메타 컨텍스트 관리자로 구성 |
| Parallelization Controller | 구성 C=(D, P, M, B)를 고릅니다. 처리량이 요청 도착률 이상인 구성 중 지연이 가장 낮은 것을 고르고, 불가능하면 처리량을 최대화합니다. Spot 확보가 실패할 수 있어 On-Demand와 Spot을 **동시에** 요청하고, 줄일 때는 On-Demand부터 반납합니다. 교체용 여유 인스턴스를 2대 정도 유지합니다 |
| Device Mapper | 남은 GPU와 새 구성의 위치를 이분 그래프로 놓고 **Kuhn-Munkres 알고리즘**으로 매칭해, 재사용할 수 있는 가중치와 KV cache를 최대화하고 전송량을 최소화합니다 |
| Migration Planner | KV cache를 먼저 옮기고, 가중치는 앞쪽 파이프라인 단계부터 옮겨서 먼저 끝난 단계가 바로 서빙을 시작하게 합니다. GPU 버퍼 메모리 상한을 넘지 않는 순서로 옮깁니다 |
| Stateful Inference Recovery | 진행 상황을 요청 단위가 아니라 **토큰 단위**로 저장합니다. 회수 알림이 오면 "남은 유예 시간 − 이전 시간" 안에 생성할 수 있는 만큼만 토큰을 더 만들고(JIT arrangement), 그 KV cache를 넘겨 다른 파이프라인에서 이어서 생성합니다 |
| 예외 처리 | 예상보다 일찍 회수되면 KV cache는 포기하고 모델 가중치만 옮깁니다. 같은 가중치 조각의 복제본이 모두 사라지면 로컬 디스크나 S3에서 다시 읽습니다 |

## 실험 조건

- 인스턴스: AWS `g4dn.12xlarge`(T4 GPU 4개)
- 회수 이력: g4dn Spot의 실제 12시간 가용성 이력에서 20분 구간 2개(AS, BS)를 뽑아 재생했습니다. On-Demand를 섞는 변형(AS+O, BS+O)도 만들었습니다.
- 모델: OPT-6.7B(GPU 4개), GPT-20B(GPU 12개), LLaMA-30B(GPU 16개)
- 부하: 고정 도착률(1.5 / 0.35 / 0.2 req/s, 감마 분포 CV=6)과 Microsoft MAF 실제 트레이스. 입력 512, 출력 128 토큰
- 기반 엔진: FasterTransformer(C++), 서버 5.6K줄 C++ + 2.2K줄 Python
- 비교 대상: Rerouting(끊긴 요청을 다른 파이프라인에서 재시작), Reparallelization(구성은 바꾸지만 모든 인스턴스를 재시작)

## 핵심 수치

| 항목 | 값 | 조건 |
|---|---|---|
| P99 지연 개선 | 2.4~9.1배 (초록 기준) | 기존 방식 대비 |
| P99 개선 (LLaMA-30B) | Reparallelization 대비 1.34~2.43배, Rerouting 대비 2.14~9.13배 | 고정 부하 |
| P99 개선 (변동 부하) | Reparallelization 대비 최대 2.94배, Rerouting 대비 최대 1.73배 | MAF 트레이스 |
| **비용 절감** | **On-Demand만 쓸 때 대비 최대 54%** | GPT-20B |
| **대가** | **평균 지연 최대 18% 증가, P99 지연 최대 90% 증가** | On-Demand만 쓸 때 대비 |
| 가격 | On-Demand 3.9 USD/h, Spot 1.9 USD/h | 논문 실험 당시 |
| 기법 전체 효과 | 꼬리 지연 1.61배(AS), 3.41배(BS) 감소 | Ablation |
| Interruption Arranger 효과 | 꼬리 지연 29% 감소 | BS 트레이스 |
| 메모리 최적화 이전 효과 | GPT-20B 최소 GPU 수 16 → 12 | |

## 프로젝트에 주는 것

- **"상태가 있는 추론"을 Spot에서 옮기는 구체적 방법**입니다. 상태 = GPU 메모리의 KV cache, 이전 단위 = 토큰입니다.
- **On-Demand와 같은 성능을 유지하는 것이 쉽지 않다는 근거**입니다. 이 논문의 최선 결과도 On-Demand 대비 P99가 최대 90% 나빠졌습니다. "비용은 줄이고 p95/p99는 On-Demand와 같다"를 목표로 하면, 이 논문 수치가 넘어야 할 기준선이 됩니다.
- 시스템의 두뇌(추론 서버)는 On-Demand CPU 인스턴스에 둡니다. **완전한 100% Spot 설계가 아닙니다.**
- 새 인스턴스 기동에서 초기화까지 약 2분이 걸려, AWS 2분 알림만으로는 대체 인스턴스를 제때 띄우기 어렵다는 근거로도 쓸 수 있습니다.

## 한계와 주의점

- **오류:** 서론에 "AWS Spot 유예 시간 30초"라고 적혀 있지만 AWS 공식 값은 2분입니다([../aws-spot-facts.md](../aws-spot-facts.md)). 3.2절의 "Azure 30초"는 맞습니다.
- 실제 회수가 아니라 **회수 이력을 재생**했습니다. 20분 구간 2개뿐입니다.
- 단일 GPU 타입(T4)만 다룹니다. 여러 타입, 여러 클라우드는 향후 과제로 남겼습니다.
- 인스턴스 확보, 배치, 스케줄링은 다루지 않습니다. 후속 논문 SkyServe는 실제 배포에서 SpotServe의 요청 실패율이 리전에 따라 2.0~75.9%였다고 보고했습니다.
- vLLM이 아니라 FasterTransformer 기반입니다.

## 인용할 때 쓰는 표현

- "SpotServe(ASPLOS'24)는 Spot GPU에서 LLM을 서빙하며 On-Demand만 쓸 때보다 비용을 최대 54% 줄였지만, 평균 지연은 최대 18%, P99 지연은 최대 90% 늘었다."
- "SpotServe는 회수 시 KV cache를 토큰 단위로 다른 인스턴스에 옮겨 생성을 이어 간다(stateful inference recovery)."

## 관련 문서

- [skyserve-spothedge-eurosys25.md](skyserve-spothedge-eurosys25.md) — SpotServe의 배치 문제를 지적하고 리전 간 분산을 제안한 후속 연구
- [crab-agent-sandbox-cr-2026.md](crab-agent-sandbox-cr-2026.md) — 추론 상태가 아니라 에이전트 샌드박스 상태를 옮기는 연구
- [../aws-spot-facts.md](../aws-spot-facts.md)
