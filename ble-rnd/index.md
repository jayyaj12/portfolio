---
layout: project
title: "BLE National R&D Project"
subtitle: "BLE 연결/명령 신뢰성을 높이기 위한 검증·상태관리 설계를 적용한 사례"
period: "2021.01 – 2023.12"
role: "Android Developer (BLE Protocol / Device Control)"
featured: false
tags:
  - Android
  - Kotlin
  - MVVM
  - Clean Architecture
  - BLE
  - NFC
metrics:
  - { value: "Checksum", label: "무결성 검증" }
  - { value: "Timestamp", label: "중복/순서 관리" }
  - { value: "Stability", label: "실사용 안정화" }
---

## Overview
BLE 특성상 환경에 따라 품질 편차가 커서, 데이터 무결성과 응답 순서를 보장하지 못하면 실사용에서 오동작이 발생했습니다. 검증 계층과 상태 흐름을 강화해 신뢰성을 끌어올렸습니다.

## Problem
- BLE 연결/응답 품질이 환경에 따라 불안정
- 패킷 유실/중복/순서 뒤틀림으로 상태 오염 가능
- 콜백 중심 흐름으로 디버깅/재현이 어려움

## Solution
- Checksum 기반 데이터 무결성 검증으로 오류 패킷 차단
- Timestamp 기반으로 최신 응답/중복 응답 구분
- 연결→탐색→구독→명령 흐름을 상태 기반으로 관리
- MVVM/Clean으로 BLE 의존 코드와 화면 로직 분리

## Result
- 데이터 오염/오동작 리스크 감소
- 실사용 환경에서 연결/명령 흐름 안정화(정성)

## Key Decisions / Trade-offs
- 검증/상태 관리 로직 추가로 복잡도는 증가했지만, 현장 안정성을 우선

## Interview-ready
- Checksum/Timestamp를 어떤 단위로 넣었고, 무엇을 막았나?
- BLE에서 재현 어려운 버그를 어떻게 재현/해결했나?
- 제조사/OS 버전별 BLE 차이를 어떻게 대응했나?
