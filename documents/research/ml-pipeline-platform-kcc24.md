---
title: 최적의 성능 및 비용을 고려한 머신러닝 파이프라인 배포 플랫폼
notion_title: (노션 자료 아님, 사용자가 2026-10-16에 제공한 PDF)
source_url: https://github.com/kookmin-sw/capstone-2024-12 (논문 본문에 적힌 구현 저장소)
source_type: 국내 학술대회 논문 (2024 한국컴퓨터종합학술대회 논문집, 621~623쪽, 3쪽)
authors: 송무현, 정승우, 김규민, 김유림, 문지훈, 박정명 (국민대학교)
published: 2024 (논문 본문 기준 2024년 5월 현재. 학술대회 날짜는 확인하지 못함)
accessed: 2026-10-16
reliability: B (3쪽 국내 학술대회 논문이고 심사 수준은 확인하지 못함. 비용 수치는 논문이 시뮬레이션이라고 밝힘)
tags: [ml-pipeline, spot, checkpoint-restore, serverless, karpenter, ray, instance-selection, korea, advisor-paper]
---

# 머신러닝 파이프라인 배포 플랫폼 (KCC 2024)

## 한 줄 요약

머신러닝 모델의 학습과 추론을 Spot 인스턴스와 서버리스로 올려 비용을 줄이는 웹 플랫폼입니다. 학습은 에포크마다 체크포인트를 저장해 회수에 대비하고, 추론은 SageMaker·Bedrock 등과 **시뮬레이션**으로 견줘 시간당 비용이 최대 77% 낮다고 보고합니다. 실제 회수를 일으킨 가용성·지연 측정은 없습니다.

## 저자와 우리 프로젝트의 관계

