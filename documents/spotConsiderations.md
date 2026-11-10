# Spot 프로젝트 고려 사항

사용자가 정리한 Spot 프로젝트 고려 사항 11개를 원문 그대로 남기고, 항목마다 지금 아는 것(근거), 아직 모르는 것, 확정한 설계 방향을 붙인 문서입니다. 정식 주제·목표를 정하고 제안서 3·4·6번(지표, 접근 방법, 리스크)을 쓸 때 출발점으로 씁니다.

- 작성: 2026-10-16
- 원문: 사용자 작성 (1절, 수정하지 않음)
- 근거 표기: `documents/` 경로, 또는 공식 문서 링크. `documents/`에 없는 근거는 `(documents 밖)`으로 표시했습니다. 5절의 공식 문서 사실은 2026-10-16에 원문을 읽고 확인했습니다.
- `확정(2026-10-16)`: Claude가 제안하고 사용자가 2026-10-16에 모두 채택한 설계 방향입니다. 4절(원문에 없는 고려 사항) A~F도 같은 날 모두 채택했습니다.

## 1. 사용자 원문 (2026-10-16)

1. Stateful Application과 Stateless Application을 각각 Spot instance에 적용하면 어떤 차이점이 있는지
2. 언제 2분전의 Termination Notice가 도착할지를 예측할 수 있을지?
3. 2분 안으로 Instance Provisioning이나 이런걸 하려면 어떻게 해야 할지
4. 비용 계산과 "중단 빈도"라는 데이터도 있는데 이를 활용할수는 없는지?
5. 참고할 수 있는 지표는 없는지?
6. 100% Spot instance로 전환하는 것이 과연 효율적일지?
7. On-Demand의 데이터를 어떻게 측정하고 비교할 건지? (굳이 실제 배포를 통하지 않아도 측정할 수 있고, 다른 방법은 없는지)
8. Spot 비용을 조회해서 어느정도 비용이 소요될지도 예측할 수 있을지?
9. 가용성을 측정하기 위해서 SLA, SLO와 AI Agent로 Application을 기준으로 어떻게 측정 할 것인지?
10. Azure나 GCP 같은 경우에는 30초로 AWS보다 더 짧은데, 어떻게 해결 할 것인지
11. aws에서 제공하는 API가 많고, 이번 주제에서도 활용할 수 있는 API가 많을텐데 찾아보기

## 2. 항목 분류

| # | 한 줄 요약 | 성격 | 겹치는 항목 | 정식 주제가 정해져야 답할 수 있나 |
|---|---|---|---|---|
| 1 | 상태 유무에 따른 차이 | 연구 질문 | 6 | 아니오 |
| 2 | 회수 알림 시점 예측 | 연구 질문 | 4 | 아니오 |
| 3 | 2분 안에 대체 용량 확보 | 설계 | 2 | 일부 (상태 크기에 달림) |
| 4 | 비용과 중단 빈도 데이터 활용 | 설계·측정 | 5, 8 | 아니오 |
| 5 | 참고할 지표 | 측정 | 4, 8, 9, 11 | 일부 |
| 6 | 100% Spot이 효율적인가 | 연구 질문 | 1 | 아니오 |
| 7 | On-Demand 기준선을 어떻게 얻나 | 측정 방법 | 9 | 일부 |
| 8 | Spot 비용 예측 | 측정·도구 | 4 | 실험 규모가 정해져야 함 |
| 9 | SLA·SLO로 가용성 측정 | 측정 방법 | 5, 7 | **예** |
| 10 | 30초 알림 클라우드 대응 | 설계·범위 | 2, 3 | 아니오 |
| 11 | 쓸 수 있는 AWS API | 도구 | 4, 5, 8 | 아니오 |

## 3. 항목별 정리

### 3.1 Stateful과 Stateless의 차이 (원문 1)

