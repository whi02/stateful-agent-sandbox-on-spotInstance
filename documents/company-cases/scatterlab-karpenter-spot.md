---
title: Karpenter와 Spot으로 저렴하고 유연하게 노드 프로비저닝하기 (스캐터랩)
notion_title: AI Agent GPU Spot으로 컴퓨팅 서비스 비용 감소
notion_category: 기업 Tech Blog
source_url: https://blog.scatterlab.co.kr/spot-karpenter
source_type: 국내 기업 공식 기술 블로그
author: 홍연준 (스캐터랩 핑퐁팀)
published: 2023-03-20
accessed: 2026-10-13
reliability: B
tags: [korea, eks, karpenter, node-termination-handler, rebalance-recommendation, fis, ai-serving, gpu]
---

# Karpenter와 Spot으로 저렴하고 유연하게 노드 프로비저닝하기 (스캐터랩)

## 한 줄 요약

AI 챗봇 "루다" 서비스의 EKS 서버(상당수가 GPU)를 Spot으로 옮기면서 Cluster Autoscaler를 Karpenter로 바꾸고, rebalance recommendation에 맞춰 노드를 미리 비우도록 NTH를 붙이고, AWS FIS로 회수를 일부러 일으켜 검증한 국내 사례입니다. 절감액이나 SLO 수치는 없습니다.

> 노션 제목은 "AI Agent"지만 원문은 **AI 모델 서빙(챗봇)** 사례입니다. 도구를 쓰며 여러 단계를 수행하는 에이전트 이야기는 아닙니다.

## 배경

- 핑퐁팀은 루다 서비스 서버를 AWS EKS에서 운영하며, 서버 상당수가 비싼 GPU 머신입니다.
- 이전에도 Inferentia 도입, Custom Metrics 기반 스케일링으로 비용을 줄여 왔습니다.
- 여러 AI 모델이 각자에게 맞는 GPU에서 서빙되어 인스턴스 타입이 많습니다.

## 왜 Karpenter로 바꿨나

- Cluster Autoscaler는 스케줄되지 못한 Pod가 생기면 ASG의 desired size를 바꾸는 방식이라 반응이 느렸습니다. Spot 회수가 일어나면 새 노드가 빨리 떠야 했습니다.
- 새 인스턴스 타입을 추가하려면 eksctl과 CloudFormation으로 Node Group을 새로 만들어야 해서 번거로웠습니다.
- Karpenter는 EC2 인스턴스를 직접 생성해 노드가 더 빨리 뜨고, 노드 설정을 CRD로 관리하므로 Helm과 ArgoCD로 다룰 수 있습니다.

## 구성 (2023년, Karpenter v0.18.1 기준)

| 항목 | 설정 |
|---|---|
| 설치 | Helm subchart로 구성하고 Git + ArgoCD로 관리. Karpenter 자체는 EKS Managed Node Group에서 실행하고 taint/toleration으로 전용 노드에 둠 |
| AWSNodeTemplate | **리전의 모든 AZ 서브넷**을 지정해 Spot 용량을 최대한 확보. gp3 100Gi 루트 볼륨 |
| Provisioner | `karpenter.sh/capacity-type`에 `spot`과 `on-demand`를 함께 지정, consolidation 활성화. 인스턴스 타입은 하드웨어에 민감한 서버가 많아 속성 기반이 아니라 직접 지정 |
| 할당 전략 | 글에 따르면 Karpenter가 priceCapacityOptimized로 Spot을 요청 |
| NTH 0.19.3 | `enableSpotInterruptionDraining`, `enableRebalanceMonitoring`, `enableRebalanceDraining`, `enableScheduledEventDraining`을 켬. Spot 노드에만 배치 |
| 추가 안정화 | On-Demand 최소 개수 설정, 커스텀 rebalance recommendation 처리 로직 (자세한 내용은 후속 글로 미룸) |
| 검증 | AWS FIS 실험 템플릿 `aws:ec2:send-spot-instance-interruptions`, `durationBeforeInterruption: PT2M`로 테스트 인프라에 회수를 주입 |

> 글의 Provisioner 예시는 `c5.xlarge`, `c6g.xlarge`(CPU 타입)입니다. 실제 GPU 타입 목록은 공개하지 않았습니다.

## NTH를 함께 쓴 이유

글이 쓰인 시점의 Karpenter는 Spot 회수 알림은 처리했지만 rebalance recommendation은 처리하지 않았습니다. NTH는 rebalance recommendation을 받으면 노드를 cordon하고 drain할 수 있습니다. 그래서 회수 2분 전보다 먼저 Pod를 On-Demand 노드나 다른 Spot 노드로 옮길 수 있습니다.

현재 Karpenter 공식 문서도 "rebalance recommendation에 대해 taint, drain, terminate를 하지 않는다"고 적고 있습니다. NTH와 함께 쓸 수는 있지만 노드 교체가 늘 수 있다고 경고합니다([../aws-spot-facts.md](../aws-spot-facts.md) 5절).

## 결과

- "운영 초반에는 많은 시행착오를 겪었지만, 어느 정도 안정성이 확보된 이후에는 훨씬 저렴하게 서버를 운영할 수 있게 되었다"는 정성적 표현만 있습니다.
- **절감률, Spot 비중, 회수 빈도, 노드 기동 시간, 모델 로딩 시간, 지연·가용성 수치는 모두 없습니다.**

## 프로젝트에 주는 것

- 국내 기업이 **AI 서빙(GPU 포함)을 프로덕션 Spot**에 올린 공개 사례입니다.
- "rebalance recommendation → 미리 drain → 2분 알림 → 종료"라는 2단계 대응 구조와, FIS로 회수를 일부러 일으켜 검증하는 방법은 실험 설계에 그대로 참고할 수 있습니다.
- On-Demand 최소 개수를 둔 것은 이 사례도 **100% Spot이 아니라는 뜻**입니다.

## 한계와 주의점

- 정량 결과가 하나도 없어 "비용을 몇 % 줄였다"는 근거로는 쓸 수 없습니다.
- 글은 rebalance recommendation이 "최대 15분 전에 발행된다"고 했지만, AWS 공식 문서에는 시간 수치가 없고 2분 알림과 동시에 올 수도 있다고 적혀 있습니다.
- Karpenter v0.18.1의 예전 API(`Provisioner`, `AWSNodeTemplate`)입니다. 지금은 `NodePool`(`karpenter.sh/v1`)과 `EC2NodeClass`를 씁니다.
- 예고했던 후속 글(On-Demand 최소 개수, 커스텀 rebalance 로직)은 찾지 못했습니다.

## 인용할 때 쓰는 표현

- "스캐터랩(2023)은 루다 서비스의 EKS 서버를 Spot으로 운영하면서 Karpenter와 NTH의 rebalance draining을 함께 쓰고 AWS FIS로 회수를 주입해 검증했다. 다만 절감률과 SLO 수치는 공개하지 않았다."

## 관련 문서

- [../aws-spot-facts.md](../aws-spot-facts.md) — rebalance recommendation, Karpenter 회수 처리의 현재 공식 설명
- [../research/skyserve-spothedge-eurosys25.md](../research/skyserve-spothedge-eurosys25.md) — GPU Spot은 CPU보다 회수가 잦고 확보가 어렵다는 측정
- [../research/spotserve-asplos24.md](../research/spotserve-asplos24.md) — 모델 로딩 시간이 2분을 넘는 문제
