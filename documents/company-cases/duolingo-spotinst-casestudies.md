---
title: Spotinst 고객사례 모음 — Duolingo, Ticketmaster, Zalando, N26
notion_title: Duolingo의 Spot instance 사용기
notion_category: 기업 Tech Blog
source_url: https://www.mdstech.co.kr/Images/Unified/pdf/ES/Spot-Inst_casestudy.pdf
source_type: 벤더(Spotinst, 현 Spot by NetApp → Flexera) 고객사례의 한국어판. 한컴MDS 배포 PDF (12쪽)
published: 표기 없음 (아래 "작성 시기" 참고)
accessed: 2026-10-13
reliability: C
tags: [vendor-case, elastigroup, ecs, kubernetes, stateful, cassandra, load-balancer, reserved-instance]
---

# Spotinst 고객사례 모음 — Duolingo, Ticketmaster, Zalando, N26

## 한 줄 요약

Spotinst(벤더)가 자사 제품 Elastigroup의 고객사례 4개를 묶은 PDF입니다. 노션 제목은 Duolingo지만 실제로는 4개 회사가 들어 있습니다. 절감률은 65~85%로 나오지만 측정 방법과 SLO 지표가 없는 마케팅 자료입니다.

## 작성 시기

- PDF에 날짜가 없습니다.
- 회사명이 "Spotinst"입니다. Spotinst는 이후 Spot by NetApp으로 이름이 바뀌었습니다. Ticketmaster 사례에 "2017년 AWS 마이그레이션 발표"가 나옵니다.
- 따라서 2018~2019년 무렵 자료로 **추정**합니다. 확인된 날짜는 아닙니다.
- 2026-10-13 확인 시 `spot.io` 고객사례 주소는 `flexera.com` 첫 화면으로 넘어갔습니다.

## 회사별 내용

### Duolingo

- **상황:** 마이크로서비스로 바꾸면서 Terraform으로 관리하는 Amazon ECS에서 워크로드를 돌렸습니다. ECS로 옮기면서 비용이 늘었습니다.
- **문제 1:** 예약 인스턴스(RI)와 Spot을 같이 썼지만, Spot보다 RI를 먼저 쓰도록 우선순위를 줄 수 없었습니다. 그래서 RI가 놀고 예산이 낭비됐습니다.
- **문제 2:** 워크로드마다 필요한 자원(메모리, CPU, 네트워크)이 다른데, 당시 ECS 구성으로는 인스턴스 타입과 크기를 섞어 쓰기 어려웠습니다.
- **해결:** Elastigroup은 쓰지 않는 RI를 먼저 찾아 쓰고, 그다음 Spot을 쓰며, Spot이 없으면 On-Demand로 대체합니다. "Spot Market Score" 히트맵으로 AZ와 인스턴스 타입 조합의 Spot 시장 상태를 보여 줍니다. ECS Autoscaler는 "Tetris Scaling"으로 작업 배치를 최적화하고 여유 용량(headroom)을 관리합니다. Terraform 플러그인을 써서 기존 배포 방식을 바꾸지 않았습니다.
- **규모:** 사용자 3억 명 이상, 월 70억 건 이상의 학습 완료
- **결과:** 프로덕션에서 **AWS Compute 비용 65% 절감**

### Ticketmaster

- **상황:** 2017년 AWS 마이그레이션을 발표하고 Kubernetes 인프라를 운영했습니다.
- **요구:** 인프라와 Pod 확장 자동화, 기존 CI/CD(Terraform, GitLab CI)와 통합, SLA 관리, **Persistent Storage**
- **Elastigroup 기능 설명(벤더 주장)**
  - Spot의 동작, 용량 추세, 가격, 중단율을 **예측**합니다. 중단 위험이 있으면 워크로드를 **최대 15분 전에 미리 재배포**한다고 주장합니다.
  - Spot이 없으면 On-Demand로 되돌리고, 이미 가진 RI를 우선 씁니다.
  - Pod-Driven Autoscaling, Tetris Scaling(스케줄되지 못한 Pod 이벤트를 분석해 맞는 크기의 인스턴스를 띄움), Smart Scale Down(메모리·CPU 사용률 40% 미만이 이어지면 축소)
- **결과:** 본문 중간에는 "1년이 안 되어 AWS 운영 비용 60% 이상 절감", 결과 절에는 "클라우드 컴퓨팅 비용 66% 절약"이라고 적혀 있습니다. 두 수치가 같은 기준인지는 알 수 없습니다.

### Zalando (stateful 사례)

