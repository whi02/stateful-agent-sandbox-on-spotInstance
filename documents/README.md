# Spot 인스턴스 연구 자료

Spot 100% 마이그레이션 research 프로젝트의 근거 자료입니다. 사용자가 노션 "자료" 페이지(프로젝트 | Projects / Spot 100% 인프라 구축 / 자료)에 고른 북마크 12개를 원문으로 다시 읽고 정리했습니다. 이후 사용자가 제공한 논문 1편(선생님의 2024 KCC 논문)을 추가했습니다(2026-10-16).

- 정리일: 2026-10-13
- 읽은 방식: PDF는 본문 전체를 텍스트로 뽑아 읽었고, 웹 글은 HTML 원문에서 본문을 뽑아 읽었습니다. 검색 요약만 보고 옮긴 수치는 없습니다.
- AWS 동작 설명은 공식 문서와 대조했습니다([aws-spot-facts.md](aws-spot-facts.md)).

## 폴더 구조

```
documents/
├── README.md              ← 이 파일: 목록, 신뢰도 기준, 핵심 수치, 자료 간 연결
├── aws-spot-facts.md      ← AWS·Karpenter 공식 문서로 확인한 Spot 동작 (다른 자료와 다르면 이것이 우선)
├── introduceDocs.md       ← 문서마다 추가일·작성 이유·내용을 2줄로 소개한 목록 (이 파일만 읽어도 전체를 파악)
├── projectPlan.md         ← 프로젝트 계획과 진행도 (사전 학습 ~ 종료 산출물, 체크박스로 진행 상태 표시)
├── spotConsiderations.md  ← 사용자가 정리한 Spot 고려 사항 11개와 항목별 근거·제안
├── SongmhT.md             ← 송무현 선생님(코치) 피드백 원문 인용, 뜻 풀이, 프로젝트에 미치는 영향
├── proposal.md            ← 제안서 초안 (CFP 7개 항목 + AI 사용 내역, 사용자 작성 칸 표시)
├── company-cases/         ← 노션 분류 "기업 Tech Blog"
└── research/              ← 노션 분류 "학술 자료 (논문)" + 사용자가 제공한 논문 1편
```

## 신뢰도 등급

| 등급 | 기준 |
|---|---|
| A | 동료 심사를 거친 학회 논문, AWS 공식 문서 |
| B | 기업이 직접 쓴 기술 블로그, AWS 공식 블로그의 고객 사례, 실험 방법과 수치를 공개한 arXiv 프리프린트 |
| C | 벤더 마케팅 고객사례, 측정치 없는 데모·튜토리얼, 제품 홍보 글 |

C등급 자료만으로 결론을 내리지 않습니다.

## 문서 목록

### 기준 문서

| 파일 | 내용 | 등급 |
|---|---|---|
| [aws-spot-facts.md](aws-spot-facts.md) | 2분 회수 알림, rebalance recommendation, SPS(1~10), 할당 전략, Karpenter 회수 처리 | A |

### company-cases/ (노션: 기업 Tech Blog)

| 파일 | 노션 제목 | 원문 | 시기 | 등급 | 한 줄 요약 |
|---|---|---|---|---|---|
| [duolingo-spotinst-casestudies.md](company-cases/duolingo-spotinst-casestudies.md) | Duolingo의 Spot instance 사용기 | Spotinst 고객사례 PDF (한컴MDS) | 날짜 없음, 2018~19 추정 | C | Duolingo 65%, Ticketmaster 60~66%, N26 85% 절감. Zalando는 stateful(Cassandra) PoC. SLO 수치 없음 |
| [aws-skyscanner-mobileye-capacity-optimized.md](company-cases/aws-skyscanner-mobileye-capacity-optimized.md) | black Friday 트래픽을 Spot 인스턴스로 버텨보기 | AWS 한국 블로그 | 2020-04 | B | Skyscanner 웹 플랫폼이 Spot 전용 K8s에서 운영. capacity-optimized로 회수 200~300회 → 10~15회 |
| [scatterlab-karpenter-spot.md](company-cases/scatterlab-karpenter-spot.md) | AI Agent GPU Spot으로 컴퓨팅 서비스 비용 감소 | 스캐터랩 기술 블로그 | 2023-03 | B | 루다 AI 서빙 EKS를 Karpenter+NTH+FIS로 Spot 운영. 정량 결과 없음 |

