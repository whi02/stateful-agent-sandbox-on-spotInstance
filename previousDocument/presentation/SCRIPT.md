# 발표 대본: Spot 100%이지만, 성능은 유지한 서비스 구축

- 발표 시간: 10분 (청중: 선생님, 다른 선수)
- 슬라이드: 11장 (1–10장 발표, 11장은 발표 후 질문)
- 대본 분량: 발표 부분 공백 제외 약 2,466자. 1분에 약 300자로 읽으면 약 8.2분입니다.
- 파일: `spot-presentation.pptx` (각 슬라이드 발표자 노트에 같은 대본이 들어 있습니다)

## 말할 때 지킬 것

근거 표의 (docs)는 공식 문서 문장 그대로, (계산)은 공식 데이터로 직접 계산한 값, (CFP)는 선생님 자료, (본인)은 제 생각입니다.

- (docs): "~ 문서에 따르면 ~라고 합니다"
- (계산): "~ 데이터로 제가 계산해 보니 ~입니다" — docs에 있는 숫자라고 말하지 않기
- (본인): "제 생각에는 ~"
- 모르는 질문: "아직 확인하지 못했고, 구현 단계에서 확인하겠습니다"

## 1. 표지 (0:30)

**화면**: 제목, 트랙, 목표 숫자 3개(가용성 99.9% · p95 300 ms · compute 비용 −50%)

**대본**

안녕하세요. 제가 진행할 프로젝트는 'Spot 100%이지만, 성능은 유지한 서비스 구축'입니다. 트랙은 A. Cost Optimization, FinOps입니다. 한 문장으로 말씀드리면, web 서비스의 서버를 전부 Spot 인스턴스로 운영하면서도 가용성 99.9%와 응답 시간 p95 300 ms를 지키고, compute 비용은 절반으로 줄이는 프로젝트입니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| 제안서 | 제목·목표 숫자 | — |

## 2. 주제 설명 (0:50)

**화면**: 왼쪽: CFP 원문 인용 / 오른쪽: 지금(On-Demand 서버) → 목표(Spot 서버 100%, 같은 성능·비용은 절반)

**대본**

이 주제는 CFP의 A 트랙 예시 주제인 'Spot 100% 운영'을 바탕으로 했습니다. CFP 원문은 화면과 같습니다. 상시 트래픽이 있는 web 서비스의 compute를 전부 Spot으로 옮기면서 가용성 99.9%, p95 300 ms를 지키고, 여러 AZ에서 Spot 회수가 동시에 일어나도 견뎌야 합니다. 쉽게 말해, On-Demand로 돌리던 서버를 Spot으로만 바꾸고, 성능은 그대로 두면서 비용은 절반으로 줄이는 것이 목표입니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| CFP | 3장 A 트랙 예시 주제 원문 | — |

## 3. 왜 이 주제인가 (1:00)

**화면**: 왼쪽: 제 이유(회사들의 비용·FinOps 관심, CFP의 면접 직무) / 오른쪽: 최대 90% 할인, 2분 전 통지

**대본**

이 주제를 고른 이유는 두 가지입니다. 첫째, 클라우드를 쓰는 회사는 비용 문제를 중요하게 생각하고, FinOps 분야에 관심을 두는 회사가 많습니다. CFP에서도 A 트랙은 Cloud engineer, FinOps, SRE 직무 면접에 강하다고 나와 있습니다. 둘째, AWS 문서 'Best practices for Amazon EC2 Spot'에 따르면 Spot은 On-Demand보다 최대 90% 저렴합니다. 그런데 같은 문서에, On-Demand와 다른 점은 AWS가 2분 전에 통지하고 회수할 수 있다는 것 하나라고 나옵니다. 이 큰 할인을 실제 서비스에 쓸 수 있는지 직접 확인해 보고 싶었습니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| 본인 | 회사들의 비용·FinOps 관심 (제 이유) | — |
| CFP | 3장 트랙 표: A 트랙 면접 직무 | — |
| docs | 최대 90% 저렴, 2분 전 통지 후 회수 | [링크](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html) |

## 4. 문제: AWS가 말하는 Spot의 한계 (1:00)

**화면**: 한계 3개(용량 보장 없음 · stateful 부적합 · On-Demand fallback 비권장) + 핵심 질문

**대본**

같은 문서에는 Spot의 한계도 분명히 적혀 있습니다. 첫째, AWS는 Spot 용량을 보장하지 않습니다. 목표 용량을 잠시라도 다 채우지 못하는 상황을 견디지 못하는 workload에는 Spot을 권하지 않는다고 합니다. 둘째, Spot은 stateless web server에는 맞지만 stateful workload에는 맞지 않습니다. 셋째, 회수될 때 On-Demand로 넘어가는 방식은 다른 Spot 회수를 늘릴 수 있어서 강하게 말린다고 합니다. 그래서 이 프로젝트의 핵심 질문은 'Spot 100%로도 가용성 99.9%를 지킬 수 있을까?'입니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| docs | 용량 미보장, stateless 적합·stateful 부적합, On-Demand fallback 강하게 비권장 | [링크](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html) |

