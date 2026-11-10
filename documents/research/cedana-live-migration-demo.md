---
title: Using Cedana to Live-Migrate Stateful Workloads Between Spot Instances
notion_title: Spot 인스턴스 사이에서 stateful 컨테이너를 옮기는 데모
notion_category: 학술 자료 (논문)
source_url: https://cedana.substack.com/p/using-cedana-to-live-migrate-stateful
source_type: 벤더(Cedana) 제품 튜토리얼 블로그
published: 2024-11-02
accessed: 2026-10-13
reliability: C
tags: [vendor, live-migration, checkpoint-restore, container, gke, stateful]
---

# Using Cedana to Live-Migrate Stateful Workloads Between Spot Instances (2024)

## 한 줄 요약

Cedana의 Save, Migrate, Resume(SMR) REST API로, GKE Spot 노드에서 돌던 XGBoost 학습 컨테이너를 체크포인트하고 다른 노드로 옮겨 이어서 실행하는 **6단계 튜토리얼**입니다. 이전에 걸린 시간, 다운타임, 비용 절감 같은 **측정값은 하나도 없습니다.**

> 노션에서는 "학술 자료(논문)"로 분류했지만 논문이 아니라 벤더 블로그입니다.

## 절차 (원문 6단계)

1. GKE 클러스터를 만듭니다. 노드 이미지는 Ubuntu + containerd여야 합니다. **Container-Optimized OS 이미지에서는 Cedana가 동작하지 않습니다.**
2. Helm으로 Cedana를 설치합니다(helm-manager, helm-helper Pod).
3. XGBoost 학습 Pod를 Spot 노드에 띄웁니다(California housing 데이터, 10,000 라운드).
4. helm-manager를 port-forward하고 `POST /containerd/checkpoint`를 호출합니다.
   - 프로세스 상태는 노드의 `/tmp/xgboost_checkpoint`에 저장합니다.
   - 컨테이너 rootfs는 `ttl.sh`(1시간 보관 임시 레지스트리)에 이미지로 올립니다.
   - **회수는 인스턴스를 직접 종료해서 흉내 냅니다.** 실제 클라우드 회수가 아닙니다.
   - 노드에 SSH로 들어가 `gsutil cp`로 체크포인트를 GCS 버킷에 복사합니다.
5. 체크포인트 이미지로 `sleep infinity` Pod를 먼저 띄운 뒤 `POST /restore`로 프로세스 상태를 붙입니다.
6. 학습이 끊긴 지점부터 이어지는지 확인합니다. 원문은 무엇을 확인하라고만 적었고 결과 화면은 없습니다.

## 프로젝트에 주는 것

- **컨테이너 단위 상태 이전의 구성 요소**를 보여 줍니다: 프로세스 상태 덤프, rootfs 이미지화, 객체 저장소로 복사, 새 노드에서 복원
- 원문 절차가 수동(SSH, gsutil, curl)이라, 이것을 회수 알림 2분 안에 자동으로 끝내려면 무엇을 만들어야 하는지 가늠하는 데 쓸 수 있습니다.
- Spot에서 상태 있는 작업을 옮기는 상용 제품이 있다는 업계 근거입니다. 다만 동작 증거가 약합니다.

## 한계와 주의점

- 측정값이 없습니다. FAQ는 "작은 워크로드는 몇 분이면 된다"고만 적었습니다. 이 정도면 2분 알림 안에 끝난다는 보장이 없습니다.
- 회수를 인스턴스 종료로 흉내 냈고, 자동화가 아니라 사람이 단계를 실행합니다.
- 체크포인트 경로 표기가 단계마다 다르고(`/tmp/xgboost_checkpoint` 대 `.tar`), `<pod-id>` 같은 자리표시자가 남아 있습니다.
- "여러 클라우드 간 이전 가능"이라고 주장하지만 시연하지 않았습니다.
- 내부 구현(CRIU 사용 여부 등)은 글에 나오지 않습니다.
- "Spot은 최대 90% 할인"과 "최대 10배 할인"을 섞어 씁니다.

## 인용할 때 쓰는 표현

- "Cedana(2024, 벤더 튜토리얼)는 GKE에서 상태 있는 컨테이너를 체크포인트해 다른 노드에서 이어 실행하는 절차를 보였지만, 이전 시간이나 다운타임은 측정하지 않았다."

## 관련 문서

- [crab-agent-sandbox-cr-2026.md](crab-agent-sandbox-cr-2026.md) — 같은 문제(컨테이너 상태 C/R)를 측정한 논문. CRIU 덤프 시간이 메모리에 비례해 수십 초까지 늘어남
- [../company-cases/duolingo-spotinst-casestudies.md](../company-cases/duolingo-spotinst-casestudies.md) — 상태를 유지하는 다른 벤더 도구(Stateful Elastigroup)