- **상황:** 유럽 17개 시장의 전자상거래 회사입니다. 클라우드 계정 200개 이상과 개발자 수천 명의 비용을 관리해야 했습니다.
- **Spot에 대한 두 가지 우려 (원문 그대로 옮김)**
  1. 회수 사전 공지 2분은 대다수 애플리케이션 운영에 너무 짧다.
  2. **전체 클러스터를 동시에 회수당하거나**, 주요 서비스가 중단되거나 성능이 떨어질 수 있다.
- **Elastigroup을 고른 이유:** 최대 15분 전 사전 예측, On-Demand 자동 대체, 여러 인스턴스 타입·크기 사용의 단순화
- **PoC 선택:** 로드밸런서 뒤의 단순한 stateless 앱 대신 일부러 **stateful 워크로드인 Cassandra 노드**를 골랐습니다.
- **Stateful Elastigroup:** 인스턴스가 바뀌어도 스토리지, 네트워크 구성, 인스턴스 상태를 유지합니다. Cassandra 노드는 IP로 식별되므로 Spot 교체 뒤에도 노드 구성(IP)을 유지하는 것이 중요했습니다.
- **결과:** "고가용성과 길어진 인스턴스 수명을 경험했다"는 정성적 표현만 있습니다. 이후 200개 팀, 300개 이상 AWS 계정에 배포했습니다. CloudFormation을 그대로 지원해서 팀별 도구를 바꾸지 않았습니다.

### N26

- **결과:** EC2 비용 **약 85% 절감**
- **요구:** On-Demand를 쓸 때와 같은 아키텍처를 유지해야 했습니다. ELB를 많이 쓰는 구조였습니다.
- **방식:** Elastigroup이 Spot 인스턴스를 로드밸런서(CLB/ALB/NLB)에 자동 등록합니다. 종료가 예측되는 인스턴스는 AWS 종료 신호가 오기 몇 분 전에 등록을 해제하고 연결을 정리(drain)하면서, 대체 인스턴스를 병렬로 띄웁니다. ELB 상태 검사로 비정상 인스턴스를 교체하는 Auto Healing도 있습니다.
- 이후 Jenkins 플러그인과 HashiCorp Nomad 연동을 확대했다고 적었습니다.

## 프로젝트에 주는 것

- Zalando의 두 가지 우려(2분은 짧다, 클러스터 전체가 동시에 회수될 수 있다)는 CFP 문구 "여러 AZ에서 Spot 회수가 동시에 일어나도 견뎌야 한다"와 같은 문제입니다. 기업도 같은 지점을 가장 걱정했다는 근거로 쓸 수 있습니다.
- Zalando는 stateful(Cassandra)을 Spot에 올린 드문 기업 사례입니다. 다만 해결을 상용 벤더 도구에 맡겼고, 가용성이나 지연 수치는 공개하지 않았습니다.
- N26의 "LB 등록 해제 → drain → 대체 인스턴스 병렬 기동" 순서는 stateless 웹 서비스의 기본 회수 대응 패턴입니다.
- 4개 사례 모두 **절감률만 있고 p95, 가용성 같은 SLO 지표가 없습니다.**

## 한계와 주의점

- 벤더가 자기 제품을 홍보하려고 쓴 자료입니다. 수치는 고객과 벤더가 스스로 보고한 값이고 측정 방법이 없습니다.
- 절감률의 기준(On-Demand 대비인지, 이전 청구서 대비인지)이 나와 있지 않습니다.
- "최대 15분 전 예측"은 벤더 주장입니다. 예측 정확도 근거는 자료에 없습니다.
- 2018~2019년 무렵의 자료로 추정되어, 현재 AWS 기능(Capacity Rebalancing, price-capacity-optimized, ECS capacity provider 등)이 나오기 전 상황입니다.
- Duolingo 수치는 다른 자료와 다릅니다. 검색 단계에서 본 RenovaCloud 글은 "한 분기 만에 컴퓨팅 비용 60% 이상, 전체 AWS 비용 25% 절감"이라고 적었습니다. 그 글은 원문을 열어 확인하지 않았습니다.

## 인용할 때 쓰는 표현

- "Spotinst 고객사례(벤더 자료)에 따르면 Duolingo는 Elastigroup 도입 후 AWS Compute 비용을 65% 줄였다고 보고했다."
- "Zalando는 Spot 도입 시 '2분 공지는 너무 짧다', '클러스터 전체가 동시에 회수될 수 있다'를 주된 우려로 꼽았다(Spotinst 고객사례)."

## 관련 문서

- [../aws-spot-facts.md](../aws-spot-facts.md) — 2분 알림, rebalance recommendation의 공식 설명
- [../research/skyserve-spothedge-eurosys25.md](../research/skyserve-spothedge-eurosys25.md) — 동시 회수를 측정한 논문
- [../research/ding-dong-ditch-2026.md](../research/ding-dong-ditch-2026.md) — 같은 타입·AZ 동시 회수 통계
