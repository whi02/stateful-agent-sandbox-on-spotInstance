# 문서 소개 (introduceDocs)

이 프로젝트의 문서가 왜 만들어졌고 무엇이 들어 있는지 한 번에 보는 목록입니다. 문서를 하나씩 열지 않아도 이 파일만 읽으면 문서화된 내용을 이해할 수 있게 적습니다.

- 대상: `documents/`의 문서와 앞으로 새로 작성하는 문서. `previousDocument/`의 문서는 적지 않습니다.
- 항목 구성: 파일, 추가일, 작성 이유, 내용, 수정(문서를 고친 경우에만)
- 새 문서를 추가하거나 기존 문서의 내용을 고치면 같은 작업에서 이 파일도 갱신합니다. (규칙은 프로젝트 `CLAUDE.md`의 "문서 소개 규칙")

## 기준 문서

### 자료 전체 안내

- 파일: [README.md](README.md)
- 추가일: 2026-10-13
- 작성 이유: 12개 자료를 매번 열어 보지 않고도 어떤 자료가 있고 어디까지 믿어도 되는지 한곳에서 보려고 썼습니다.
- 내용: 폴더 구조, 문서 목록, 신뢰도 등급(A/B/C) 기준, 조건을 붙인 핵심 수치 표, 동시 회수·상태 이전·가용성 신호로 묶은 자료 간 연결을 담았습니다. 어느 자료에도 없는 빈칸(예: On-Demand와 같은 p95를 측정한 자료 없음)과 문서 추가 규칙도 있습니다.
- 수정: 2026-10-14 — 폴더 구조에 introduceDocs.md와 projectPlan.md 한 줄씩을 추가했습니다.
- 수정: 2026-10-16 — 폴더 구조에 spotConsiderations.md 한 줄을 추가했습니다.
- 수정: 2026-10-16 — 사용자가 제공한 논문(선생님의 KCC 2024)을 추가해 문서 목록, 핵심 수치(2건), 상태 이전 표, "어느 자료에도 없는 것"을 보강했습니다.
- 수정: 2026-10-16 — "아직 확인하지 못한 것"에서 GCP·Azure 회수 알림 줄을 지웠습니다 (공식 문서로 확인해 aws-spot-facts.md 9절에 옮김).

### AWS·Karpenter 공식 문서 기준 Spot 동작

- 파일: [aws-spot-facts.md](aws-spot-facts.md)
- 추가일: 2026-10-13
- 작성 이유: 논문과 블로그마다 AWS Spot 동작 설명이 달라서(예: SpotServe 논문의 "AWS 30초"는 오류) 맞는 값을 가릴 기준이 필요했습니다.
- 내용: AWS·Karpenter 공식 문서로 확인한 2분 회수 알림(best effort), rebalance recommendation, SPS(1~10점), 할당 전략(price-capacity-optimized 권장), Karpenter의 회수 처리를 정리했습니다. 자료끼리 설명이 다르면 이 문서를 따릅니다.
- 수정: 2026-10-16 — 8절(FIS 회수 대기 2~15분, FIS 요금, Spot 기본 할당량 5 vCPU)과 9절(Azure 30초, GCP 알림 0~120초 설정 + 종료 처리 최대 30초)을 추가하고, 6절의 "GCP·Azure 30초 미확인" 줄을 지웠습니다.

### 프로젝트 계획과 진행도

- 파일: [projectPlan.md](projectPlan.md)
- 추가일: 2026-10-14
- 작성 이유: 프로젝트가 어디까지 왔고 다음에 무엇을 해야 하는지 사용자와 Claude Code가 같은 문서로 확인하려고 만들었습니다.
- 내용: 선행 작업(0), 사전 학습, 제안서 작성(7항목), Linear 구성, 진행 중 산출물, 종료 산출물을 계층 목록으로 나누고 최하위 작업마다 체크박스로 진행 상태를 표시했습니다. 확정한 주제, 예산(AWS $200 이하 목표), 기간(목표 종료 11월 초, 최종 마감 11월 말), 열린 결정도 상단에 적었습니다.
- 수정: 2026-10-16 — 주 단위 큰 일정(M1 주제·목표 확정 ~ M4 실험·정리, 목표 종료 11-08, 버퍼, 최종 마감 11-30)을 상단에 추가했습니다.

### Spot 프로젝트 고려 사항