## 5. 목표 (1:10)

**화면**: 숫자 카드 3개(99.9% · 300 ms · 50%) + 도전 목표 99.99%를 측정·보고만 하는 이유

**대본**

목표는 숫자 세 개입니다. 가용성 99.9%, p95 300 ms, compute 비용 50% 절감입니다. 99.9%는 Spot으로 운영하는 4주 동안 약 40분까지만 멈출 수 있다는 뜻이고, 이 40분은 제가 계산한 값입니다. p95는 부하 생성기가 잰 사용자 쪽 응답 시간으로 판단합니다. 99.99%도 검토했습니다. Well-Architected의 가용성 계산법에 따르면 의존하는 구성 요소의 가용성은 서로 곱합니다. 문서에 나온 ALB와 Multi-AZ DB의 설계 가용성이 각각 99.99%라서, 둘만 곱해도 약 99.98%입니다. 그래서 99.99%는 공식 목표가 아니라 측정해서 보고하는 도전 목표로 두었습니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| CFP | 99.9%, p95 300 ms (A 트랙 예시 주제 조건) | — |
| 계산 | 4주 허용 중단 약 40분 = 28일 × 1,440분 × 0.1% | — |
| docs | hard dependency 가용성은 곱으로 계산 | [링크](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/availability.html) |
| docs | ELB·Multi-AZ DB 설계 가용성 99.99% | [링크](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/appendix-a-designed-for-availability-for-select-aws-services.html) |
| 계산 | 99.99% × 99.99% ≈ 99.98% | — |

## 6. 계획 (1:00)

**화면**: 타임라인 4단계(학습·구축 → 기준선 2주 → Spot 100% 4주 → 분석·보고) + Must / Out of scope

**대본**

계획은 같은 서비스를 두 번 운영해서 비교하는 방식입니다. 먼저 On-Demand로 2주 운영하면서 비용, 가용성, 응답 시간의 기준선을 측정합니다. 그다음 같은 부하로 Spot 100%로 바꿔 4주 운영하고, 이 기간에 회수 실험도 합니다. 학습·구축과 분석에는 나머지 약 4주를 씁니다. 반드시 할 일은 화면의 다섯 가지이고, AI Agent, DB의 Spot 전환, On-Demand fallback, Multi-Region은 범위에서 뺐습니다. DB를 뺀 이유는 AWS가 stateful workload에는 Spot을 권하지 않기 때문입니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| 제안서 | 측정 기간, Scope | — |
| docs | stateful workload에는 Spot 부적합 | [링크](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html) |

## 7. 어떻게 ①: 구조와 AWS 권장 설정 (1:10)

**화면**: 왼쪽: 부하 생성기 → ALB(서울 4 AZ) → ECS service(EC2 Spot 100%) → managed DB / 오른쪽: 문서 근거가 있는 설정 4개

**대본**

구조는 ECS on EC2입니다. 부하 생성기가 ALB로 계속 요청을 보내고, ALB는 서울 리전 4개 AZ에 퍼진 ECS 서비스로 요청을 나눕니다. ECS가 쓰는 EC2는 전부 Spot이고, DB는 managed 서비스로 Spot 범위 밖에 둡니다. 설정은 AWS 문서의 권장 사항을 따릅니다. Spot best practices에 따르면 instance type을 10개 이상 쓰고 모든 AZ를 쓰라고 합니다. 할당 전략은 price-capacity-optimized를 권장하는데, API 문서를 보면 기본값이 lowest-price라서 직접 지정해야 합니다. Capacity Rebalancing을 켜면 회수 위험이 높아진 인스턴스를 미리 교체합니다. 이 설정들로 회수 위험을 여러 곳에 나눌 수 있을 것 같습니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| docs | instance type 10개 이상, 모든 AZ, price-capacity-optimized 권장 | [링크](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-best-practices.html) |
| docs | API 기본값 lowest-price | [링크](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_InstancesDistribution.html) |
| docs | Capacity Rebalancing | [링크](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-capacity-rebalancing.html) |
| docs | 서울 리전 AZ 4개 | [링크](https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions.html) |

## 8. 어떻게 ②: 회수 대응과 검증 (1:10)

**화면**: 대응 3단계(2분 전 통지 → 새 task 먼저 → 요청 실패 방지) + FIS 검증

**대본**