- 아는 것
  - AWS는 stateless 컨테이너·웹·배치에 price-capacity-optimized 할당 전략을 권장하고, 재시작 비용이 큰 워크로드는 체크포인트를 구현하라고 합니다 (`aws-spot-facts.md` 4절).
  - Stateless는 회수되면 진행 중이던 요청만 실패하고 재시도로 끝납니다. SkyServe는 클라이언트 재시도를 쓰고 요청 실패율 0.34~0.62%를 보고했습니다 (LLM 70B 서빙, 약 22시간, A등급, `research/skyserve-spothedge-eurosys25.md`).
  - Stateful은 상태를 잃으면 처음부터 다시 합니다. 처음부터 재시작하면 작업 완료 시간이 최대 1.67배(SWE-Bench)였고, 대화 기록만 복구하면 Terminal-Bench 작업의 8~13%만 성공했습니다 (Crab, arXiv 심사 전, B등급, `research/crab-agent-sandbox-cr-2026.md`).
- 정리: stateless의 문제는 **용량**이고, stateful의 문제는 **용량 + 2분 안의 상태 이전**입니다. stateful에서는 상태 크기가 2분 예산을 좌우합니다.
- 확정(2026-10-16): 이 질문이 곧 연구 질문입니다. 상태 보존 수준에 "수준 0: 상태를 버리고 처음부터 재실행"을 대조군으로 더해 수준 0~3을 비교하면, stateless식 대응과 stateful 대응의 차이가 수치로 나옵니다.

### 3.2 회수 알림 시점을 예측할 수 있나 (원문 2)

- 아는 것
  - AWS는 회수가 언제든 일어날 수 있고, 알림이 오기 전에 회수될 수도 있다고 적고 있습니다 ([Prepare for interruptions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/prepare-for-interruptions.html), documents 밖).
  - Rebalance recommendation은 2분 알림보다 먼저 올 수 있지만 보장되지 않고, 동시에 올 수도 있습니다 (`aws-spot-facts.md` 2절).
  - SPS와 회수 빈도 점수가 둘 다 높아도 24시간 안에 14.71%가 한 번 이상 회수됐습니다. 과거 이력으로 만든 단순 예측 모델은 Accuracy 0.73이었습니다 (SpotLake, 503건, 단일 인스턴스 요청, 2022, A등급).
  - Ding-Dong Ditch는 3~60분 뒤 **풀의 가용성 상태**를 F1 약 0.85~0.90으로 예측했습니다. 회수 시각을 맞힌 것은 아니고, 평가는 배치 분석(TPC-DS) 시뮬레이션입니다 (arXiv 심사 전, B등급).
  - 같은 인스턴스 타입·AZ의 동시 회수는 92.9%가 3분 안에 일어났습니다 (Ding-Dong Ditch, 실제 회수 2,635건).
- 정리: 회수 시각을 정확히 예측한다는 근거는 공식 문서에도 논문에도 없습니다.
- 확정(2026-10-16): 질문을 "2분보다 일찍 위험 신호를 얻을 수 있나"로 바꾸면 측정할 수 있습니다.
  - 신호 1: rebalance recommendation의 실제 리드 타임. FIS는 실험 시작 즉시 rebalance recommendation을 보내므로(5절) FIS로는 측정할 수 없고, 자연 회수에서만 측정됩니다.
  - 신호 2: **같은 풀의 첫 회수를 나머지 노드의 조기 경보로 쓰기.** 한 노드가 알림을 받으면 같은 타입·AZ의 다른 노드도 3분 안에 회수될 가능성이 높으므로, 그 노드들의 상태를 미리 저장합니다. 이것으로 벌 수 있는 시간을 재는 것은 연구 요소 후보입니다. **(2026-10-25 변경: 선생님 피드백으로 지금은 하지 않고 업그레이드 후보로 내림, `SongmhT.md` 참고)**
  - 신호 3: SPS를 주기적으로 조회해 점수가 낮은 풀을 피하기.

### 3.3 2분 안에 대체 인스턴스를 띄우려면 (원문 3)

