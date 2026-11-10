---
title: AWS·Karpenter 공식 문서로 확인한 Spot 동작
source_type: AWS 공식 문서, Karpenter 공식 문서
reliability: A
accessed: 2026-10-13
updated: 2026-10-16
tags: [spot, interruption-notice, rebalance-recommendation, spot-placement-score, allocation-strategy, karpenter, fis, spot-quota, azure, gcp]
---

# AWS·Karpenter 공식 문서로 확인한 Spot 동작

다른 자료에 나오는 Spot 동작 설명이 공식 문서와 맞는지 대조할 때 쓰는 기준 문서입니다. 1~7절은 2026-10-13에, 8~9절은 2026-10-16에 공식 문서 원문을 읽고 옮겼습니다. 자료끼리 설명이 다르면 이 문서를 우선합니다.

## 1. 회수 알림 (Spot Instance interruption notice)

출처: [Spot Instance interruption notices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-instance-termination-notices.html)

- EC2가 Spot 인스턴스를 stop 또는 terminate하기 **2분 전**에 알림을 보낸다.
- 회수 동작을 hibernate로 지정하면 알림은 오지만 2분 여유는 없다. 최대 절전이 즉시 시작되기 때문이다.
- 알림은 **best effort**다. 반드시 온다는 보장이 없다.
- 받는 방법은 두 가지다.
  - EventBridge 이벤트: `detail-type: "EC2 Spot Instance Interruption Warning"`, `source: "aws.ec2"`
  - 인스턴스 메타데이터: `http://169.254.169.254/latest/meta-data/spot/instance-action` (회수 예정이 없으면 404)
- AWS는 메타데이터를 **5초마다** 확인하라고 권장한다.

## 2. 리밸런스 권고 (EC2 instance rebalance recommendation)

출처: [EC2 instance rebalance recommendations](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/rebalance-recommendations.html)

- 회수 위험이 높아졌다는 신호다. 2분 회수 알림보다 **먼저 올 수 있다.**
- 항상 먼저 오지는 않는다. 2분 알림과 **동시에 올 수도 있다.** 이것도 best effort다.
- 공식 문서에는 "몇 분 전에 온다"는 시간 수치가 없다. 스캐터랩 글의 "최대 15분 전"은 공식 문서로 확인되지 않은 표현이다.
- 받는 방법
  - EventBridge: `detail-type: "EC2 Instance Rebalance Recommendation"`
  - 메타데이터: `events/recommendations/rebalance` (5초마다 확인 권장)
- Auto Scaling 그룹, EC2 Fleet, Spot Fleet의 **Capacity Rebalancing**을 켜면 이 신호를 받을 때 대체 Spot 인스턴스를 미리 띄운다.
- 2020-11-05 00:00 UTC 이후에 시작한 Spot 인스턴스에만 적용된다.

## 3. Spot Placement Score (SPS)

출처: [Spot placement score](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-placement-score.html), [How Spot placement score works](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/how-sps-works.html)

- Spot 요청이 성공할 가능성을 리전 또는 AZ 단위로 **1~10점**으로 알려 준다. 10점은 "성공 가능성이 매우 높지만 보장은 아님", 1점은 "성공 가능성이 낮음"이다.
- 용량이나 회수 위험을 **보장하지 않는다.** 권고일 뿐이다.
- 상위 10개 리전 또는 AZ만 돌려준다.
- 인스턴스 타입은 **최소 3개**를 지정해야 한다. 3개 미만이면 낮은 점수를 돌려준다.
- 점수는 실제 요청이 같은 구성(목표 용량, 단위, 인스턴스 타입)이고 `capacity-optimized` 할당 전략을 쓸 때만 의미가 있다.
- 같은 요청이라도 시간에 따라 점수가 바뀐다. 점수를 받으면 바로 쓰고, 요청할 때마다 새로 조회하라고 권장한다.
- 제한
  - 목표 용량 상한은 최근 Spot 사용량에 따라 정해진다.
  - 의도에 맞지 않는 사용 패턴이 보이면 24시간 동안 새 요청 구성 수를 제한할 수 있다.
- 추가 요금은 없다.

> 점수 범위 주의: SpotLake(2022)와 HPDC'25 논문의 데이터는 SPS가 1~3 범위로 나온다. 공식 문서의 1~10과 왜 다른지는 두 논문 모두 설명하지 않는다. 원인을 확인하지 못했다.

## 4. Spot 할당 전략 (allocation strategy)

출처: [Allocation strategies (EC2 Fleet / Spot Fleet)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet-allocation-strategy.html)

| 전략 | 공식 설명 요약 | AWS 권장 여부 |
|---|---|---|
| price-capacity-optimized | 용량 여유가 큰 풀을 먼저 고르고, 그중 가장 싼 풀에서 요청한다 | **권장.** stateless 컨테이너, 마이크로서비스, 웹 애플리케이션, 배치 대부분에 가장 좋은 선택 |
| capacity-optimized | 실시간 용량 데이터로 가장 여유 있는 풀에서 요청한다 | 회수 비용이 매우 큰 워크로드에 선택적으로 사용 |
| capacity-optimized-prioritized | 용량 우선, 인스턴스 타입 우선순위는 best effort로 반영 | 특정 타입 선호가 있을 때 |
| diversified | 모든 풀에 고르게 나눈다 | 큰 fleet이나 오래 도는 fleet |
| lowest-price | 가장 싼 풀에서 요청한다 | **비권장.** 회수 위험이 가장 높다 |

