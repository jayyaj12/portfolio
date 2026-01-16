---
layout: project
title: "Android 기반 실시간 엘리베이터 제어 및 DID 시스템"
subtitle: "레거시 Java 프로젝트를 Kotlin 기반의 MVVM, Clean Architecture로 재설계하고, 반응형 UI와 운영 자동화 시스템을 구축한 사례"
period: "2023.06 – 2025.06"
role: "Android Developer"
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
  - { value: "50%+", label: "현장 출동 비용 감소" }
---

## 1. Project Overview
레거시 Java 기반의 엘리베이터 제어 프로그램을 **Kotlin 기반의 MVVM, Clean Architecture 구조로 재설계**하고, 다양한 디스플레이 환경에 대응하는 **반응형 DID(Digital Information Display) 시스템을 구축**했습니다. 또한 Serial/UDP 통신 안정화 및 OTA(Over-the-Air) 원격 업데이트 시스템을 구현하여 150대 이상의 상용 디바이스를 안정적으로 운영하고 운영 비용을 절감했습니다.

<div class="mermaid">
graph TD
    subgraph A["건물 서버실 (Building Server Room)"]
        direction LR
        subgraph B["운영 PC"]
            Server["서버 프로그램"]
        end
    end

    subgraph C["엘리베이터 내부 (Inside Elevator)"]
        direction LR
        Android["Android DID 보드"]
    end

    Server -- "UDP 통신 (제어/상태 전송)" --> Android
</div>

---

## 2. Background & Problem
- **레거시 코드의 한계**: 기존 Java 기반 제어 프로그램은 아키텍처 부재로 유지보수가 어려웠고, 잦은 장애의 원인이 되었습니다.
- **화면 파편화**: 다양한 규격의 디스플레이 장비에 대응하기 위한 반응형 UI 아키텍처가 부재하여 UI가 깨지는 문제가 발생했습니다.
- **UI 렌더링 병목**: 고빈도로 수신되는 센서 데이터 처리가 UI 스레드에 부하를 주어 잦은 멈춤(Lag) 현상이 발생했습니다.
- **높은 운영 비용**: 장애 발생 시 원격 진단 및 복구 수단이 없어 매번 현장 출동이 필요했습니다.

---

## 3. Technical Strategy

### A. Architecture: 레거시 코드 재설계 (Refactoring)
- **How**: 레거시 Java 기반 제어 프로그램을 분석하여, **Kotlin 기반의 MVVM 및 Clean Architecture 구조로 재설계 및 재구현**했습니다. 데이터 처리, 비즈니스 로직, UI를 명확히 분리하여 코드의 테스트 용이성과 유지보수성을 향상시켰습니다.
- **Why**: 아키텍처 부재로 인한 복잡성을 해결하고, 향후 기능 확장 및 안정적인 운영을 위한 기반을 마련하기 위함이었습니다.

### B. UI: 반응형 DID 시스템 구축 (Responsive UI for DID)
- **How**: 다양한 디스플레이 규격(해상도, 비율)에 동적으로 대응하기 위해, **ConstraintLayout과 맞춤형 View Component**를 중심으로 반응형 UI 아키텍처를 설계했습니다.
- **Why**: 엘리베이터 DID 시스템의 특성상 설치 환경이 모두 다르기 때문에, 어떤 디스플레이에서도 UI가 깨지지 않고 일관된 정보를 표시하도록 하여 화면 파편화 문제를 근본적으로 해결했습니다.

### C. Communication: 통신 최적화 및 프로토콜 설계
- **Serial(RS-232/485)**: **비트 마스킹(Bitmasking)** 기반의 경량 패킷 처리를 통해 데이터 파싱 속도를 최적화했습니다. 1 Byte에 여러 상태를 비트 단위로 압축하여 GC 발생과 연산 지연을 최소화했습니다.
- **UDP Multicast**: 로컬 네트워크 내 다중 디바이스 제어를 위한 비동기 통신 구조를 설계했습니다.

<div class="mermaid">
graph LR
    Byte["1 Byte Packet"]
    Door["Door (2bit)"]
    Dir["Direction (2bit)"]
    Err["Error (4bit)"]
    
    Byte --- Door & Dir & Err
    
    style Door fill:#f9f,stroke:#333,stroke-width:2px
    style Dir fill:#bbf,stroke:#333,stroke-width:2px
    style Err fill:#ff9,stroke:#333,stroke-width:2px
</div>

### D. Performance: UI 렌더링 성능 개선
- **How**: Kotlin Coroutines의 **`SharedFlow`와 `conflate()`** 연산자를 적용하여, 렌더링 속도보다 빠르게 수신되는 데이터는 최신 값만 유지하고 중간 값은 드롭(Drop) 처리했습니다.
- **Why**: 모든 센서 데이터를 렌더링하는 것은 불가능하므로, UI 스레드 점유율을 방어하여 ANR(Application Not Responding)을 예방하고 사용자 경험을 개선했습니다.

### E. DevOps: 운영 자동화 (Operation Automation)
- **How**: Android 시스템 권한(System Signature)으로 `Runtime.exec()`를 사용하여 **Shell 스크립트를 실행**하는 방식을 도입, 일반 앱에서는 불가능한 **무인 업데이트(Silent Install) 및 원격 재부팅** 기능을 구현했습니다.
- **Why**: 관리자가 상주하지 않는 무인 환경 특성상, OS 레벨에서 시스템을 강제 제어하여 장애 발생 시 신속하게 자동 복구하고 업데이트를 배포하기 위함이었습니다.

---

## 4. Key Results
- **150대 이상**의 상용 디바이스 무중단 안정 운영 달성 (GS그룹 본사 등)
- 아키텍처 재설계 및 성능 최적화로 UI 렌더링 반응 지연 시간 **84% 단축** (1250ms → 200ms)
- 원격 업데이트 및 자동 복구 시스템 구축으로 **현장 출동 비용 50% 이상 절감**

---

## 5. Evidence & Benchmarks
**UI 반응 지연 84% 개선 근거**

- **Tools**: `System.nanoTime()` 기반의 구간 로깅 및 Android Profiler (JankStats) 활용
- **Metric**: 데이터 수신 시점(t1)부터 UI 반영 완료 시점(t2)까지의 Latency 분포 측정

#### Before: Rendering Queue Bottleneck
- 센서 데이터 수신 주기: **50ms** / 메인 스레드 처리 시간: **약 10~15ms**
- **문제**: 1초에 20개 패킷이 수신되면 렌더링 큐에 데이터가 계속 쌓여, 약 25개 패킷이 밀렸을 때 **50ms × 25 = 1,250ms의 심각한 지연** 발생

#### After: Conflation Strategy
- `Flow.conflate()` 적용: 렌더링 중 들어온 중간 데이터는 버리고 **최신 데이터만 소비**
- **결과**: 렌더링 주기에 맞춰 최신 상태만 반영하므로 **지연 시간을 200ms 이내**로 유지하여 사용자가 인지하지 못하는 수준으로 개선

---

## 6. Deep Dive & Trade-offs
- **시리얼 데이터 파편화(Fragmentation) 해결**: RS-232 통신 시 발생하는 패킷 조각화 문제를 해결하기 위해, **Circular Buffer**에 수신 바이트를 누적하고 Header/Footer 패턴 매칭으로 온전한 패킷을 재조립하는 파서(Parser)를 직접 구현했습니다.
- **안정성 우선 설계**: 시스템 안정성을 최우선으로 하여 통신 로직을 방어적으로 설계했으며, 이는 코드 복잡도 증가를 감수하고 운영 리스크를 최소화하기 위한 선택이었습니다.