- 파일: [spotConsiderations.md](spotConsiderations.md)
- 추가일: 2026-10-16
- 작성 이유: 사용자가 정리한 Spot 고려 사항 11개를 정식 주제·목표를 정하고 제안서 지표·접근 방법·리스크를 쓸 때 출발점으로 쓰려고 남겼습니다.
- 내용: 사용자 원문 11개를 그대로 두고, 항목마다 근거(`documents/`와 공식 문서), 모르는 것, 확정한 설계 방향(예: 상태를 버리는 수준 0을 대조군으로 추가, 알림 시간 120초/30초/0초를 실험 변수로)을 붙였습니다. 원문에 없던 고려 사항(동시 회수, Spot 기본 할당량 5 vCPU, 자연 회수가 드물 가능성 등)과, 2026-10-16에 공식 문서로 확인한 사실(warm pool은 Spot 미지원, FIS 회수 대기 2~15분, GCP는 알림 시간 0~120초 설정 가능 등)도 담았습니다.
- 수정: 2026-10-25 — 선생님 피드백(`SongmhT.md`)에 따라 3.2의 풀 단위 조기 경보를 업그레이드 후보로 내리고, 6절에 반영 내용을 추가했습니다.
- 수정: 2026-10-16 — 사용자가 항목별 제안과 4절 A~F를 모두 채택해 "제안(미확정)" 표시를 "확정"으로 바꾸고, 6절을 "확정 이후 남은 것"으로 고쳤습니다.

### 송무현 선생님 피드백

- 파일: [SongmhT.md](SongmhT.md)
- 추가일: 2026-10-16
- 작성 이유: 코치가 AI Agent의 뜻과 프로젝트 범위에 대해 준 답변을 원문 그대로 남기고, 이미 확정한 결정과 어떻게 이어지는지 한곳에서 보려고 만들었습니다.
- 내용: AI Agent를 "실행 환경을 마이그레이션하는 것"으로 보는 해석을 인정했고(관리하는 agent도 가능), LLM 서버와 SPS 신호 분석은 지금 제외하며 "MCP tool 실행환경이 Spot에서 운영되는 관점"에서 직관적인 것부터 단계적으로 시작하라는 방침입니다. MCP draft 스펙으로 확인한 상태 핸들 개념, 기존 확정 사항(조기 경보 등)에 미치는 영향, 후속 질문 후보도 담았습니다. 영향은 아직 다른 문서에 반영하지 않은 제안입니다.
- 수정: 2026-10-25 — 4절의 영향을 `projectPlan.md`, `CLAUDE.md`, `spotConsiderations.md`에 반영했습니다(사용자 승인). 이 문서의 내용 자체는 바뀌지 않았습니다.

### 제안서 초안

- 파일: [proposal.md](proposal.md)
- 추가일: 2026-10-26
- 작성 이유: 프로젝트 계획의 제안서 단계에서 요구하는 7개 항목과 AI 사용 내역을, 지금까지 확정한 주제·목표·Scope로 한 문서에 모으려고 만들었습니다.
- 내용: 제목 후보, 목표 4가지와 정량·정성 지표(목표치는 On-Demand 기준선을 재기 전의 가설), Must / Nice to Have / Out of scope, 상태 보존 수준 0~3 설계, 서울 리전 단가로 만든 약 $90의 비용 추정(가정 포함, 계산기로 재계산 필요), 리스크 3개, 학습 계획입니다. 문제 정의, 제목·트랙 선택, 학습 계획의 현재 수준, AI 사용 내역은 사용자 작성 칸으로 비워 두었습니다.

## 기업 사례 (company-cases/)

### Skyscanner·Mobileye 사례: 웹 서비스 Spot 100%

- 파일: [company-cases/aws-skyscanner-mobileye-capacity-optimized.md](company-cases/aws-skyscanner-mobileye-capacity-optimized.md)
- 추가일: 2026-10-13
- 작성 이유: 상시 트래픽 웹 서비스를 Spot 100%로 운영한 기업 사례가 이미 있는지 확인하려고 정리했습니다.
- 내용: AWS 공식 블로그(2020)에서 Skyscanner가 웹 플랫폼을 Spot 전용 Kubernetes로 운영하고, 할당 전략을 바꿔 블랙프라이데이 대비 테스트의 회수를 200~300회에서 10~15회로 줄였다는 내용입니다. 지연·가용성 수치는 없습니다.

### Duolingo 등 Spotinst 고객사례