- 회수 시 재시작 비용이 큰 워크로드라도 체크포인트를 구현하면 price-capacity-optimized가 잘 맞는다고 설명한다.
- 2020년 Skyscanner·Mobileye 사례(company-cases 참고)는 price-capacity-optimized가 나오기 전 글이라 capacity-optimized를 다룬다.

## 5. Karpenter의 회수 처리

출처: [Karpenter Disruption](https://karpenter.sh/docs/concepts/disruption/), [Karpenter NodePools](https://karpenter.sh/docs/concepts/nodepools/)

- Karpenter가 직접 처리하는 이벤트: Spot Interruption Warning, Scheduled Change Health Event(유지보수), Instance Terminating, Instance Stopping, Instance Status Check Failure
- 이 처리를 쓰려면 **SQS 큐와 EventBridge 규칙**을 만들고 `--interruption-queue` 인자를 지정해야 한다.
- **Rebalance recommendation은 이벤트만 기록하고 taint, drain, terminate는 하지 않는다.** ("Karpenter does not currently support taint, drain, and terminate logic for Spot Rebalance Recommendations.")
- AWS Node Termination Handler(NTH)를 함께 쓸 수는 있다. 다만 NTH가 rebalance recommendation마다 cordon과 drain을 하므로 노드 교체(churn)가 더 많아질 수 있다고 경고한다.
- 현재 API는 `NodePool`(`karpenter.sh/v1`)과 `EC2NodeClass`다. 스캐터랩 글(2023)의 `Provisioner`(`karpenter.sh/v1alpha5`), `AWSNodeTemplate`은 예전 API다.
- NodePool이 여러 capacity type을 허용하면 `reserved` → `spot` → `on-demand` 순서로 우선한다.

## 6. 공식 문서로 확인하지 않은 값 (다른 자료에서 인용)

| 값 | 출처 | 상태 |
|---|---|---|
| Spot Instance Advisor 회수 빈도 구간(<5%, 5–10%, 10–15%, 15–20%, >20%) | SpotLake 논문 | AWS 페이지 미확인 |
| On-Demand 대비 최대 90% 할인 | AWS 마케팅 문구, 여러 자료 | 일반 표현. 실제 할인율은 타입·리전·시점마다 다름 |

## 7. 자료에 나온 잘못된 설명

- **SpotServe 논문**(ASPLOS'24) 서론은 "AWS Spot 유예 시간 30초"라고 적었다. AWS 공식 기준은 **2분**이다. 30초는 Azure 기준이다. 논문 3.2절은 Azure를 30초라고 바르게 적었다.
- **스캐터랩 글**(2023)은 Karpenter가 Spot Fleet으로 인스턴스를 요청한다고 적었다. 현재 Karpenter 문서에서 이 표현은 확인하지 못했다.
- **"GCP Spot 알림 30초"**(SkyServe, Ding-Dong Ditch 논문)는 현재 GCP 문서와 다르다. 지금은 알림 시간을 사용자가 정할 수 있고, 30초는 종료 처리 시간의 최대값이다 (9절).

## 8. 회수 테스트(FIS)와 Spot 할당량

출처: [FIS actions reference](https://docs.aws.amazon.com/fis/latest/userguide/fis-actions-reference.html), [FIS pricing](https://aws.amazon.com/fis/pricing/), [Prepare for interruptions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/prepare-for-interruptions.html), [Spot Instance quotas](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-limits.html)

- FIS 액션 `aws:ec2:send-spot-instance-interruptions`
  - 액션이 시작되면 대상 인스턴스에 **즉시** rebalance recommendation을 보낸다.
  - `durationBeforeInterruption`(회수 대기 시간)은 **2~15분**으로만 정할 수 있다. 회수 2분 전에 회수 알림이 오고, 회수 시각이 되면 회수 동작(terminate 또는 stop)에 따라 처리된다.
  - 그래서 FIS로는 2분보다 짧은 알림이나 알림 없는 회수를 만들 수 없다.
- AWS는 알림이 오기 전에 회수될 수도 있다고 보고, On-Demand 인스턴스를 직접 종료해 예고 없는 중단도 테스트하라고 권장한다.
- FIS 요금: 액션 1분당 **$0.10** (단일 계정 기준)
- Spot 할당량은 vCPU 수로 관리한다.
  - Standard 계열(A, C, D, H, I, M, R, T, Z) Spot 요청 기본값: **5 vCPU**
  - GPU 계열(G·VT, P), DL, Inf, Trn, F, X 기본값: **0**
  - 모두 조정할 수 있고, 사용량에 따라 자동으로 조정된다. 부족하면 Service Quotas에서 증설을 요청한다 (`Max spot instance count exceeded` 오류가 날 때도).
  - 종료한 Spot 인스턴스의 요청을 취소하지 않으면, EC2가 종료를 감지할 때까지 할당량에 계속 잡힌다.

## 9. 다른 클라우드의 회수 알림

| 클라우드 | 알림 | 출처 |
|---|---|---|
| Azure | 회수 **30초 전** 알림(Scheduled Events), best effort. Spot VM에는 SLA가 없음 | [Azure Spot VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/spot-vms) |
| GCP | 알림 시간을 생성할 때 정함: 기본 **0초**, **120초 권장**. 그 뒤 종료 처리 시간은 best effort이며 **최대 30초** | [GCP Spot VMs](https://docs.cloud.google.com/compute/docs/instances/spot) |
| AWS (비교) | 회수 2분 전 알림, best effort (1절) | 1절 |
