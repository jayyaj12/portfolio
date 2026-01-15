---
layout: project
title: "Real-time Elevator Control System"
subtitle: "RS-232/485 + UDP 기반 제어/표시 시스템을 Android에서 안정적으로 운영한 사례"
period: "2023.06 – 2025.06"
role: "Sole Android Developer (Device Control / Operation Automation)"
featured: true
tags:
  - Android
  - Kotlin
  - Java
  - MVVM
  - Clean Architecture
  - Serial(RS-232/485)
  - UDP
metrics:
  - { value: "150+", label: "상용 디바이스 운영" }
  - { value: "84%", label: "UI 반응 지연 개선" }
  - { value: "OTA", label: "원격 업데이트/운영 자동화" }
---

## Overview
현장 장비와 연결되는 Android 앱을 단독 개발·운영하며, Serial/UDP 통신 기반 제어를 안정화하고 OTA/자동복구/로그 수집을 통해 운영 비용과 장애 대응 시간을 줄였습니다.

## Problem
- 현장 환경에서 통신 품질(지연/유실/노이즈)이 불안정
- 실시간 UI 업데이트로 인한 지연/프레임 저하
- 장애 발생 시 원격 진단/복구 수단 부족 → 운영 비용 증가

## Solution
- 통신 채널(Serial/UDP)별 에러 처리/타임아웃/재시도 정책 분리
- 상태 중심 UI 업데이트로 불필요한 렌더링 감소
- OTA + 자동 재부팅 + 로그 수집으로 운영 자동화 체계 구축
- 레거시(Java) → Kotlin 기반으로 단계적 리팩토링

## Result
- 150대+ 상용 디바이스 안정 운영
- UI 반응 지연 84% 개선
- 장애 대응 자동화로 운영 효율/복구 속도 개선(정성)

## Key Decisions / Trade-offs
- 안정성을 위해 통신 처리 로직을 방어적으로 설계(코드량 증가 vs 운영 리스크 감소)
- 운영 자동화를 우선해 초기 구현 범위를 확장(초기 비용 vs 장기 효율)

## Interview-ready
- Serial과 UDP 장애를 어떻게 구분/진단했나?
- UI 84% 개선을 어떤 접근으로 달성했나(상태/렌더링/스레딩)?
- OTA 실패/중단/롤백 시나리오는 어떻게 설계했나?