- 아는 것
  - SkyServe 측정에서 Llama-2-7b(vLLM) 복제본 하나를 띄우는 데 183초가 걸려 2분을 넘었습니다 (AWS, 이미지 사전 설치, GPU, A등급).
  - CPU 컨테이너 호스트가 뜨는 데 걸리는 시간은 `documents/`에 없습니다. 직접 재야 합니다.
  - EC2 Auto Scaling warm pool은 mixed instances 그룹의 Spot을 지원하지 않습니다 (5절).
  - AWS 권장: Auto Scaling 그룹으로 대체 인스턴스를 자동 기동, 필요한 소프트웨어를 넣은 AMI 사용 (5절). Capacity Rebalancing을 켜면 rebalance recommendation 때 대체 Spot을 미리 띄웁니다 (`aws-spot-facts.md` 2절).
  - SkyServe는 Spot 복제본을 필요보다 몇 개 더 띄워 두는 방식(과잉 프로비저닝)으로 회수 충격을 흡수했고, 회수가 리전 간에 분산돼 있으면 여분이 적어도 충분했다고 보고했습니다.
- 정리: 2분 안에 새 인스턴스를 띄우는 것은 보장할 수 없습니다.
- 확정(2026-10-16): 2분은 새 인스턴스를 기다리는 데 쓰지 않고, **상태를 내구성 있는 저장소(S3·EBS)로 내보내는 데** 씁니다. 복원할 곳은 다른 풀에 미리 떠 있는 여분 호스트이거나, 나중에 뜨는 대체 호스트입니다. 대체 호스트의 기동 시간은 "복구 지연" 지표로 따로 잽니다.

### 3.4 비용 계산과 중단 빈도 데이터 활용 (원문 4)

