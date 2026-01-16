---
layout: project
title: "Android 기반 실시간 엘리베이터 제어 및 DID 시스템"
subtitle: "레거시 Java 프로젝트를 Kotlin 기반의 MVVM, Clean Architecture로 재설계하고, 반응형 UI와 운영 자동화 시스템을 구축한 사례"
period: "2023.06 – 2025.06"
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
    subgraph A["건물 서버실"]
        direction LR
        subgraph B["운영 PC"]
            Server["서버 프로그램"]
        end
    end

    subgraph C["엘리베이터 내부"]
        direction LR
        Android["Android DID 보드"]
    end

    Server <-- "UDP 통신 (제어/상태 전송)" --> Android
</div>

---

## 2. Background & Problem
- **레거시 시스템의 한계**: 기존 Java 기반 제어 프로그램은 아키텍처 부재로 비즈니스 로직과 UI가 분리되지 않았고, 단일 파일의 코드 길이가 **7,000줄**을 넘는 등 복잡도가 매우 높아 기능 추가 및 유지보수가 거의 불가능한 상태였습니다.
- **비효율적인 데이터 처리**: 수신된 `ByteArray` 패킷을 문자열로 변환 후, 특정 위치의 문자를 파싱하여 상태를 확인하는 비효율적인 구조로 인해 성능 저하가 발생했습니다.
- **화면 파편화**: 다양한 규격의 디스플레이 장비에 대응하기 위한 반응형 UI 아키텍처가 부재하여 UI가 깨지는 문제가 발생했습니다.
- **높은 운영 비용**: 장애 발생 시 원격 진단 및 복구 수단이 없어 매번 현장 출동이 필요했습니다.

---

## 3. Technical Strategy

### A. Architecture: 레거시 시스템 재구축 (Re-engineering)
- **How**: 유지보수가 불가능했던 기존 레거시 시스템을 분석하여, **Kotlin 기반의 MVVM 및 Clean Architecture 구조로 완전히 재구축**했습니다. 데이터 처리, 비즈니스 로직, UI를 명확히 분리하여 코드의 테스트 용이성과 확장성을 확보했습니다.
- **Why**: 단순한 코드 개선(Refactoring)을 넘어, 향후 2~3년의 서비스 확장 계획에 대응할 수 있는 지속 가능한 아키텍처 기반을 마련하기 위함이었습니다.

### B. UI: 반응형 DID 시스템 구축 (Responsive UI for DID)
- **How**: 다양한 디스플레이 규격(해상도, 비율)에 동적으로 대응하기 위해, **ConstraintLayout과 맞춤형 View Component**를 중심으로 반응형 UI 아키텍처를 설계했습니다.
- **Why**: 엘리베이터 DID 시스템의 특성상 설치 환경이 모두 다르기 때문에, 어떤 디스플레이에서도 UI가 깨지지 않고 일관된 정보를 표시하도록 하여 화면 파편화 문제를 근본적으로 해결했습니다.

### C. Communication: 통신 프로토콜 설계
- **Serial(RS-232/485)**: 실시간 제어를 위한 시리얼 통신 프로토콜을 설계하고 파서를 구현했습니다.
- **UDP Multicast**: 로컬 네트워크 내 다중 디바이스 동시 제어를 위한 비동기 통신 구조를 설계했습니다.

### D. Performance: 데이터 파싱 성능 최적화
- **How**: 100ms마다 반복 실행되는 데이터 파싱 로직의 성능 병목을 해결하기 위해, 기존의 문자열 변환 및 파싱 로직을 **비트 마스킹(Bitmasking)**으로 대체했습니다.
- **Why**: `ByteArray`를 `String`으로 변환하고, 그 문자열의 특정 문자가 '1'인지 '0'인지 확인하는 기존 방식은 CPU에 큰 부하를 주었습니다. 이 CPU 집약적인 작업을 `(byte & MASK)`라는 단일 비트 연산으로 변경하여, 파싱에 소요되는 시간을 거의 0으로 만들어 메인 스레드의 병목을 원천적으로 제거했습니다.

<div class="mermaid">
graph LR
    subgraph "Status Byte (8bit)"
        Door["Door (2bit)"]
        Dir["Direction (2bit)"]
        Err["Error (4bit)"]
    end
    
    Byte("Packet") -.-> Door & Dir & Err
    
    style Door fill:#f9f,stroke:#333,stroke-width:2px
    style Dir fill:#bbf,stroke:#333,stroke-width:2px
    style Err fill:#ff9,stroke:#333,stroke-width:2px
