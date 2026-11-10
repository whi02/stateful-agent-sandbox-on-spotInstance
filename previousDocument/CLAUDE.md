# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 이 폴더의 성격

코드 저장소가 아니라 문서 폴더다. build·lint·test 명령이 없고 git repository도 아니다.

WorldSkills Cloud Computing 훈련의 "프로젝트 트랙" 자료다. 선수 4명이 약 10주 동안 각자 AWS 프로젝트를 제안하고 직접 lead한다.

| 파일 | 내용 |
|---|---|
| `WorldSkills_CloudComputing_Project_Track_CFP.pdf` / `.md` | 트랙 CFP (초안, 2026-09-20). 트랙 A~G, 제안서 7개 항목, Linear 관리, AI coding tool 원칙, 필수 산출물, Claude Academy 강좌, Bedrock 비용 한도 |
| `Example_Proposal_Cost_Anomaly_Detection.pdf` / `.md` | CFP 4장 요건에 맞춘 예시 제안서 (트랙 A "비용 이상 1시간 내 차단"). 수치는 모두 가상 값 |

- PDF가 원본이고, md는 PDF 문장을 요약 없이 그대로 옮긴 사본이다. PDF가 바뀌면 md를 다시 맞춘다.
- 두 md는 서로 링크한다. CFP 4장은 예시 제안서 md로, 예시 제안서는 CFP의 `#4-제안서-제출-요건` anchor로 연결된다. 제목을 바꾸면 이 anchor가 깨지는지 확인한다.
- 예시 제안서의 architecture diagram은 PDF 텍스트 layer에 들어 있던 mermaid 원본이다.
- 작성자는 PDF 텍스트 layer에 `@Someone`으로 나오지만 렌더링된 화면에는 "송무현"이다. md에는 화면 기준으로 적는다.

## PDF 다루기

이 환경에는 `pdftotext`·`pdfinfo`(poppler)와 `pypdf`·`pdfplumber`가 설치되어 있지 않다.

- 본문은 Read 도구로 PDF를 직접 읽는다 (페이지 이미지와 텍스트가 함께 나온다).
- 링크 URL은 읽은 텍스트에 드러나지 않는다. Python 표준 라이브러리만으로 raw bytes와 `zlib`로 푼 stream에서 `/URI (...)`를 정규식으로 찾는다. URL 문자는 `\072`(`:`), `\057`(`/`), `\056`(`.`), `\055`(`-`)처럼 8진수 escape로 저장되어 있다.

## 제안서 작업을 도울 때

CFP 4장의 AI 사용 원칙이 이 트랙 선수에게 적용된다.

- 문제 정의(2번)와 목표(3번)는 선수가 먼저 초안을 쓴다. AI는 그 뒤에 빠진 점과 반론을 찾는 데만 쓴다.
- AI 제안을 받아들이거나 버린 이유는 제안서 끝 "AI 사용 내역"에 한두 줄로 남긴다.
- 제안서의 모든 문장은 선수가 자기 말로 설명할 수 있어야 한다.