- 아는 것
  - Spot Instance Advisor는 회수 빈도를 5개 구간(<5%, 5~10%, 10~15%, 15~20%, >20%)으로 보여 주며, 지난달 기준입니다 (SpotLake 논문 인용, AWS 페이지 미확인, `aws-spot-facts.md` 6절).
  - 공식 API는 이번 검색에서 찾지 못했습니다. 비공식 도구(예: [spotinfo](https://github.com/alexei-led/spotinfo))는 있습니다 (documents 밖, 미검증).
  - SpotLake: 가격 이력은 가용성 예측에 쓸모가 없습니다. SPS와 회수 빈도가 서로 다른 신호를 주면 SPS를 우선합니다 (A등급).
  - 비교 참고: Azure는 시간당 eviction rate(최근 7일 기준)를 포털과 Resource Graph로 제공합니다 (5절).
- 확정(2026-10-16): 기대 비용 모델을 만듭니다.
  - 기대 비용 = Spot 단가 × 실행 시간 + 예상 회수 횟수 × 회수 1회 비용
  - 회수 1회 비용 = 다시 실행한 시간 + 상태 저장·전송 비용 + 대체 호스트 기동 시간
  - 회수 빈도 구간은 정확한 확률로 쓰지 말고, 인스턴스 타입 후보를 거르는 데만 씁니다.
  - AWS가 회수하면 Linux 인스턴스는 첫 1시간 요금이 없습니다 (5절). 짧은 실험이 많으면 비용 계산에 반영합니다.

### 3.5 참고할 지표 (원문 5)

- 정리: 이 항목은 두 종류로 나눠야 쓰기 쉽습니다.
  - **입력 지표 (Spot 시장 신호):** SPS, 회수 빈도 구간, 가격 이력, rebalance·회수 이벤트 → 3.4, 3.8, 3.11과 같은 내용입니다.
  - **출력 지표 (서비스 품질):** 아래 표처럼 논문들이 쓴 지표를 참고합니다. 무엇을 고를지는 3.9와 정식 주제에 달려 있습니다.

| 지표 | 쓴 자료 | 조건 |
|---|---|---|
| 요청 실패율, P50/P90/P99 지연, On-Demand 대비 비용 | SkyServe (A) | LLM 서빙, 실제 회수 |
| 복구 정확도, 작업 완료 시간 증가율(중앙값·p95), 체크포인트 트래픽 | Crab (B) | 에이전트 샌드박스, 회수 시뮬레이션 |
| P99 지연, 토큰당 비용 | SpotServe (A) | LLM 서빙, 회수 이력 재생 |
| 예측 F1, 잃어버린 연산 시간 | Ding-Dong Ditch (B) | 배치 분석 시뮬레이션 |

- 확정(2026-10-16): 이 항목은 3.4와 3.9에 흡수시키고 따로 두지 않습니다.

### 3.6 100% Spot이 효율적인가 (원문 6)

- 아는 것
  - Skyscanner는 사용자용 웹 플랫폼을 Spot 전용 Kubernetes에서 운영했습니다 (2020, B등급).
  - 스캐터랩은 On-Demand 최소 개수를 유지했습니다 (2023, B등급).
  - SkyServe (GPU LLM 서빙 기준, A등급)
    - On-Demand 1대를 고정으로 두면 순수 Spot보다 비용이 1.56배였습니다 (g5.48xlarge, Spot 확보가 쉬운 시기).
    - 단일 리전 순수 Spot은 요청의 49.4%가 실패하거나 시간 초과됐습니다.
    - 필요할 때만 On-Demand로 넘기는 동적 fallback은 SkyServe 전체 비용의 26~31%였지만 가용성에 필요했습니다.
  - CPU Spot은 확보 가능 시간이 95.6~99.9%로, GPU(16.7~90.4%)보다 훨씬 높았습니다 (SkyServe, CPU는 GCP 트레이스).
- 정리: 정답은 없고, 강한 근거는 GPU 기준이라 CPU 샌드박스에 그대로 쓸 수 없습니다.
- 확정(2026-10-16): "100%"를 먼저 정의합니다.
  - (a) 항상 Spot만 사용
  - (b) 평소에는 Spot만 사용하고, Spot을 확보하지 못할 때만 On-Demand로 넘김
  - 두 방식을 같은 회수 시나리오에서 비교하면 이 질문 자체가 결과가 됩니다. 어느 쪽을 "Spot 100%"라고 부를지는 제안서에 명시합니다.

### 3.7 On-Demand 기준선을 어떻게 얻나 (원문 7)

- 아는 것
  - AWS 공식 문서의 Spot과 On-Demand 차이 표에는 기동 방식, 용량, 가격, rebalance 신호, 회수만 있고 성능 항목은 없습니다 (5절).
  - AWS 한국 블로그(2020)는 "스팟 인스턴스는 EC2로 인해 중단될 수 있다는 점(2분 사전 공지)을 제외하고 온디맨드 인스턴스와 동일합니다"라고 적었습니다 (documents 밖: `company-cases/aws-skyscanner-mobileye-capacity-optimized.md`의 원문 블로그에 있는 문장, B등급).
  - On-Demand 비용은 가격표로 계산할 수 있어 배포하지 않아도 됩니다.
  - 논문들은 실제 회수 트레이스 재생(SkyServe, Ding-Dong Ditch) 또는 무작위 회수 시뮬레이션(Crab)으로 여러 시나리오를 평가했습니다.
- 확정(2026-10-16)
  - **성능 기준선 = 같은 인스턴스 타입에서 회수 없이 실행한 결과.** On-Demand를 따로 오래 띄울 필요가 없습니다.
  - **비용 기준선 = On-Demand 단가로 계산한 값.**
  - 검증: On-Demand 1대와 Spot 1대에서 같은 재생을 짧게 한 번씩 돌려 성능 차이가 없는지만 확인합니다.
  - 시뮬레이터는 회수 시점·횟수를 넓게 훑는 데 쓰고, 주요 시나리오는 AWS에서 실측합니다. 이 주제의 새로움은 실측에 있으므로 시뮬레이션만으로 끝내지 않습니다.

### 3.8 Spot 비용을 미리 예측할 수 있나 (원문 8)

- 아는 것 (5절)
  - `describe-spot-price-history`로 최근 90일 가격을 인스턴스 타입·AZ·OS별로 볼 수 있습니다.
  - 가격은 장기 수급에 따라 서서히 바뀌고, 현재 최저가 페이지는 5분마다 갱신됩니다.
  - 숨은 비용: 퍼블릭 IPv4 주소 1개당 $0.005/시간, NAT Gateway $0.045/시간 + 처리 1GB당 $0.045 (us-east-2 예시), FIS $0.10/액션-분, EKS 클러스터 $0.10/시간 (`projectPlan.md` 2.4.1), 그 밖에 EBS·S3·데이터 전송
- 정리: 가격 변동이 느려서 단기 예측은 어렵지 않습니다. 어려운 것은 실험 규모(호스트 수 × 시간 × 회수 횟수)를 먼저 정하는 일입니다.
- 확정(2026-10-16): 실험 계획표를 먼저 만들고 `projectPlan.md` 2.5.1(시세 조회)과 2.5.2(Pricing Calculator)에서 계산합니다. 리전 선택(예: 서울 대 미국 동부)에 따라 단가가 달라지므로 리전도 함께 정합니다.

### 3.9 SLA·SLO로 가용성을 어떻게 재나 (원문 9)

- 용어 ([Google SRE Book, Service Level Objectives](https://sre.google/sre-book/service-level-objectives/), documents 밖)
  - SLI: 서비스 수준의 한 측면을 정량으로 정의한 지표
  - SLO: SLI로 잰 서비스 수준의 목표 값이나 범위
  - SLA: 사용자와 맺은 계약으로, SLO를 지키거나 못 지켰을 때의 결과(보상 등)가 들어 있음. 결과가 정해져 있지 않으면 SLO입니다.
- 아는 것
  - AWS Compute SLA는 리전 99.99%, 인스턴스 99.5%를 약속하며 페이지에 Spot에 대한 언급이 없습니다. Spot에 적용되는지는 확인하지 못했습니다 (5절).
  - Azure는 Spot VM에 SLA가 없다고 명시합니다 (5절).
- 정리: 이 프로젝트가 정하는 것은 SLA가 아니라 **SLI와 SLO**입니다.
- 확정(2026-10-16): SLI 후보입니다. 정식 주제가 정해진 뒤 2~3개를 고릅니다.
  - 작업 생존율: 회수가 일어나도 처음부터 다시 하지 않고 끝난 작업의 비율
  - 작업 정확도: 끝난 작업의 최종 상태(파일, 테스트 결과)가 회수 없는 실행과 같은 비율
  - 회수 1회당 복구 시간
  - 작업 완료 시간 p95 증가율 (회수 없는 실행 대비)
  - 명령(도구 호출) 1회 응답 지연 p95: ms 단위라 CFP의 p95 개념을 살릴 수 있는 후보
  - 가용성을 요청 기준(성공한 명령 수 / 전체)으로 잴지 시간 기준으로 잴지도 정해야 합니다.

### 3.10 30초 알림인 Azure·GCP는 어떻게 하나 (원문 10)

- 아는 것 (5절)
  - Azure: 회수 30초 전에 알림, best effort, Spot VM에는 SLA 없음
  - GCP: 전제가 바뀌었습니다. 알림 시간을 사용자가 정할 수 있고(기본 0초, 120초 권장), 그 뒤의 종료 처리 시간은 best effort이며 최대 30초입니다.
  - AWS도 2분 알림은 best effort이고, 알림 없이 회수될 수 있습니다.
- 정리: 어느 클라우드든 "알림이 짧거나 없는 경우"를 설계에 넣어야 합니다.
- 확정(2026-10-16)
  - 다중 클라우드 구현은 Out of scope로 둡니다. 지원받는 예산이 AWS뿐입니다.
  - 대신 **알림 시간을 실험 변수로 둡니다: 120초 / 30초 / 0초.**
    - FIS는 회수 대기 시간을 2~15분으로만 정할 수 있습니다(5절). 그래서 30초는 회수 처리 코드가 알림을 받고 90초를 기다렸다 시작하도록 흉내 냅니다.
    - 0초는 인스턴스를 직접 종료해 재현합니다. AWS 문서도 이 방식의 테스트를 권합니다.
  - 이렇게 하면 결과를 다른 클라우드에도 적용할 수 있다고 주장할 근거가 생깁니다.

### 3.11 쓸 수 있는 AWS API (원문 11)

| 역할 | API·기능 | 하는 일 | 확인 |
|---|---|---|---|
| 사전 선택 | `DescribeSpotPriceHistory` | 최근 90일 Spot 가격 | 5절 |
| 사전 선택 | `GetSpotPlacementScores` (SPS) | 리전·AZ별 확보 가능성 1~10점 | `aws-spot-facts.md` 3절 |
| 사전 선택 | Spot Instance Advisor | 회수 빈도 구간, 절감률 | 공식 API 미확인 |
| 사전 선택 | `GetInstanceTypesFromInstanceRequirements`, `DescribeInstanceTypes` | 속성으로 인스턴스 타입 후보 뽑기 | 이번에 문서 미확인 |
| 사전 선택 | Price List Query API (`GetProducts`) | On-Demand 단가 조회 | 이번에 문서 미확인 |
| 실행 | `CreateFleet` (EC2 Fleet), Auto Scaling 그룹 mixed instances + Capacity Rebalancing | 여러 풀에 걸쳐 Spot 요청, 자동 대체 | `aws-spot-facts.md` 2·4절 |
| 감지 | EventBridge `EC2 Spot Instance Interruption Warning`, `EC2 Instance Rebalance Recommendation` | 회수 알림·위험 신호 받기 | `aws-spot-facts.md` 1·2절 |
| 감지 | IMDS `spot/instance-action`, `events/recommendations/rebalance` | 인스턴스 안에서 5초마다 확인 | `aws-spot-facts.md` 1·2절 |
| 실험 | FIS `aws:ec2:send-spot-instance-interruptions` | 회수 주입 (대기 2~15분) | 5절 |
| 실험 | FIS `aws:ec2:terminate-instances` 또는 직접 종료 | 알림 없는 회수 재현 | 5절 (직접 종료 권장 문구) |
| 비용 | Cost Explorer, CUR 2.0, AWS Budgets·Budgets actions | 실제 비용 확인, 예산 알림·차단 | CFP 8장 링크 |
| 할당량 | Service Quotas (Spot vCPU 할당량) | 띄울 수 있는 Spot vCPU 확인·증설 요청 | 5절 |

## 4. 원문에 없는 고려 사항 (확정 2026-10-16)

- A. **동시 회수.** CFP 문구에 "여러 AZ에서 Spot 회수가 동시에 일어나도 견뎌야 한다"가 있는데 원문 목록에는 없습니다. 같은 타입·AZ는 3분 안에 함께 회수되는 경우가 92.9%였고(Ding-Dong Ditch, B), 같은 리전 존들도 함께 회수되는 경향이 있습니다(SkyServe, GPU 기준, A).
- B. **Spot vCPU 할당량.** Standard 계열(A, C, D, H, I, M, R, T, Z) Spot 요청의 기본 할당량이 **5 vCPU**입니다(조정 가능, 사용량에 따라 자동 조정, 5절). 호스트 여러 대로 실험하려면 미리 증설 요청이 필요할 수 있습니다. SPS의 목표 용량 상한도 최근 Spot 사용량을 기준으로 정해집니다(`aws-spot-facts.md` 3절).
- C. **실험 기간에 자연 회수가 거의 없을 수 있음.** CPU Spot은 확보 가능 시간이 높아(95.6~99.9%, SkyServe, GCP 트레이스) 몇 주 실험에서 자연 회수를 못 볼 수 있습니다. 실험은 FIS 중심으로 설계하고, 자연 회수는 관측되면 따로 기록합니다. 포트폴리오에는 "FIS로 AWS의 실제 회수 절차(알림·종료)를 썼지만 회수 시점은 실험자가 정했다"고 정확히 씁니다.
- D. **상태 저장 위치와 2분 안의 전송량.** CRIU 프로세스 체크포인트는 동시성이 높을 때 128MB 7.8초 ~ 1GB 47.4초가 걸렸고, 대역폭 500MB/s인 공유 EBS에서는 체크포인트 트래픽이 병목이 됐습니다(Crab, c6id.32xlarge, B).
- E. **대체 용량 확보 실패.** 단일 리전에서 GPU Spot을 전혀 못 구한 시간이 33.1%인 트레이스가 있었고(SkyServe, A), 단일 노드 SPS가 최고점인 타입도 50대를 한 번에 요청하면 성공률이 약 20%였습니다(HPDC'25, A). 이 프로젝트 규모(호스트 몇 대, CPU)에서는 덜하겠지만 측정해 볼 항목입니다.
- F. **격리와 보안.** 에이전트 샌드박스는 신뢰할 수 없는 명령을 실행합니다. 컨테이너는 격리가 약하고, microVM(Firecracker)은 EC2 nested virtualization이 필요해 지원 인스턴스 타입이 7·8세대 Intel 계열(M7i, C7i, R7i, C8i 등)로 좁아집니다 (documents 밖, [AWS Nested virtualization](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-ec2-nested-virtualization.html), 2026-10-13 확인).

## 5. 이번에 공식 문서로 확인한 사실 (2026-10-16)

| 사실 | 출처 |
|---|---|
| warm pool은 mixed instances 그룹의 Spot을 지원하지 않음. mixed instances policy는 On-Demand만으로 설정해야 함 | [Warm pools](https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-warm-pools.html) |
| FIS 회수 주입: 회수 대기 2~15분, 액션 시작 즉시 rebalance recommendation 전송, 회수 2분 전에 회수 알림 | [FIS actions reference](https://docs.aws.amazon.com/fis/latest/userguide/fis-actions-reference.html) |
| FIS 요금: 액션 1분당 $0.10 (단일 계정) | [FIS pricing](https://aws.amazon.com/fis/pricing/) |
| 알림이 오기 전에 회수될 수 있으므로, On-Demand 인스턴스를 직접 종료해 예고 없는 중단도 테스트하라고 권장 | [Prepare for interruptions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/prepare-for-interruptions.html) |
| Spot 가격 이력 최근 90일, 현재 최저가 페이지 5분마다 갱신, 가격은 장기 수급에 따라 서서히 변함 | [Spot price history](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances-history.html), [Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html) |
| Spot과 On-Demand의 차이 표에 성능 항목은 없음 (기동, 용량, 가격, rebalance, 회수만 다름) | [Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html) |
| EC2가 회수한 경우 Linux는 첫 1시간 요금 없음, 그 뒤는 초 단위 과금 | [Billing for interrupted Spot](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/billing-for-interrupted-spot-instances.html) |
| Standard 계열 Spot 요청 기본 할당량 5 vCPU, 조정 가능, 사용량에 따라 자동 조정 | [Spot Instance quotas](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-limits.html) |
| 퍼블릭 IPv4 $0.005/시간(사용 중·유휴 동일), NAT Gateway $0.045/시간 + 1GB당 $0.045 (us-east-2 예시) | [VPC pricing](https://aws.amazon.com/vpc/pricing/) |
| AWS Compute SLA: 리전 99.99%, 인스턴스 99.5%. 페이지에 Spot 언급 없음 | [Compute SLA](https://aws.amazon.com/compute/sla/) |
| Azure Spot: 30초 알림 best effort, SLA 없음, 시간당 eviction rate를 Resource Graph로 제공 | [Azure Spot VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/spot-vms) |
| GCP Spot: 알림 시간 사용자 설정(기본 0초, 120초 권장), 종료 처리 best effort 최대 30초 | [GCP Spot VMs](https://docs.cloud.google.com/compute/docs/instances/spot) |

## 6. 확정 이후 남은 것

- 2026-10-16 채택: 3.1 수준 0(대조군) 추가, 3.2 풀 단위 조기 경보를 연구 요소로, 3.3 2분은 상태 내보내기에 사용, 3.4 기대 비용 모델, 3.5를 3.4·3.9에 흡수, 3.6 (a)·(b) 둘 다 비교, 3.7 기준선 얻는 방법, 3.8 실험 계획표 먼저 작성, 3.9 SLO로 정의, 3.10 알림 시간(120초/30초/0초)을 실험 변수로, 4절 A~F
- 정식 주제(AI Agent의 범위)가 정해져야 구체화할 것: 3.9 SLI 2~3개 선택, 3.7의 비교 대상, 3.3의 상태 크기
- 아직 고를 것: 3.6의 (a)·(b) 중 제안서에서 "Spot 100%"라고 부를 쪽 (비교는 둘 다 함)
- 새 작업이 생기는 항목은 `projectPlan.md`에 작업으로 옮깁니다.
- 2026-10-25 선생님 피드백 반영(`SongmhT.md`): 3.2의 풀 단위 조기 경보와 SPS 신호 분석은 지금 하지 않고 업그레이드 후보로 내렸습니다. 인스턴스 선택은 "요구 사양 중 가장 안정적일 것 같은 것"을 고르는 단순한 방식으로 시작합니다. 나머지 확정 항목은 그대로입니다.
