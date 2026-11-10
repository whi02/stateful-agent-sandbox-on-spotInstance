---
title: The Runtime Behind Production Deep Agents
notion_title: 에이전트를 중단 후에도 이어서 실행하는 방식
notion_category: 학술 자료 (논문)
source_url: https://www.langchain.com/blog/runtime-behind-production-deep-agents
source_type: 기업(LangChain) 공식 블로그, 자사 제품 개념 가이드
authors: Sydney Runkle, Vivek Trivedy (LangChain)
published: 2026-04-20
accessed: 2026-10-13
reliability: B (자사 런타임 동작 설명으로서). 효과에 대한 주장은 C
tags: [ai-agent, durable-execution, checkpoint, postgresql, langgraph, human-in-the-loop, streaming]
---

# The Runtime Behind Production Deep Agents (LangChain, 2026)

## 한 줄 요약

오래 도는 AI 에이전트를 프로덕션에서 운영하려면 **durable execution**(단계마다 상태를 외부 DB에 저장하고, 워커가 죽으면 다른 워커가 마지막 체크포인트부터 이어 가는 실행)이 기반이라고 설명하는 LangChain의 제품 가이드입니다. **Spot은 한 번도 언급하지 않습니다.** 이 프로젝트에서는 "에이전트 상태를 애플리케이션 수준에서 외부에 저장하는 방식"의 대표 예로 씁니다.

> 노션에서는 "학술 자료(논문)"로 분류했지만 논문이 아니라 기업 블로그입니다.

## 핵심 내용

### Durable execution
- 에이전트 루프(모델 추론 → 도구 호출 → 결과 관찰 반복)는 몇 분에서 몇 시간까지 걸립니다. 워커가 죽었다고 처음부터 다시 하면 이미 쓴 토큰과 도구 실행이 낭비됩니다. 원문 예시는 "20분 걸린 리서치 에이전트"입니다.
- 동작 방식(원문 설명)
  - 관리형 작업 큐 위에서 실행되며 자동으로 체크포인트를 만듭니다.
  - 그래프 실행의 super-step마다 체크포인트를 저장소(기본 **PostgreSQL**)에 씁니다. 키는 `thread_id`입니다.
  - **워커가 죽으면 그 실행의 lease가 풀리고, 다른 워커가 최신 체크포인트부터 이어 갑니다.**
  - 사람 승인을 기다릴 때는 프로세스가 자리를 내놓고 무기한 잠들었다가 재개됩니다.
  - 노드별로 재시도 정책(backoff, 최대 횟수, 재시도할 예외)을 정합니다.

### 메모리
- 단기 메모리: 한 대화(thread) 안의 메시지, 도구 호출, 중간 상태. 체크포인트에 들어 있습니다.
- 장기 메모리: 대화를 넘어서는 사용자·조직 단위 기억. 네임스페이스 기반 key-value store, 기본 PostgreSQL, 임베딩 검색 지원

### 실시간 상호작용
- 스트리밍은 `Last-Event-ID` 헤더로 끊긴 지점부터 다시 받을 수 있습니다. 연결이 끊겨도 출력이 빠지지 않습니다.
- 실행 중 새 메시지가 오면(double-texting) enqueue(기본), reject, interrupt, rollback 중 하나로 처리합니다.

### 기타
- 코드 실행용 샌드박스(Daytona, Modal, Runloop, LangSmith Sandboxes)를 붙이면 `execute` 도구가 생깁니다. 웜 풀로 콜드 스타트를 없애고, 인증 프록시로 비밀 값을 샌드박스 밖에 둡니다.
- 체크포인트를 골라 상태를 바꾸고 거기서 다시 실행하는 time travel(분기), 정기 실행(cron), MCP·A2A·웹훅 연동을 설명합니다.

## 프로젝트에 주는 것

- **Spot 회수를 "워커 장애"로 보면** 이 구조가 그대로 대응책이 됩니다. 상태가 PostgreSQL에 있으면 워커가 회수돼도 다른 워커가 lease를 받아 이어 갑니다.
- 다만 Crab 논문은 LangGraph류를 "대화만 저장"하는 프레임워크 수준 방식으로 분류합니다. 에이전트가 샌드박스에서 파일을 바꾸거나 프로세스를 띄우면 대화만으로는 복구가 대부분 실패한다고 측정했습니다(Terminal-Bench 8~13%). **상태가 어디에 있느냐(DB의 대화 상태인지, 샌드박스 OS 상태인지)에 따라 필요한 기법이 달라진다**는 점을 보여 주는 대비 자료입니다.
- 스트리밍 재개(`Last-Event-ID`)는 회수 중에도 사용자가 응답을 끊김 없이 받게 하는 설계 요소로 참고할 수 있습니다.

## 한계와 주의점

- 자사 제품(LangSmith Deployment, Agent Server) 홍보 글입니다. 성능 수치나 장애 복구 측정이 없습니다.
- Spot이나 인스턴스 회수를 직접 다루지 않습니다. Spot과의 연결은 이 문서를 정리하며 붙인 해석입니다.
- 체크포인트 쓰기 비용, 재개에 걸리는 시간 같은 운영 수치가 없습니다.

## 인용할 때 쓰는 표현

- "LangChain(2026)의 에이전트 런타임은 그래프 실행 단계마다 체크포인트를 PostgreSQL에 저장하고, 워커가 죽으면 lease가 풀려 다른 워커가 최신 체크포인트부터 실행을 이어 간다."

## 관련 문서

- [crab-agent-sandbox-cr-2026.md](crab-agent-sandbox-cr-2026.md) — 대화만 저장하면 부족한 경우를 측정한 논문
- [cedana-live-migration-demo.md](cedana-live-migration-demo.md) — 컨테이너(프로세스) 수준 상태 이전
