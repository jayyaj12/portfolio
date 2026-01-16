---
layout: project
title: "정부 국책과제: BLE 디바이스 제어 시스템"
subtitle: "불안정한 무선 환경에서의 데이터 무결성 및 연결 신뢰성 확보 전략"
period: "2021.01 – 2023.12"
featured: true
tags:
  - Android
  - Kotlin
  - MVVM
  - BLE / NFC
  - Clean Architecture
  - FSM (State Machine)
metrics:
  - { value: "100%", label: "KC 인증 성공률" }
  - { value: "3종", label: "앱 전 주기 개발" }
  - { value: "5개", label: "협력 기관 리딩" }
---

## 1. Project Overview
**"3년간 국책과제 개발팀 리드: 헬스케어 디바이스 제어 앱 3종 설계·구현·운영"**

정부 국책과제의 안드로이드 파트 리더로서, 5개 협력 기관과 협업하며 헬스케어 웨어러블 기기 연동 앱 3종을 개발했습니다.
무선 통신(BLE/NFC) 환경의 불안정성을 극복하기 위해 **이중 검증 계층(Double Validation Layer)**과 **FSM(Finite State Machine)**을 도입하여 데이터 무결성을 확보했으며, 엣지 케이스(Edge Case) 중심의 검증을 통해 **SW 기능성 KC 인증을 100% 성공률**로 획득했습니다.

---

## 2. Background & Problem
- **불안정한 연결 품질**: 무선 환경 특성상 연결 끊김 및 패킷 유실이 빈번하여 안정적인 데이터 수집이 어려움
- **데이터 무결성 훼손**: 노이즈로 인한 비트 플립(Bit Flip), 중복 수신, 순서 뒤바뀜 등으로 데이터 신뢰성 저하
- **디버깅의 어려움**: 비동기 콜백(Callback)의 복잡한 흐름으로 인해 문제 발생 시 원인 파악과 재현이 난해함

---

## 3. Technical Strategy

### A. 통신 신뢰성 확보 (Reliability)
데이터의 **정확성(Accuracy)**과 **순서(Sequence)**를 보장하기 위해 애플리케이션 레벨에서 검증 로직을 강화했습니다.

- **Checksum & Packet Validation (무결성 검증)**
  - **Problem**: 하드웨어 레벨의 검증을 통과하더라도, 애플리케이션 전달 과정에서 데이터가 변조될 가능성 존재
  - **Solution**: 수신된 바이트 배열을 `CRC-16` 알고리즘으로 2차 검증. 검증 실패 패킷은 즉시 **폐기(Drop)**하여 비즈니스 로직 오염 원천 차단

- **Timestamp & Sequence (순서 보장)**
  - **Problem**: BLE의 재전송(Retransmission) 특성으로 인해 패킷이 중복되거나 순서가 뒤바뀌어 도착하는 현상
  - **Solution**: 패킷 헤더에 타임스탬프를 포함시키고, 수신 측에서 `LastProcessedTime`과 대조하여 **유효한 최신 패킷만 처리**

<div class="mermaid">
sequenceDiagram
    participant App
    participant Device
    App->>Device: Request Command
    Device-->>App: Response Packet
    Note over App: 1. Check CRC-16 (Integrity)
    alt CRC Valid
        Note over App: 2. Check Timestamp (Sequence)
        App->>UI: Update State
    else CRC Invalid
        App->>App: Drop Packet (Log Warning)
    end
</div>

### B. 상태 기반 흐름 제어 및 자가 복구 (FSM & Self-Healing)
복잡한 비동기 통신 흐름을 제어하고, 예외 상황에서 사용자 개입 없이 시스템을 복구하기 위해 FSM과 Watchdog을 적용했습니다.

- **Finite State Machine (FSM)**
  - **Implementation**: 연결 상태(`Disconnected`, `Scanning`, `Connecting`, `Ready`)를 Kotlin `Sealed Class`로 정의
  - **Effect**: `StateFlow`를 통해 상태 변화를 UI 및 로직에 일관되게 전파하며, 중첩된 콜백(Callback Hell)으로 인한 예외 처리 누락 방지

- **NFC/BLE 연결 예외 처리 및 UI 자동 복구**
  - **Problem**: 정전기(ESD) 테스트 등 외부 충격으로 통신 스택이 멈추거나, NFC 태깅 실패 시 앱이 멈춘 것처럼 보이는 현상
  - **Solution**: `BluetoothAdapter` 및 NFC 어댑터 상태를 주기적으로 모니터링하는 **Watchdog**을 구현하여, 비정상 감지 시 프로세스를 자동 재시작하고 사용자에게는 '재연결 중' 상태를 시각화하여 UX 경험 유지

<div class="mermaid">
stateDiagram-v2
    [*] --> Disconnected
    Disconnected --> Scanning : User Action / Auto Retry
    Scanning --> Connecting : Device Found
    Connecting --> Ready : Connection Success
    Connecting --> Disconnected : Connection Fail
    Ready --> Disconnected : Error / User Action
</div>

---

## 4. Key Results
- **프로젝트 리딩**: 5개 협력 기관과의 인터페이스 표준화 주도 및 일정 리스크 관리로 과제 완수
- **품질 인증**: 엣지 케이스(Edge Case) 중심의 시나리오 테스트를 통해 **SW 기능성 KC 인증 획득 (성공률 100%)**
- **데이터 신뢰성**: CRC 및 타임스탬프 검증 도입 후, 데이터 오염으로 인한 오동작 리스크 **0% 달성**

---

## 5. Engineering Evidence
단순한 연결 구현을 넘어, **데이터 무결성을 보장하기 위해 설계한 패킷 구조**입니다.

**[Packet Structure Design]**

| STX (1B) | LEN (1B) | CMD (1B) | PAYLOAD (N) | TIMESTAMP (4B) | CRC16 (2B) | ETX (1B) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Start | Length | Command | Data | Sequence | Checksum | End |

- **CRC16**: Payload 변조 여부를 앱 레벨에서 엄격하게 검증
- **TIMESTAMP**: `System.currentTimeMillis()`를 압축하여 패킷 순서 및 중복 여부 판단

> **Measurement**: 필드 테스트 로그 분석 결과, 전체 수신 패킷의 약 **3~5%**가 CRC 에러 또는 중복 패킷으로 식별되어 방어되었습니다. 이 방어 로직이 없었다면 데이터 정합성 문제가 발생했을 것입니다.

---

## 6. Deep Dive & Trade-offs
### KC 인증(EMC/ESD) 대응 경험
- **Challenge**: 하드웨어 ESD(정전기) 내성 테스트 중, 하드웨어 충격으로 인해 안드로이드의 Bluetooth Stack이 일시적으로 멈추는 현상 발생
- **Action**:
  - 단순한 예외 처리를 넘어, 시스템 레벨에서 어댑터 상태를 감지하고 스스로 복구하는 **Self-Healing(자가 복구)** 메커니즘 구현
  - 비정상 상태 감지 시, 사용자 개입 없이 BLE 스캔 및 연결 프로세스를 자동으로 재시작
- **Result**: 하드웨어적 불안정 상황에서도 앱이 종료되지 않고 서비스를 유지하는 **복원력(Resilience)**을 입증하여 인증 통과
