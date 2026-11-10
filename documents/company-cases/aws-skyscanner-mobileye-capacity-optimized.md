---
title: Amazon EC2 Auto Scaling 스팟 인스턴스 활용 방법 및 고객 사례 (Skyscanner, Mobileye)
notion_title: black Friday 트래픽을 Spot 인스턴스로 버텨보기
notion_category: 기업 Tech Blog
source_url: https://aws.amazon.com/ko/blogs/korea/capacity-optimized-spot-instance-allocation-in-action-at-mobileye-and-skyscanner/
source_url_original: https://aws.amazon.com/blogs/aws/capacity-optimized-spot-instance-allocation-in-action-at-mobileye-and-skyscanner
source_type: AWS 공식 블로그의 고객 사례 (Jeff Barr 글의 한국어 번역)
published: 2020-04-21 (한국어판 게시일)
accessed: 2026-10-13
reliability: B
tags: [aws-official-blog, capacity-optimized, auto-scaling-group, kubernetes, 100-percent-spot, web-frontend, black-friday]
---

# Amazon EC2 Auto Scaling 스팟 인스턴스 활용 방법 및 고객 사례 (Skyscanner, Mobileye)

## 한 줄 요약

AWS가 Auto Scaling 그룹의 Spot 할당 전략(최저가 vs 용량 최적화)을 설명하고, 용량 최적화로 바꾼 두 고객의 결과를 소개한 글입니다. **Skyscanner는 사용자가 쓰는 웹 플랫폼(프런트엔드)을 전부 Spot 위의 Kubernetes에서 운영합니다.** 공개된 자료 중 "상시 트래픽 웹 서비스 100% Spot"에 가장 가까운 기업 사례입니다.

## 할당 전략 설명 (글이 쓰인 2020년 기준)

- **최저가(lowest-price):** 가장 싼 풀에서 할당합니다. 풀 용량 깊이를 고려하지 않으므로 중단 비용이 낮은 워크로드에 맞습니다.
- **용량 최적화(capacity-optimized):** 실시간 용량 데이터로 가장 여유 있는 풀에서 할당합니다. 중단 비용이 높은 워크로드에 맞습니다.
- 콘솔에서 "Combine purchase options and instance types"를 고르고, 인스턴스 패밀리·세대·크기를 최대한 넓게 고르라고 권합니다. 인스턴스 타입별 가중치도 줄 수 있고, 서브넷(AZ)을 넓게 고르는 것도 도움이 된다고 설명합니다.

> 현재 AWS는 price-capacity-optimized를 권장하고 lowest-price는 권장하지 않습니다. 이 글은 price-capacity-optimized가 나오기 전 글입니다. 자세한 내용은 [../aws-spot-facts.md](../aws-spot-facts.md) 4절을 보세요.

## Skyscanner (온라인 여행 예약)

- 자사 사이트의 **프런트엔드 처리를 Spot에서 실행**하며, 하루 최대 40,000 코어를 씁니다.
- 온라인 플랫폼은 **Spot으로만 구성된 Kubernetes 클러스터**에서 돌아갑니다. 원문은 "스팟 인스턴스에서 전적으로 지원하는 Kubernetes 클러스터"라고 표현합니다.
- 용량 최적화 할당의 효과
  - 새 서비스 출시 시간: 기존 인프라 6~7주 → AWS 50분
  - 비용: AZ와 인스턴스 타입을 다양화해 **코어당 최대 70% 절감**
  - **회수 횟수: 블랙프라이데이 대비 테스트에서 최저가 구성은 200~300회, 용량 최적화 구성은 10~15회**

## Mobileye (자율주행 비전 기술)

- 분석, 머신러닝, Docker로 패키징한 AWS Batch 워크로드를 Spot에서 돌립니다.
- 보통 동시 코어 200,000~300,000개, 하루 피크 약 500,000개를 **모두 Spot**으로 씁니다.
- 용량 최적화로 바꾸고 Spot 모범 사례를 일부 적용한 뒤 **전체 중단 비율이 약 75% 줄었습니다.**

## 프로젝트에 주는 것

- **사용자가 쓰는 웹 서비스도 100% Spot으로 운영할 수 있다는 공식 사례**입니다(Skyscanner). CFP의 "상시 트래픽 web 서비스 compute를 전부 Spot으로"와 같은 형태입니다.
- 할당 전략 하나만 바꿔도 회수 횟수가 한 자릿수 배 이상 달라질 수 있다는 근거입니다(200~300회 → 10~15회). 실험에서 할당 전략을 비교 변수로 둘 이유가 됩니다.
- 이미 2020년에 기업이 해낸 일이므로, "웹 서비스를 100% Spot으로 옮겼다"는 것만으로는 새 연구가 되기 어렵다는 근거로도 쓸 수 있습니다.

## 한계와 주의점

- **지연 시간, 가용성, 오류율 같은 SLO 수치가 없습니다.** "100% Spot에서 SLO를 지켰다"는 근거로는 쓸 수 없습니다.
- 200~300회 대 10~15회 비교의 조건(테스트 기간, fleet 크기, 인스턴스 타입 목록)이 공개되지 않았습니다.
- 70% 절감은 "코어당", "최대"라는 조건이 붙은 값입니다.
- 글에서 링크한 Skyscanner 발표 영상과 Mobileye re:Invent 발표는 열어 보지 않았습니다.
- 2020년 글이라 현재 콘솔 화면과 권장 전략이 다릅니다.

## 인용할 때 쓰는 표현

- "AWS 공식 블로그(2020)에 따르면 Skyscanner의 온라인 플랫폼은 Spot으로만 구성된 Kubernetes 클러스터에서 실행되며, 블랙프라이데이 대비 테스트에서 capacity-optimized 전략으로 회수가 200~300회에서 10~15회로 줄었다."

## 관련 문서

- [../aws-spot-facts.md](../aws-spot-facts.md) — 현재 할당 전략 권장 사항
- [scatterlab-karpenter-spot.md](scatterlab-karpenter-spot.md) — Karpenter에서 priceCapacityOptimized를 쓴 국내 사례
- [../research/skyserve-spothedge-eurosys25.md](../research/skyserve-spothedge-eurosys25.md) — GPU에서는 같은 리전 안 분산만으로 부족하다는 측정