- 제1저자 송무현은 사용자의 선생님(코치)이고, CFP 작성자와 같은 분입니다.
- 같은 저자가 `research/`의 [HPDC'25 포스터](multinode-sps-hpdc25.md), [Ding-Dong Ditch](ding-dong-ditch-2026.md)에도 저자로 있습니다 (사용자 확인, 2026-10-16). 김규민도 Ding-Dong Ditch의 저자입니다.
- (해석, 원문 아님) 선생님이 Spot 가용성 신호(SPS, 회수 예측)와 ML 워크로드에 관심이 있다는 근거입니다. 선생님이 이 프로젝트에 무엇을 기대하는지는 이 논문만으로는 알 수 없습니다.

## 핵심 내용

### 플랫폼 흐름 (논문 2.1, 그림 2)
1. 사용자가 웹에서 모델과 데이터셋을 올림
2. 학습 코드와 에포크 수만 주면 가격·비용이 최적화된 클러스터에서 학습
3. 학습된 모델을 평가해 확장 가능한 클러스터에 배포하고 엔드포인트 URL을 받음

### 인스턴스 선출 알고리즘 (논문 2.2)
1. 가용한 컴퓨팅 자원 목록 추출
2. 각 유형의 Spot 가격 수집
3. 가격과 성능(가속기 FLOPS를 성능으로 가정)을 0~1로 정규화
4. 선형 보간으로 최종 점수 계산
5. 점수순으로 정렬해 사용할 유형 집합을 고름

추론용 소형 모델은 서버리스에 배포하고, 모델 측정 API가 실제 추론으로 얻은 자원 사용량을 DB에 기록해 배포 환경을 추천합니다.

### 추론 아키텍처 (논문 2.3)
- 서버리스(CPU만으로 충분한 경우)와 Spot 인스턴스(생성형 AI처럼 가속기가 필요한 경우)를 선택합니다. 논문은 2024년 5월 기준 상용 서버리스가 GPU를 지원하지 않는다고 적었습니다.
- 회수 알림이 오면 즉시 대체 Spot 자원을 만들고, 가용성 유지를 위해 On-Demand를 최소한만 배치합니다.

### 학습 아키텍처 (논문 2.4, 그림 3)
- 데이터 병렬 분산 학습을 씁니다.
- 회수 흐름: Spot 회수 이벤트 → Event Handler → 체크포인트 저장소 → Node Scaler가 새 Spot 요청(Recommender API가 노드풀 갱신) → 학습 작업 복원
- **에포크마다 체크포인트**를 저장하고, 회수되면 직전 에포크 체크포인트에서 이어서 학습합니다.

## 실험 조건과 핵심 수치

| 항목 | 값 | 조건 |
|---|---|---|
| 체크포인트 오버헤드 | 에포크 약 9분 38초 중 체크포인트 평균 약 2.97초 (약 0.5%, 제 계산) | g4dn.xlarge(T4) 1대, GPT-2, Simple Wikipedia 4,000개, 배치 4. **실제 회수 없음** |
| 추론 시간당 비용 | 제안 플랫폼 $13.1 대 SageMaker On-Demand $37.7, 절약 플랜 1년 $27.8, 3년 $19.2 | Llama-2-13B, 시간당 입력 8,730,000·출력 1,166,400 토큰. **시뮬레이션** (논문 표 1) |
| 같은 표 | Bedrock $57.2, Azure ML On-Demand $32.8(1년 $28.6, 3년 $17.2), GCP Vertex AI $33.8 | 같은 시나리오 |
| "최대 77% 절감" | Bedrock($57.2) 대비. SageMaker On-Demand 대비 약 65%, SageMaker 3년 절약 플랜($19.2) 대비 약 32% (제 계산) | 논문 3.1절 |

## 구현 저장소 확인 (2026-10-16, master 브랜치, 내려받은 일부 파일만 읽음)

- 저장소: `kookmin-sw/capstone-2024-12`. 마지막 푸시 2024-05-23. **라이선스 없음**(GitHub API 기준). 템플릿 저장소의 fork입니다. 구성은 Jupyter Notebook, Python, JavaScript, Terraform(HCL) 순입니다.
- 읽은 범위: `recommend/`, `automation/karpenter_node_pool_deploy`, `automation/deploy_train`, `automation/llama_train_deploy`, `IaC/kubernetes_cluster`, `train/ray`, `README.md` 등. 백엔드·프론트엔드와 다른 자동화 폴더는 읽지 않았습니다.

| 논문의 설명 | 코드에서 확인한 것 |
|---|---|
| 에포크마다 체크포인트, 회수 시 복원 | Ray Train이 에포크마다(주석 "매 에포크 checkpoint") model·optimizer·epoch를 저장하고 `storage_path`로 **S3**에 보냄. `FailureConfig(max_failures=-1)`로 실패 시 무제한 재시도하며, 재시작하면 `train.get_checkpoint()`로 복원. 체크포인트는 2개만 유지 |
| 회수 알림을 받는 Event Handler | 직접 만든 핸들러 코드는 읽은 범위에서 찾지 못함. 대신 Karpenter 0.35.1에 `interruptionQueue`(SQS + EventBridge 규칙)를 설정함. (해석, 원문 아님) Karpenter의 내장 회수 처리를 쓴 것으로 보임 |
| 인스턴스 선출 점수 | `family.py`: 점수 = (1−α)×가격 점수 + α×성능 점수, **α=0.2**, 가격 점수 = 1 − 가격/최대가격. 가격은 `describe_spot_price_history`에서 가져옴. **SPS·회수 빈도는 쓰지 않음** |
| (논문에 없음) | `per_single_node/recommender.py`는 SpotLake 데이터에서 `SPS >= 2`로 거른 뒤 가장 싼 유형을 고름. 다만 읽은 코드에서 데이터 경로가 `rawdata/aws/2024/03/10`으로 **고정**(실시간 아님)이고 회수 빈도(IF) 열은 버림 |
| On-Demand 최소 배치 | Ray head와 job submitter는 `ray-ondemand-nodepool`(On-Demand, CPU t3·m4·m5·m6 계열), GPU worker는 추천된 Spot 노드풀에서 실행. 즉 **제어부는 On-Demand, 작업부는 Spot** |
| 비용 77%, 체크포인트 2.97초 | 재현 코드·결과는 읽은 범위와 README에서 찾지 못함 |

## 프로젝트에 주는 것

- **우리가 확정한 구조와 같은 틀입니다.** 알림 → 저장 → 대체 용량 → 복원, 가격·성능 기반 인스턴스 선택, On-Demand 최소 배치가 모두 들어 있어 "후속 연구"로 설명하기 쉽습니다 (`../spotConsiderations.md` 3.1, 3.3, 3.6).
- **논문이 비워 둔 것이 우리 프로젝트의 측정 대상입니다.**
  - 실제 회수 아래에서의 가용성·지연·복구 시간(논문은 비용 시뮬레이션과 회수 없는 체크포인트 오버헤드뿐)
  - "즉시 대체 Spot 생성"이 2분 안에 되는지. SkyServe는 GPU 복제본 하나를 띄우는 데 183초를 쟀습니다(A등급, [skyserve-spothedge-eurosys25.md](skyserve-spothedge-eurosys25.md)).
  - 인스턴스 선택에 가용성 신호를 넣는 것. 논문 알고리즘과 `family.py`는 가격·성능만 보지만, SpotLake는 가격 이력이 가용성 예측에 쓸모없다고 했습니다(A등급, [spotlake-iiswc22.md](spotlake-iiswc22.md)).
- **논문의 "Spot 활용"은 100%가 아닙니다.** 제어부(Ray head)가 On-Demand라서, 우리의 "Spot 100%" 정의 (a)·(b)(`../spotConsiderations.md` 3.6)를 정할 때 비교 대상으로 쓸 수 있습니다.
- (해석, 원문 아님) 학습에서의 체크포인트(에포크 단위, 외부 저장소)는 우리 상태 보존 수준 중 "저장소에 기록하는" 쪽에 가깝고, 에이전트 샌드박스에서는 파일과 프로세스까지 필요하다는 점이 다릅니다([crab-agent-sandbox-cr-2026.md](crab-agent-sandbox-cr-2026.md)).

## 한계와 주의점

- 3쪽 논문이라 방법과 실험이 짧습니다. 비용은 시뮬레이션이고, 체크포인트 오버헤드는 인스턴스 1대에서 쟀습니다.
- 77%는 가장 비싼 비교 대상(Bedrock) 기준의 최대값입니다. 비교 대상 중에는 토큰당 과금하는 서비스와 시간당 과금하는 인스턴스가 섞여 있어 조건이 같지 않습니다 (해석, 원문 아님).
- 논문 수치의 기준이 된 Spot 가격 시점과 리전은 논문에 적혀 있지 않습니다.
- 저장소는 라이선스가 없어 코드 재사용 가능 여부가 불분명합니다. 참고용으로만 보고, 쓰려면 선생님께 확인이 필요합니다.
- 코드는 내려받아 읽기만 했고 실행하지 않았습니다.

## 인용할 때 쓰는 표현

- "송무현 외(KCC 2024)는 학습은 에포크마다 체크포인트를 저장하고 추론은 서버리스와 Spot을 혼합하는 ML 파이프라인 플랫폼을 제안했고, 추론 시간당 비용이 SageMaker·Bedrock 등과의 시뮬레이션 비교에서 최대 77% 낮다고 보고했다. 다만 실제 회수 환경의 가용성·지연은 측정하지 않았다."

## 관련 문서

- [multinode-sps-hpdc25.md](multinode-sps-hpdc25.md), [ding-dong-ditch-2026.md](ding-dong-ditch-2026.md) — 같은 저자의 Spot 가용성 연구
- [spotlake-iiswc22.md](spotlake-iiswc22.md) — 가격이 아니라 SPS를 보라는 근거
- [skyserve-spothedge-eurosys25.md](skyserve-spothedge-eurosys25.md) — 대체 인스턴스 기동 시간, 동적 On-Demand fallback
- [crab-agent-sandbox-cr-2026.md](crab-agent-sandbox-cr-2026.md) — 에이전트 상태 보존 수준
- [../spotConsiderations.md](../spotConsiderations.md) — 3.1(상태 차이), 3.3(2분 안의 대체), 3.6("100%"의 정의)
