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

## 1. Project Overview
현장 장비와 연결되는 Android 앱을 단독 개발·운영하며, Serial/UDP 통신 기반 제어를 안정화하고 OTA/자동복구/로그 수집을 통해 운영 비용과 장애 대응 시간을 줄였습니다.

---

## 2. Background & Problem
- 현장 환경에서 통신 품질(지연/유실/노이즈)이 불안정
- 실시간 UI 업데이트로 인한 지연/프레임 저하
- 장애 발생 시 원격 진단/복구 수단 부족 → 운영 비용 증가

---

## 3. Technical Strategy

### A. 통신 최적화 및 프로토콜 설계
- **Serial(RS-232/485)**: 비트 마스킹(Bitmasking) 기반의 경량 패킷 처리를 통해 데이터 파싱 속도 최적화
  - **How**: 1 Byte에 도어(Open/Close), 방향(Up/Down), 에러 등 다중 상태를 비트 단위로 패킹하고, `(byte & MASK)` 연산으로 상태 추출
  - **Why**: 고빈도 패킷 수신 환경에서 String/JSON 파싱으로 인한 메모리 할당(GC)과 연산 지연을 제거하기 위함
- **UDP Multicast**: 로컬 네트워크 내 다중 디바이스 제어를 위한 비동기 통신 구조 설계

### B. UI 렌더링 성능 개선
- 수신되는 패킷 빈도(High Frequency)와 UI 갱신 주기의 불일치로 인한 렉(Lag) 발생
- **Coroutine Flow & Conflation**:
  - **How**: Kotlin Coroutines의 `SharedFlow`와 `conflate`(또는 `sample`) 연산자를 적용하여, 렌더링 속도보다 빠르게 들어오는 데이터 중 최신 상태만 유지하고 이전 데이터는 드롭(Drop) 처리
  - **Why**: 모든 센서 데이터를 그리는 것은 불가능하므로, 사람의 인지 속도에 맞춰 UI 스레드 점유율을 방어하고 ANR(Application Not Responding)을 예방

### C. 운영 자동화 (DevOps)
- **System Level Control**:
  - **How**: Android 시스템 권한(System Signature)을 획득한 상태에서 `Runtime.exec()`를 통해 쉘 스크립트를 실행, 앱 레벨에서 불가능한 사일런트 업데이트(Silent Install)와 재부팅 구현
  - **Why**: 현장 관리자가 없는 무인 환경이므로, OS 레벨의 강제성이 있는 복구 수단이 필수적임

---

## 4. Key Results
- 150대+ 상용 디바이스 안정 운영
- UI 반응 지연 84% 개선
- 장애 대응 자동화로 운영 효율/복구 속도 개선(정성)

---

## 5. Deep Dive & Trade-offs
- **시리얼 데이터 파편화(Fragmentation) 해결**:
  - RS-232 통신 시 패킷이 잘려서 들어오는 현상을 해결하기 위해, **Circular Buffer**를 도입하여 바이트를 누적하고 Header/Footer 패턴 매칭으로 온전한 패킷을 조립하는 파서(Parser)를 직접 구현
- 안정성을 위해 통신 처리 로직을 방어적으로 설계(코드량 증가 vs 운영 리스크 감소)
- 운영 자동화를 우선해 초기 구현 범위를 확장(초기 비용 vs 장기 효율)