- 파일: [company-cases/duolingo-spotinst-casestudies.md](company-cases/duolingo-spotinst-casestudies.md)
- 추가일: 2026-10-13
- 작성 이유: 기업들이 Spot을 쓸 때 무엇을 걱정했고 절감을 얼마나 주장하는지 보려고 정리했습니다.
- 내용: 벤더 마케팅 자료(C등급)로, Duolingo 65%, Ticketmaster 60~66%, N26 85% 절감 보고와 Zalando의 우려("2분은 짧다", "클러스터 전체가 동시에 회수될 수 있다"), Cassandra(stateful) PoC를 담았습니다. 절감률만 있고 p95·가용성 같은 SLO 수치는 없습니다.

### 스캐터랩의 Karpenter Spot 운영

- 파일: [company-cases/scatterlab-karpenter-spot.md](company-cases/scatterlab-karpenter-spot.md)
- 추가일: 2026-10-13
- 작성 이유: 국내 기업이 AI 서빙(GPU 포함)을 프로덕션 Spot에 올린 방법과 회수 대응 구조를 실험 설계에 참고하려고 정리했습니다.
- 내용: 스캐터랩(2023)이 EKS 서버를 Karpenter와 Node Termination Handler로 Spot 운영하고 AWS FIS로 회수를 일부러 일으켜 검증한 방법입니다. 에이전트가 아니라 AI 챗봇 서빙 사례이고, 절감률과 SLO 수치는 없습니다.

## 연구 자료 (research/)

### SpotServe: Spot에서 LLM 추론 상태 옮기기

- 파일: [research/spotserve-asplos24.md](research/spotserve-asplos24.md)
- 추가일: 2026-10-13
- 작성 이유: 상태가 있는 LLM 추론을 Spot에서 옮기는 구체적인 방법과, On-Demand 대비 성능이 얼마나 나빠지는지 기준선을 얻으려고 정리했습니다.
- 내용: ASPLOS'24 논문으로, 회수 알림이 오면 KV cache(추론 중간 상태)를 토큰 단위로 다른 인스턴스에 옮겨 생성을 이어 갑니다. On-Demand만 대비 비용 최대 −54%, 대신 평균 지연 최대 +18%, P99 최대 +90%입니다(T4 GPU, 회수 이력 재생 실험).

### SkyServe: 여러 AZ 동시 회수에 대처하기

- 파일: [research/skyserve-spothedge-eurosys25.md](research/skyserve-spothedge-eurosys25.md)
- 추가일: 2026-10-13
- 작성 이유: 여러 AZ에서 Spot이 동시에 회수될 때 어떻게 대처하는지 근거와 수치를 얻으려고 정리했습니다.
- 내용: EuroSys'25 논문으로, 같은 리전 존들이 함께 회수되는 경향(회수 후 5분 내 추가 회수 83~97%, AWS GPU 기준)을 측정했습니다. 리전 간 분산과 동적 On-Demand fallback(SpotHedge)으로 On-Demand만 대비 비용 −41~44%, 요청 실패율 0.34~0.62%를 보고했습니다(LLM 70B 서빙, 약 22시간).

### Crab: 에이전트 샌드박스 상태 저장·복원

- 파일: [research/crab-agent-sandbox-cr-2026.md](research/crab-agent-sandbox-cr-2026.md)
- 추가일: 2026-10-13
- 작성 이유: 상태를 가진 AI agent를 Spot에 올리는 일을 직접 다룬 연구가 있는지, 있다면 어디까지 측정했는지 확인하려고 정리했습니다.
- 내용: 2026년 arXiv 논문(심사 전)으로, 에이전트 샌드박스의 대화·파일·프로세스 상태를 OS 수준에서 저장하고 복원합니다. 대화만 복구하면 8~13% 성공인데 Crab은 100%이고, Spot 실험은 실제 회수가 아니라 60초 알림 시뮬레이션입니다.

### Cedana: 컨테이너 상태 이전 데모

- 파일: [research/cedana-live-migration-demo.md](research/cedana-live-migration-demo.md)
- 추가일: 2026-10-13
- 작성 이유: 컨테이너의 상태를 다른 Spot 노드로 옮기는 상용 도구가 있는지, 절차가 어떤지 보려고 정리했습니다.
- 내용: Cedana 블로그(2024)의 GKE 6단계 튜토리얼로, 학습 중인 컨테이너를 체크포인트해 다른 노드에서 복원합니다. 벤더 튜토리얼(C등급)이라 이전 시간·다운타임 측정값이 없고, 회수는 인스턴스를 직접 종료해 흉내 냈습니다.

