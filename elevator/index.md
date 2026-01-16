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
- **통신 환경 불안정**: 현장 특성상 노이즈로 인한 지연/유실 발생
- **UI 렌더링 성능 저하**: 실시간 고빈도 데이터 수신으로 인한 프레임 드랍(Lag)
- **운영 비용 증가**: 장애 발생 시 원격 진단/복구 수단이 없어 현장 출동 필요

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

<div class="mermaid">
sequenceDiagram
    participant Sensor
    participant Flow
    participant UI
    
    Sensor->>Flow: Data 1 (50ms)
    Sensor->>Flow: Data 2 (100ms)
    Sensor->>Flow: Data 3 (150ms)
    Note over Flow: Conflate (Drop 1, 2)
    Flow->>UI: Emit Data 3
    Note over UI: Render (200ms)
</div>

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

## 5. Performance Analysis
**"UI 반응 지연 84% 개선"**에 대한 구체적인 산출 근거와 측정 방법입니다.

**[Measurement Method]**
- **Tools**: `System.nanoTime()` 기반의 구간 로깅 및 Android Profiler (JankStats) 활용
- **Metric**: (데이터 수신 시점 `t1` - UI 반영 완료 시점 `t2`)의 Latency 분포 측정

**[Before: Rendering Queue Bottleneck]**
- 센서 데이터 수신 주기: **50ms**
- 메인 스레드 처리 시간: **약 10~15ms** (복잡한 UI 갱신)
- **문제**: 1초에 20개 패킷 수신 → 큐에 쌓임 → 약 25개 패킷이 밀렸을 때 **50ms × 25 = 1,250ms 지연** 발생

**[After: Conflation Strategy]**
- `Flow.conflate()` 적용: 렌더링 중 들어온 중간 데이터는 버리고 **최신 데이터만 소비**
- **결과**: 렌더링 주기(약 16ms~33ms)에 맞춰 최신 상태만 반영하므로 **지연 시간 200ms 이내(사람이 인지 못하는 수준)**로 유지

---

## 6. Deep Dive & Trade-offs
- **시리얼 데이터 파편화(Fragmentation) 해결**:
  - RS-232 통신 시 패킷이 잘려서 들어오는 현상을 해결하기 위해, **Circular Buffer**를 도입하여 바이트를 누적하고 Header/Footer 패턴 매칭으로 온전한 패킷을 조립하는 파서(Parser)를 직접 구현
- 안정성을 위해 통신 처리 로직을 방어적으로 설계(코드량 증가 vs 운영 리스크 감소)
- 운영 자동화를 우선해 초기 구현 범위를 확장(초기 비용 vs 장기 효율)