### research/ (노션: 학술 자료)

| 파일 | 노션 제목 | 원문 | 시기 | 등급 | 한 줄 요약 |
|---|---|---|---|---|---|
| [spotserve-asplos24.md](research/spotserve-asplos24.md) | stateful 한 LLM 추론에서 상태를 옮기는 방법 | ASPLOS'24 | 2023-11 | A | KV cache를 토큰 단위로 옮겨 Spot에서 LLM 서빙. 비용 −54%, 대신 P99 최대 +90% |
| [skyserve-spothedge-eurosys25.md](research/skyserve-spothedge-eurosys25.md) | 여러 AZ에서 동시에 Spot 회수가 진행되었을 때 대처 방법 | EuroSys'25 | 2025 | A | 같은 리전 존들은 함께 회수됨(5분 내 추가 회수 83~97%). 리전 간 분산 + 동적 On-Demand fallback으로 비용 −41~44%, 실패율 0.34~0.62% |
| [crab-agent-sandbox-cr-2026.md](research/crab-agent-sandbox-cr-2026.md) | AI Agent Spot 인스턴스 활용기 | arXiv | 2026-04 | B | 에이전트 샌드박스 C/R. 대화만 복구 시 8~13% 성공, Crab 100%. Spot 실험은 60초 알림 시뮬레이션 |
| [cedana-live-migration-demo.md](research/cedana-live-migration-demo.md) | Spot 인스턴스 사이에서 stateful 컨테이너를 옮기는 데모 | Cedana 블로그 | 2024-11 | C | GKE 컨테이너 체크포인트·복원 6단계 튜토리얼. 측정치 없음 |
| [spotlake-iiswc22.md](research/spotlake-iiswc22.md) | Spot 회수를 미리 예측해 대응하게 하는 방향 (1/3) | IISWC'22 | 2022 | A | 가격 이력은 쓸모없음. SPS 우선. 이력 기반 예측 Accuracy 0.73 |
| [multinode-sps-hpdc25.md](research/multinode-sps-hpdc25.md) | Spot 회수를 미리 예측해 대응하게 하는 방향 (2/3) | HPDC'25 | 2025-07 | A | 단일 노드 SPS 최고점 타입도 50대 요청 시 성공률 약 20% |
| [ding-dong-ditch-2026.md](research/ding-dong-ditch-2026.md) | Spot 회수를 미리 예측해 대응하게 하는 방향 (3/3) | arXiv | 2026-05 | B | 같은 타입·AZ 동시 회수 92.9%가 3분 내. 60분 앞 예측 F1 약 0.85 |
| [langchain-deep-agents-runtime.md](research/langchain-deep-agents-runtime.md) | 에이전트를 중단 후에도 이어서 실행하는 방식 | LangChain 블로그 | 2026-04 | B/C | 단계별 체크포인트를 PostgreSQL에 저장, 워커가 죽으면 다른 워커가 이어 감. Spot 언급 없음 |
| [ml-pipeline-platform-kcc24.md](research/ml-pipeline-platform-kcc24.md) | (노션 아님, 사용자 제공 PDF) | KCC 2024 (선생님 논문) | 2024 | B | ML 학습·추론을 Spot·서버리스로 운영하는 플랫폼. 학습은 에포크 체크포인트, 추론 비용 −77%(Bedrock 대비, 시뮬레이션). 실제 회수 측정 없음. 저장소 코드는 Ray Train 체크포인트 + Karpenter 인터럽션 큐 |

## 핵심 수치 (조건 포함)

수치를 인용할 때는 오른쪽 조건을 같이 적습니다.