회수가 일어나면 이렇게 대응합니다. EC2 문서에 따르면 Spot은 회수 2분 전에 통지가 옵니다. rebalance 신호가 더 먼저 올 수도 있지만 항상 먼저 온다는 보장은 없습니다. ECS 문서에 따르면 managed instance draining이 기본으로 켜져 있어서, 기존 task를 멈추기 전에 새 task를 먼저 띄웁니다. ALB 문서를 보면 target을 빼는 deregistration delay 기본값이 300초라서 2분보다 깁니다. 그대로 두면 요청이 실패할 수 있어 이 값을 줄이겠습니다. 검증은 AWS FIS로 합니다. Spot 회수를 일부러 일으키는 action이 있어서, 여러 AZ 동시 회수를 재현해 실제로 견디는지 확인하겠습니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| docs | 2분 전 통지, rebalance 신호는 먼저 올 수 있으나 보장 없음 | [링크](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/rebalance-recommendations.html) |
| docs | managed instance draining: 새 task 먼저, 기본 활성 | [링크](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/managed-instance-draining.html) |
| docs | deregistration delay 기본 300초 | [링크](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/edit-target-group-attributes.html) |
| docs | FIS Spot 회수 action, 통지 후 2~15분, tag 지정 시 기본 5대 | [링크](https://docs.aws.amazon.com/fis/latest/userguide/fis-actions-reference.html) |

## 9. 어떻게 ③: 50% 절감은 어떻게 나오나 (1:10)

**화면**: 64%(Advisor 집계) → 1.33배(REL11-BP05 + 서울 AZ 4개) → 약 52%(직접 계산) + trade-off + 예상 비용

**대본**

50%가 어디서 나왔는지 말씀드리겠습니다. 여기 숫자 일부는 docs에 그대로 있는 숫자가 아니라 docs 데이터로 제가 계산한 값입니다. Spot Instance Advisor 원본 데이터를 집계해 보니, 서울 리전 웹 서버용 34개 type의 할인율은 58%에서 72%, 중앙값 64%였습니다. Well-Architected REL11-BP05는 AZ 하나가 사라져도 버틸 만큼 미리 띄워 두라고 합니다. 서울은 AZ가 4개라 1.33배가 필요하고, 0.36에 1.33을 곱하면 약 0.48, 즉 약 52% 절감입니다. 다만 AZ 2개 동시 회수까지 여유 용량으로 막으면 28%로 떨어집니다. 이 균형이 핵심 난제입니다. 예상 비용은 AWS 가격표 기준 6주 약 230달러이고, Pricing Calculator로 다시 확인하겠습니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| 계산 | Advisor 원본 데이터 집계: 서울 34개 type 58~72%, 중앙값 64% (2026-10-10) | [링크](https://aws.amazon.com/ec2/spot/instance-advisor/) |
| docs | AZ 하나가 사라져도 버틸 만큼 미리 프로비저닝 (static stability) | [링크](https://docs.aws.amazon.com/wellarchitected/latest/framework/rel_withstand_component_failures_static_stability.html) |
| docs | 서울 AZ 4개 | [링크](https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions.html) |
| 계산 | 4/3 ≈ 1.33배, 0.36 × 1.33 ≈ 0.48 → 약 52%, 2개 AZ 대비 시 28% | — |
| 계산 | 예상 비용 6주 약 $230 (AWS Price List 단가 기반 예측) | [링크](https://calculator.aws/) |

## 10. 마무리 (0:30)

**화면**: 한 문장 정리 + 다음 단계 + 감사합니다

**대본**

정리하겠습니다. AWS가 용량을 보장하지 않는 Spot으로, 설계와 측정을 통해 가용성 99.9%를 지키고 compute 비용을 절반으로 줄이는 것이 이 프로젝트의 목표입니다. 다음 단계는 운영할 서비스와 부하 생성 도구를 정하고 기준선 측정을 시작하는 것입니다. 들어 주셔서 감사합니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| 제안서 | 목표·다음 단계 | — |

## 11. 질문 (발표 후 선생님께) (발표 후)

**화면**: 결정이 필요한 질문 5개

**대본**

발표 후에 여쭤보라고 하셔서, 결정이 필요한 다섯 가지를 정리했습니다. 첫째, 운영할 서비스를 직접 만든 stateless API와 managed DB로 진행해도 될지. 둘째, 부하 생성 도구로 k6를 서버 1대에서 상시 실행해도 될지. 셋째, Scope 구분이 적절한지. 넷째, 사용할 AWS 계정과 월 예산 한도, 그리고 FIS 실험과 권한 설정을 해도 되는지. 다섯째, 기준선 2주와 Spot 4주, 학습·구축·분석 약 4주로 잡은 일정이 적절한지입니다.

**근거**

| 구분 | 내용 | 출처 |
|---|---|---|
| 본인 | Slack DM으로 정리한 질문 | — |