### SpotLake: Spot 가용성은 무엇을 보고 판단하나

- 파일: [research/spotlake-iiswc22.md](research/spotlake-iiswc22.md)
- 추가일: 2026-10-13
- 작성 이유: Spot 인스턴스 타입과 AZ를 고를 때 가격, SPS, 회수 빈도 중 어떤 신호를 믿어야 하는지 근거를 얻으려고 정리했습니다.
- 내용: IISWC'22 논문으로, 가격 이력은 가용성 예측에 쓸모가 없고 SPS가 높으면 24시간 안에 확보 실패가 0%였습니다(503건, 단일 인스턴스 요청). 과거 이력을 쓴 예측은 Accuracy 0.73입니다.

### 다중 노드 SPS: 여러 대를 한꺼번에 확보할 수 있나

- 파일: [research/multinode-sps-hpdc25.md](research/multinode-sps-hpdc25.md)
- 추가일: 2026-10-13
- 작성 이유: 여러 AZ에서 동시 회수된 뒤 여러 대를 한 번에 다시 확보해야 할 때, 단일 노드 SPS를 믿어도 되는지 확인하려고 정리했습니다.
- 내용: HPDC'25 2쪽 논문으로, 단일 노드 SPS가 최고점인 타입도 50대를 요청하면 확보 성공률이 약 20%로 떨어졌습니다. 목표 노드 수별 SPS를 10분마다 모아 SpotLake에 공개했고, 한국어 전문 정리본은 `spot-proposal/`에 있습니다.

### Ding-Dong Ditch: 동시 회수 통계와 가용성 예측

- 파일: [research/ding-dong-ditch-2026.md](research/ding-dong-ditch-2026.md)
- 추가일: 2026-10-13
- 작성 이유: 동시 회수가 얼마나 빨리, 어떤 단위로 몰리는지와 회수를 미리 예측할 수 있는지 수치로 확인하려고 정리했습니다.
- 내용: 2026년 arXiv 논문(심사 전)으로, 실제 회수 2,635건에서 같은 인스턴스 타입·AZ의 동시 회수 92.9%가 3분 안에 일어났고 60분 앞 가용성을 F1 약 0.85로 예측했습니다. 예측 평가는 배치 분석(TPC-DS) 시뮬레이션입니다.

### LangChain: 중단돼도 이어서 실행하는 에이전트 런타임

- 파일: [research/langchain-deep-agents-runtime.md](research/langchain-deep-agents-runtime.md)
- 추가일: 2026-10-13
- 작성 이유: 에이전트 상태를 외부 DB에 저장하는 애플리케이션 수준 방법을, Crab의 샌드박스 수준 방법과 견주려고 정리했습니다.
- 내용: LangChain 블로그(2026)가 설명하는 durable execution으로, 단계마다 체크포인트를 PostgreSQL에 저장하고 워커가 죽으면 다른 워커가 이어 받습니다. 논문이 아니라 제품 가이드이고 Spot은 언급하지 않으며, Spot과의 연결은 문서를 정리하며 붙인 해석입니다.

### 선생님 논문: ML 파이프라인 배포 플랫폼 (KCC 2024)

- 파일: [research/ml-pipeline-platform-kcc24.md](research/ml-pipeline-platform-kcc24.md)
- 추가일: 2026-10-16
- 작성 이유: 선생님(송무현)이 쓴 논문에서 "AI Agent"와 이 프로젝트에 무엇을 기대하시는지 추론하고, 후속 연구로 무엇을 비워 두었는지 보려고 정리했습니다. 선생님은 HPDC'25, Ding-Dong Ditch의 저자이기도 합니다.
- 내용: ML 학습·추론을 Spot·서버리스로 운영하는 플랫폼으로, 학습은 에포크마다 체크포인트를 저장하고 추론 비용은 Bedrock 대비 최대 77% 낮다고 보고합니다(시뮬레이션). 실제 회수 환경의 가용성·지연 측정은 없습니다. 논문에 AI Agent는 나오지 않고, 구현 저장소를 읽은 결과 체크포인트는 Ray Train과 S3, 회수 처리는 Karpenter 인터럽션 큐였으며 인스턴스 선택은 가격·성능 점수였습니다. 저장소에는 라이선스가 없습니다.