| 주장 | 수치 | 조건 | 문서 |
|---|---|---|---|
| AWS 회수 알림 | 2분 전, best effort | hibernate는 2분 여유 없음 | aws-spot-facts |
| 같은 존 연쇄 회수 | 회수 후 5분 내 추가 회수 83~97% | AWS p3.2xlarge(GPU) 16대, 3개 존, 3주 | skyserve |
| 같은 풀 동시 회수 | 3분 내 92.9%, 1분 내 85% 이상 | 같은 타입·AZ, 회수 2,635건, AWS+Azure | ding-dong-ditch |
| 존 간 회수 상관 | 같은 리전 최대 0.59, 다른 리전 0.03~0.27 | AWS V100, 2개월 | skyserve |
| GPU 대 CPU Spot 확보율 | GPU 16.7~90.4%, CPU 95.6~99.9% | 트레이스별, CPU는 GCP | skyserve |
| 할당 전략 효과 | 회수 200~300회 → 10~15회 | Skyscanner 블랙프라이데이 테스트, 조건 비공개 | aws-skyscanner |
| LLM Spot 서빙 비용과 대가 | 비용 −54%, 평균 지연 +18%, P99 +90% | On-Demand만 대비, GPT-20B, T4 | spotserve |
| 리전 간 Spot 서빙 | 비용 −41~44%, 실패율 0.34~0.62% | LLM 70B, 약 22시간, 3개 리전 | skyserve |
| 대체 인스턴스 준비 시간 | 183초 (Llama-2-7b, vLLM) | AWS, 이미지 사전 설치 | skyserve |
| 에이전트 대화만 복구 | 8~13% 성공 | Terminal-Bench, 장애 1회 | crab |
| 에이전트 샌드박스 C/R | 중앙값 0.71초, p95 1.00초 | 60초 알림 시뮬레이션, 호스트 1대 | crab |
| CRIU 체크포인트 시간 | 128MB 7.8초 ~ 1GB 47.4초 | 동시성 높을 때, c6id.32xlarge | crab |
| SPS 높음 → 확보 | 24시간 내 확보 실패 0% | 단일 인스턴스 요청, 2022 | spotlake |
| 다중 노드 확보 | SPS 최고점 타입도 50대면 성공률 약 20% | AWS, 2024~25 | multinode-sps |
| 벤더 사례 절감률 | 65~85% | 벤더 자가 보고, 기준 불명 | duolingo-spotinst |
| ML 추론 플랫폼 비용 | 시간당 $13.1, Bedrock($57.2) 대비 77%, SageMaker On-Demand($37.7) 대비 약 65% (제 계산) | Llama-2-13B 시나리오, **시뮬레이션**, 비교 대상마다 과금 방식 다름 | ml-pipeline-platform-kcc24 |
| 학습 체크포인트 오버헤드 | 에포크 약 9분 38초 중 평균 2.97초 (약 0.5%, 제 계산) | g4dn.xlarge 1대, GPT-2, 실제 회수 없음 | ml-pipeline-platform-kcc24 |

## 자료 간 연결

### 1. 동시 회수 (CFP: "여러 AZ에서 Spot 회수가 동시에 일어나도 견뎌야 한다")
- 기업의 우려: Zalando가 "클러스터 전체 동시 회수"를 주된 걱정으로 꼽음 (duolingo-spotinst)
- 풀 단위: 같은 타입·AZ는 3분 안에 함께 회수됨 (ding-dong-ditch)
- 존·리전 단위: 같은 리전의 존들도 함께 회수되는 경향, 다른 리전은 상관이 낮음 (skyserve)
- 대응 근거: 인스턴스 타입·AZ 다양화 (aws-skyscanner, scatterlab), 리전 간 분산 + 동적 fallback (skyserve)
- 빈칸: **CPU 웹 서비스의 동시 회수를 측정한 자료는 없습니다.** SkyServe와 DDD의 회수 통계는 주로 GPU 또는 혼합 타입입니다.

### 2. 상태를 어디에 두고 어떻게 옮기나