</div>

### E. DevOps: 원격 무인 업데이트 (OTA Automation)
- **How**: 아래와 같은 프로세스로 원격 무인 업데이트(OTA) 시스템을 구축했습니다.
  1. Android 디바이스는 서버와 **상시 UDP 통신**하며 연결 상태를 유지합니다.
  2. 관리자가 서버에서 업데이트 명령을 실행하면, 서버는 해당 디바이스에 **UDP로 업데이트 명령을 전송**합니다.
  3. 명령을 수신한 디바이스는 지정된 **FTP 서버** 경로에서 **신규 APK 파일을 다운로드**합니다.
  4. 다운로드 완료 후, 시스템 권한으로 **Shell 스크립트(`pm install -r ...`)를 실행**하여 앱을 강제로 재설치(Silent Install)하고 디바이스를 재부팅하여 업데이트를 완료합니다.
- **Why**: 관리자가 상주하지 않는 수백 대의 디바이스를 물리적 방문 없이 원격으로 일괄 업데이트하고, 장애 발생 시 자동 복구하며 **원격으로 로그를 수집하여 신속하게 원인을 분석**하기 위함이었습니다.

---

## 4. Key Results
- **150대 이상**의 상용 디바이스 무중단 안정 운영 달성 (GS그룹 본사 등)
- **데이터 파싱 로직 최적화**로 UI 반응 지연 시간 **84% 단축** (1250ms → 200ms)
- 원격 업데이트 및 자동 복구 시스템 구축으로 **현장 출동 비용 50% 이상 절감**

---

## 5. Evidence & Benchmarks
**파싱 최적화를 통한 UI 반응 지연 84% 개선**

- **Tools**: `System.nanoTime()` 기반의 구간 로깅 및 Android Profiler (JankStats) 활용
- **Metric**: 데이터 수신 시점(t1)부터 UI 반영 완료 시점(t2)까지의 Latency 분포 측정

#### Before: 문자열 파싱 병목 (String Parsing Bottleneck)
- **문제**: 100ms 주기로 데이터 변경이 감지될 때마다, `ByteArray`를 `String`으로 변환하고 문자 단위로 상태를 확인하는 **CPU 집약적인 파싱 로직**이 실행되었습니다. 이 파싱 작업이 메인 스레드를 수십 ms 동안 점유하면서 UI 업데이트를 지연시켰고, 이로 인해 최종 **반응 속도가 1250ms까지 지연**되는 현상이 발생했습니다.

#### After: 비트마스킹 적용 (Bitmasking Applied)
- `String` 변환 및 파싱 로직을 모두 제거하고, `(byte & MASK)`라는 단일 비트 연산으로 로직을 대체했습니다.
- **결과**: 파싱에 소요되던 CPU 점유 시간이 거의 0에 가깝게 줄어들면서 메인 스레드의 병목이 완전히 해소되었습니다. 이를 통해 **지연 시간을 200ms 이내**로 안정적으로 유지하여, 사용자가 인지하지 못하는 수준의 빠른 반응 속도를 확보할 수 있었습니다.

---

## 6. Deep Dive & Trade-offs
- **시리얼 데이터 파편화(Fragmentation) 해결**: RS-232 통신은 데이터가 조각나 수신되는 문제가 잦아, 이를 해결할 **데이터 재조립 파서(Parser)를 직접 구현**했습니다.
  - 먼저, 수신되는 데이터 조각들을 임시 저장소인 **Circular Buffer**에 순서대로 쌓습니다.
  - 그 다음, 파서가 버퍼를 계속 감시하다 약속된 **시작 신호(Header)**를 발견하면 데이터 기록을 시작합니다.
  - 마지막으로, **끝 신호(Footer)**를 만날 때까지 들어온 모든 조각을 하나로 합쳐 **온전한 명령어 패킷으로 복원**합니다. 이를 통해 통신 안정성을 확보했습니다.
- **안정성 우선 설계**: 시스템 안정성을 최우선으로 하여 통신 로직을 방어적으로 설계했으며, 이는 코드 복잡도 증가를 감수하고 운영 리스크를 최소화하기 위한 선택이었습니다.