| 상태가 있는 곳 | 방법 | 자료 | 측정 여부 |
|---|---|---|---|
| GPU 메모리 (KV cache) | 토큰 단위 커밋, NCCL로 인스턴스 간 전송 | spotserve | 측정함 (재생 트레이스) |
| 샌드박스 OS (파일, 프로세스) | eBPF로 턴별 변화 분류, ZFS + CRIU | crab | 측정함 (시뮬레이션) |
| 컨테이너 프로세스 | 체크포인트 → 객체 저장소 → 복원 | cedana | 측정 안 함 |
| 외부 DB (대화·그래프 상태) | 단계별 체크포인트, lease 넘김 | langchain | 측정 안 함 |
| 모델 학습 상태 | 에포크마다 체크포인트를 S3에 저장, 실패 시 재시도로 복원 (Ray Train) | ml-pipeline-platform-kcc24 | 오버헤드만 측정 (회수 없음, 1대) |
| 디스크·IP (Cassandra 노드) | 벤더 Stateful Elastigroup | duolingo-spotinst (Zalando) | 측정 안 함 |
| 상태 없음 | 진행 중 요청 버리고 재시도 | skyserve | 측정함 |

### 3. 가용성 신호와 예측
- 공식 신호: 2분 알림, rebalance recommendation, SPS, Spot Instance Advisor (aws-spot-facts)
- 신호의 신뢰도: 가격은 쓸모없음, SPS 우선 (spotlake). 단일 노드 SPS는 다중 노드 확보를 과대평가 (multinode-sps)
- 저비용 직접 관측과 3~60분 앞 예측 (ding-dong-ditch)
- 예측을 쓰는 운영: 벤더의 "최대 15분 전 예측"(duolingo-spotinst, 근거 없음), rebalance recommendation 기반 사전 drain (scatterlab)

### 4. 어느 자료에도 없는 것
- **On-Demand와 같은 p95·가용성을 유지하면서 Spot으로 옮겼다는 측정 결과.** 기업 사례는 절감률만, 논문은 지연이 늘거나(SpotServe) 작업 완료 시간만(Crab) 보고합니다.
- 실제 AWS 회수 환경에서 AI 에이전트를 운영한 측정. Crab의 Spot 실험은 시뮬레이션입니다.
- 짧은 웹 요청(수백 ms)에서 리전 간 분산의 지연 비용. SkyServe는 수 초짜리 LLM 요청이라 리전 간 왕복이 묻힌다고 전제했습니다.
- 회수 알림 뒤 "즉시 대체 Spot 생성"으로 가용성을 확보했다는 측정. KCC 2024 논문은 이를 주장하지만 수치가 없고, SkyServe는 대체 GPU 복제본 하나를 띄우는 데 183초를 쟀습니다.
- 인스턴스 선택에 가용성 신호(SPS, 회수 빈도)를 넣어 실제 확보·회수로 검증한 사례. KCC 2024 논문의 선출 점수는 가격·성능만 씁니다(저장소의 `per_single_node` 추천기만 `SPS >= 2` 필터).

## 아직 확인하지 못한 것

- Duolingo 고객사례 PDF의 작성 연도
- Ding-Dong Ditch의 학회 게재 여부 (IEEE CLOUD 2026으로 검색되나 본문 미확인)
- SpotLake·HPDC'25의 SPS 범위(1~3)와 AWS 공식 범위(1~10)가 다른 이유
- 스캐터랩 후속 글(On-Demand 최소 개수, 커스텀 rebalance 로직)

## 문서 추가 규칙

1. 새 자료는 사용자 승인을 받은 뒤 추가합니다.
2. 원문(PDF 전문 또는 HTML 본문)을 직접 읽고 씁니다. 검색 요약만 보고 수치를 옮기지 않습니다.
3. 기존 문서와 같은 형식을 씁니다: frontmatter(title, notion_title, source_url, source_type, published, accessed, reliability, tags) → 한 줄 요약 → 핵심 내용 → 실험 조건 → 핵심 수치 → 프로젝트에 주는 것 → 한계와 주의점 → 인용할 때 쓰는 표현 → 관련 문서
4. 원문에 없는 해석은 "(해석, 원문 아님)"으로 표시합니다.
5. 이 README의 문서 목록, 핵심 수치, 자료 간 연결을 함께 고칩니다.
